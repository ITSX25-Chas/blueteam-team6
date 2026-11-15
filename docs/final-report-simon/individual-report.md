# Individuell slutrapport
**Namn:** Simon Niklasson
**Projekt/team:** Team 6
**Datum:** 2026-11-15

---

## 1. Professionell sammanfattning
Projektets syfte har varit att utvärdera säkerheten och motståndskraften i företagets webbapplikation och tillhörande infrastruktur, både ur ett försvarande perspektiv (Blue Team) och genom ett kontrollerat penetrationstest med efterföljande logganalys (Red Team). Resultatet visar att miljön innehåller flera kritiska säkerhetsbrister som tillsammans gör det möjligt för en angripare att snabbt gå från ett enkelt webbanrop till att komma åt känsliga personuppgifter och ta över hela den bakomliggande servern.

De mest allvarliga bristerna vi identifierade är:
- **Bristande inloggning och behörighetskontroll:** Obehöriga kan kringgå inloggningen eller ge sig själva administratörsrättigheter genom att ändra en enkel cookie i webbläsaren.
- **Möjlighet att köra kommandon och ta över servern:** Ett diagnostikverktyg på administrationssidan tillåter att vem som helst som når sidan kan köra egna systemkommandon och få högsta möjliga behörighet (`root`) på servern.
- **Oskyddade personuppgifter och hemligheter på insidan:** Lösenord och interna systemadresser ligger sparade i klartext i konfigurationsfiler och kodhistorik, vilket gjorde att vi kunde hämta ut en intern sammanställning med anställdas löner och personnummer.

Ur ett verksamhetsperspektiv innebär detta en mycket hög risk. Ett verkligt intrång skulle inte bara kunna leda till driftstopp och förlorad kontroll över IT-miljön, utan även till en allvarlig personuppgiftsincident med anmälningsplikt och sanktionsavgifter enligt GDPR samt skadat förtroende hos kunder och anställda.

För att vända läget rekommenderas att företaget **omedelbart** säkrar inloggningen och flyttar behörighetskontrollen till serversidan, vilket stänger den enkla vägen in. Parallellt bör diagnostikverktyget stängas av eller byggas om så att det inte kan köra kommandon på servern, och alla lösenord och personuppgifter skyddas med ordentlig åtkomstkontroll.

---

För att ledningen ska kunna följa upp att säkerhetsarbetet ger effekt föreslår jag tre centrala KPI:er:
- **Åtgärdstid för kritiska sårbarheter (MTTR)**
- **Efterlevnad av minsta möjliga behörighet (Least Privilege)**
- **Exponerade hemligheter i kod och konfiguration**

**Åtgärdstid för kritiska sårbarheter** mäter hur snabbt organisationen klarar av att stänga allvarliga säkerhetshål när de väl har upptäckts. Vårt mål bör vara att kritiska brister åtgärdas inom 48 timmar, så att ett upptäckt hål inte blir stående öppet.

**Efterlevnad av minsta möjliga behörighet** visar i vilken grad våra systemkonton, molntjänster och applikationer bara har tillgång till exakt det de behöver för att fungera. Under testerna såg vi att webbapplikationen kunde läsa skyddade administratörsfiler på servern och att molnkonton hade för breda rättigheter. Genom att hålla detta mått nära 100 % begränsar vi skadan kraftigt om ett intrång ändå skulle ske.

**Exponerade hemligheter i kod och konfiguration** mäter antalet lösenord, nycklar och känsliga adresser som ligger oskyddade i källkod, konfigurationsfiler eller Git-historik. Målvärdet ska vara 0, eftersom varje läckt nyckel ger en angripare en gratis genväg vidare in i företagets system och datalager.

---

## 2. Kontext och avgränsning
Projektet har genomförts i två sammanhängande delar mot företagets labb- och målmiljö. I den första delen (Blue Team) kartlade och analyserade vi sårbarheter i applikationen, Git-historiken och molninfrastrukturen (Google Cloud och Terraform) och kopplade dem till tekniska svagheter (CWE), angripartekniker (MITRE ATT&CK) och en samlad hotbild. I den andra delen (Red Team) genomförde vi ett strukturerat penetrationstest mot målmiljön på `http://10.0.0.9:8080` följt av ett post-mortem-arbete där vi granskade försvarssidans loggar (`web.jsonl`, `db.jsonl` och `internal.jsonl`).

Alla tester under Red Team-fasen utfördes inom ramarna för en undertecknad Rules of Engagement (RoE) under testperioden 19–30 oktober 2026. Vi arbetade uteslutande mot vår tilldelade målmiljö under ändringsstopp, utan att påverka tillgängligheten för andra eller ändra i systemet. Eftersom rapporterna innehåller detaljer om sårbarheter, interna sökvägar och infrastruktur har materialet hanterats enligt klassificeringen `TLP:AMBER`.

---

## 3. Mitt individuella bidrag
Under projektet har jag arbetat tätt tillsammans med övriga medlemmar i Team 6 och bidragit aktivt i såväl det praktiska letandet efter sårbarheter som i logganalysen och rapportskrivningen.

I det praktiska arbetet bidrog jag till att hitta och verifiera flera av buggarna genom att systematiskt undersöka hur webbapplikationen hanterade användarkonton, parametrar, cookies och inmatningsfält:
- **Kartläggning av användare, profilsidor och behörigheter:** Under rekognosceringen var jag med och undersökte applikationens endpoints med `curl` och i webbläsaren (`/login`, `/register`, `/profile`, `/admin` och `/admin/diag`). Jag testade bland annat hur vanliga användarkonton och profilsidor fungerade, där vi upptäckte att man kunde läsa ut administratörens profiluppgifter direkt via `/profile?id=1` utan autentisering och att servern litade på en klientstyrd cookie (`role=admin`) för att ge tillgång till `/admin`. Upptäcktera kring hur vanliga konton och behörigheter betedde sig användes även av resten av gruppen när vi testade vidare.
- **Inloggning och kommandoinjektion:** Jag var med och testade SQL-injektion i inloggningsformuläret (`' OR '1'='1'`) och undersökte därefter nätverksdiagnostiken under `/admin/diag`. Genom att testa att skicka med shell-operatorer som `|` och `;` efter värdnamnet (t.ex. `| cat secret_config.py`) kunde vi bekräfta att applikationen körde våra kommandon direkt på servern.
- **Åtkomst till internt API och rättighetseskalering till root:** Utifrån innehållet i `secret_config.py` såg vi adressen `http://internal:8000/export` och kunde via ett Python-anrop i diagnostikverktyget hämta ut de interna personuppgifterna och flaggan.
- **Infrastruktur, post-mortem och dokumentation:** Under Blue Team-delen var jag även med och letade flaggor i Git-historik, dolda branches och molnkonfigurationer samt granskade Terraform-filen `main.tf`. Efter Red Team-testerna var jag med och vi diskutera CWE- och MITRE ATT&CK-mappningar och formulera åtgärdsförslagen i våra rapporter.

---

## 4. Viktigaste projektresultat och evidens

### 1. Bristande behörighetskontroll och autentisering (Broken Access Control & SQL Injection)
Webbtjänsten saknar tillförlitlig kontroll av både inloggning och användarbehörigheter på serversidan. En angripare kan dels logga in som administratör utan lösenord via SQL-injektion, dels nå administratörssidan `/admin` och användarprofiler (`/profile?id=1`) genom att skicka med en egen cookie (`role=admin`).
#### Evidens
I Red Team-rapporten (avsnitt 4.1 och 4.2 samt Bilaga 2 och 3) visar vi hur kommandot `curl -i -b "role=admin" http://127.0.0.1:8080/admin` ger direkt åtkomst till administrationssidan och motsvarande flagga (`broken_access_control`), samt hur inloggning med `' OR '1'='1'` ger administratörsåtkomst (`sqli_foothold`) och syns i `db.jsonl` (rad 23). Motsvarande sårbarheter (SQLi och IDOR) bekräftades även i Blue Team-rapporten (avsnitt 4.5 och 4.6).
#### Verksamhets- och säkerhetsbetydelse
Eftersom säkerhetskontrollen går att runda på några sekunder saknar webbtjänsten i praktiken ett fungerande skalskydd. Vem som helst som kan nå tjänsten utifrån kan utge sig för att vara administratör och få tillgång till interna verktyg och uppgifter.
Se även [lessons-learned.md](./lessons-learned.md).
#### Rekommendation
**Prioritet: Omgående (P1).**
Utvecklingsteamet måste införa parametriserade databasfrågor (prepared statements) i inloggningen och ersätta den osignerade rollcookien med en säker, serverbaserad sessionshantering där användarens behörighet kontrolleras på servern vid varje anrop.

### 2. Kommandoinjektion och fullständig serverkompromettering (Command Injection & Privilege Escalation)
Administrationssidans nätverksdiagnostik (`/admin/diag`) skickar användarens inmatning direkt till operativsystemets skal utan kontroll. I kombination med att systemverktyget `/usr/bin/find` kan köras med förhöjda rättigheter på servern kan en angripare exekvera valfria kommandon och läsa filer som tillhör systemadministratören (`root`).
#### Evidens
Genom att mata in `| cat secret_config.py` i diagnostiken läste vi ut applikationens hemliga konfigurationsfil och RCE-flaggan (Red Team-rapporten avsnitt 4.3, Bilaga 4 samt `web.jsonl` rad 253). Genom att därefter köra `/usr/bin/find /root/flag5.txt -exec cat {} \;` via samma endpoint kunde vi läsa ut innehållet i `/root/flag5.txt` (`privesc_root`, avsnitt 4.5, Bilaga 6 samt `web.jsonl` rad 32866). I Blue Team-rapporten såg vi en liknande risk genom Server-Side Template Injection (SSTI, avsnitt 4.1 och 4.10).
#### Verksamhets- och säkerhetsbetydelse
När en angripare kan köra egna kommandon och nå `root`-nivå är inte bara webbapplikationen utan hela servern förlorad. Angriparen kan stänga ner tjänsten, manipulera data, radera lokala loggar eller använda servern för att angripa andra delar av infrastrukturen.
Se [kpi.md](./kpi.md) (KPI 1 och KPI 2) samt [lessons-learned.md](./lessons-learned.md) (Lesson learned 2).
#### Rekommendation
**Prioritet: Omgående (P1).**
Stäng av eller bygg om `/admin/diag` så att den aldrig anropar operativsystemets shell med användarinmatning, och tillåt endast strikt validerade IP-adresser. Ta bort onödiga förhöjda rättigheter (SUID) på systemverktyg som `find` och kör webbapplikationen under ett konto med minimala rättigheter.

### 3. Exponering av interna personuppgifter (PII) och hemligheter i kod och infrastruktur
Känsliga uppgifter som databaslösenord, sessionsnycklar och interna API-adresser lagras i klartext i konfigurationsfiler och versionshistorik, och interna tjänster saknar egen åtkomstkontroll.
#### Evidens
I `secret_config.py` (Red Team Bilaga 4) exponerades både databaslösenordet (`DB_PASSWORD = "<redacted>"`) och adressen `http://internal:8000/export`. Via kommandoinjektionen kunde vi anropa `/export` och hämta ut en JSON-struktur med anställdas namn, roller, löner och personnummer samt motsvarande flagga (`internal_pii_exfil`, Red Team avsnitt 4.4, Bilaga 5 samt `internal.jsonl` rad 22). I Blue Team-rapporten (avsnitt 4.2–4.4, 4.7–4.9 och kapitel 10) påvisades dessutom läckta hemligheter i `terraform.tfstate`-backup, Git-historik, en gömd Git-branch, en hårdkodad Flask `SECRET_KEY` samt en brandväggsregel i `main.tf` som tillät trafik från `0.0.0.0/0` på samtliga protokoll.
#### Verksamhets- och säkerhetsbetydelse
Att anställdas personnummer och löner kan hämtas ut utan autentisering utgör en allvarlig personuppgiftsincident enligt GDPR, vilket kan medföra dryga sanktionsavgifter och skadat förtroende. Samtidigt gör öppna brandväggsregler och läckta nycklar i kod och molnkonfiguration att ett intrång snabbt sprider sig till hela molnmiljön.
Se [kpi.md](./kpi.md) (KPI 2 och KPI 3) samt [lessons-learned.md](./lessons-learned.md) (Lesson learned 1).
#### Rekommendation
**Prioritet: Hög (P1/P2).**
Flytta samtliga lösenord och nycklar till en dedikerad Secret Manager och rotera alla exponerade hemligheter. Inför autentisering på det interna API:et `/export`, rensa känslig historik i Git och begränsa brandväggsreglerna i Terraform så att endast nödvändiga portar är öppna.

---

## 5. KPI:er och ledningsrapportering
För att ge ledningen ett tydligt och begripligt beslutsunderlag har jag i [kpi.md](./kpi.md) tagit fram tre KPI:er som knyter direkt an till de brister vi upptäckte:

#### 1. Åtgärdstid för kritiska sårbarheter (MTTR)
Visar hur lång tid det i genomsnitt tar från att en kritisk sårbarhet (som SQL-injektionen eller kommandoinjektionen) upptäcks tills den är åtgärdad i produktion. Målvärdet är under 48 timmar för kritiska brister. Om tiden drar iväg visar det för ledningen att säkerhetsfixar behöver prioriteras upp framför vanlig nyutveckling.

#### 2. Efterlevnad av minsta möjliga behörighet (Least Privilege Compliance Rate)
Visar hur stor andel av våra systemkonton, serverprocesser och molnidentiteter som bara har de rättigheter de faktiskt behöver (målvärde 100 %, gränsvärde 95 %). Detta mått adresserar direkt varför vi kunde läsa `/root/flag5.txt` och komma åt Cloud Storage. Ett högt värde säkerställer att ett intrång i en enskild webbtjänst inte leder till att hela servern eller molnprojektet tas över.

#### 3. Exponerade hemligheter i kod och konfiguration (Unsecured Secrets in Code & Config)
Visar antalet oskyddade lösenord, API-nycklar och sessionsnycklar i vår källkod, våra konfigurationsfiler och vår Git-historik. Målvärdet är 0. Genom att mäta detta kontinuerligt med automatisk skanning i CI/CD-flödet förhindrar vi att angripare hittar färdiga databaslösenord (som i `secret_config.py`) eller nycklar i äldre commits.

**Begränsning i mätningen:** Dessa tre KPI:er ger god kontroll över hur snabbt vi lagar brister och hur väl vi begränsar behörigheter och skyddar hemligheter, men de mäter inte i sig hur snabbt vi upptäcker ett pågående angrepp i loggarna. Därför måste de kombineras med de förbättringar inom loggövervakning och larm som beskrivs i post-mortem-analysen nedan.

---

## 6. Incidentrespons, post-mortem och förbättringsarbete
Efter penetrationstestet genomförde vi en post-mortem-analys av målmiljöns loggfiler (`web.jsonl`, `db.jsonl` och `internal.jsonl`) för att utreda vad försvarssidan hade möjlighet att upptäcka.

Analysen visade att **samtliga fem angreppssteg lämnade tydliga spår i loggarna**:
1. **Broken Access Control:** Anropet `GET /admin` med HTTP-status `200` loggades i `web.jsonl` (rad 206, 20 okt kl. 13:51 svensk tid), även om själva cookie-värdet `role=admin` inte syntes i standardloggen.
2. **SQL Injection:** Den manipulerade SQL-frågan `SELECT id, username, role FROM users WHERE username = 'admin' AND password = 'admin ' OR '1'='1' '` loggades i klartext i `db.jsonl` (rad 23, 20 okt kl. 13:50).
3. **Command Injection:** Anropet `GET /admin/diag?host=8.8.8.8+|+cat+secret_config.py` loggades direkt i `web.jsonl` (rad 253, 20 okt kl. 13:55).
4. **Exponering av personuppgifter:** Anropet `GET /export` från det interna nätverket (`172.18.0.4`) med status `200` registrerades i `internal.jsonl` (rad 22, 20 okt kl. 14:34).
5. **Åtkomst till root-fil:** Anropet mot `/admin/diag` med `/usr/bin/find+/root/flag5.txt+-exec+cat+{}+\\;` syntes i klartext i `web.jsonl` (rad 32866, 27 okt kl. 14:33).

Den viktigaste lärdomen från vårt post-mortem (se [lessons-learned.md](./lessons-learned.md)) är att **passiv loggning inte räcker om ingen övervakar eller larmar på loggarna**. I den ursprungliga miljön (`A_original`) fanns 34 368 loggrader jämfört med 87 rader i `B_testfonster`, men trots att våra angrepp syntes tydligt i råloggarna triggades inga automatiska säkerhetslarm och inga incidentåtgärder sattes in under testet.

**Konkret förbättringsarbete och uppföljning:**
- **Kort sikt (Omgående):** Stäng de tekniska hålen i inloggning, rollcookies och `/admin/diag` samt begränsa behörigheterna på servern och det interna API:et.
- **Medellång sikt:** Samla loggarna från `web`, `db` och `internal` centralt (SIEM) och sätt upp automatiska larmregler för tydliga indikatorer: SQL-syntaxfel och avvikande inloggningar i databasen, anrop till `/admin/diag` som innehåller tecken som `;` eller `|`, oväntade processer som startas av webbservern (som `sh`, `python3` eller `find`) samt anrop mot `/export`.
- **Uppföljning:** Genomför ett nytt kontrollerat test efter att åtgärderna och larmreglerna är på plats för att verifiera dels att sårbarheterna är borta, dels att larmen faktiskt löser ut i realtid.

---

## 7. AI i säkerhetsarbetet

### Konkreta användningsfall
Under projektets gång har vi använt generativa AI-verktyg som ett stöd och bollplank främst i dokumentations- och analysfasen:
- Vi använde AI för att ta fram ett förslag på grundstruktur och mall för våra tekniska rapporter, där vi sedan själva fyllde i våra fynd, kommandon, skärmdumpar och slutsatser.
- Vi använde AI för att snygga till språket och göra texterna mer lättlästa och enhetliga.
- Vi bollade möjliga CWE-klassificeringar och MITRE ATT&CK-tekniker med AI för att få uppslag på vilka kategorier som bäst beskrev våra sårbarheter.
- Vi tog även hjälp av AI som bollplank när vi strukturerade post-mortem-analysen av loggfilerna och när jag skulle utforma tydliga KPI:er anpassade för ledningsnivå.

I själva det praktiska penetrationstestet och letandet efter buggar i målmiljön har vi däremot i princip inte använt AI, utan där arbetade vi manuellt med `curl`, webbläsarens utvecklarverktyg och vanliga Linux-kommandon.

### När AI var lämpligt
AI fungerade väldigt bra när det gällde att spara tid kring struktur, formuleringar och överblick. När man har många tekniska anteckningar från flera veckors arbete är det lätt att texten blir spretig, och där var AI ett bra stöd för att få en röd tråd och anpassa tonen så att även en icke-teknisk mottagare kan förstå vad en sårbarhet innebär. Det var också användbart som ett första bollplank för att snabbt hitta kandidater bland CWE- och MITRE ATT&CK-ID:n som vi sedan kunde granska vidare själva.

### När AI var olämpligt eller otillräckligt
AI är olämpligt att lita på blint vid teknisk klassificering och praktisk exploatering:
- När vi bollade CWE- och MITRE ATT&CK-tekniker märkte vi att AI gärna föreslår en mappning även när det inte finns någon bra 1:1-koppling, eller blandar ihop en ren kodsårbarhet (som IDOR / CWE-639) med angripartekniker i ATT&CK som egentligen handlar om stulna konton (`T1078`).
- Under själva testerna mot målmiljön hade det varit direkt olämpligt att låta en AI generera och köra färdiga exploit-kommandon utan att vi själva förstod varje del av kommandot, eftersom man då riskerar att bryta mot Rules of Engagement (RoE) eller orsaka oavsiktlig påverkan i miljön.

### Verifiering och mänsklig kontroll
Inga förslag från AI har tagits rakt av in i våra rapporter. När vi diskuterade CWE- och MITRE ATT&CK-mappningar gick vi alltid in på MITRE:s och CWE:s egna officiella hemsidor och läste beskrivningarna för att verifiera att tekniken faktiskt stämde med det vi gjorde i miljön. På samma sätt har alla loggutdrag, tidsstämplar, radnummer och kommandon kontrollerats direkt mot våra egna terminalutskrifter, skärmdumpar och råloggarna (`web.jsonl`, `db.jsonl` och `internal.jsonl`).

### Risker och professionella gränser
I en yrkesprofessionell säkerhetskontext finns det två stora risker med generativ AI som man alltid måste förhålla sig till:
1. **Sekretess och känslig data (TLP och kunddata):** Våra rapporter är klassade som `TLP:AMBER` och innehåller intern infrastruktur, exploit-vägar och i labbmiljön även fiktiva personuppgifter och lösenord. I ett skarpt kunduppdrag får man aldrig klistra in kundens privata källkod, riktiga lösenord/tokens, interna IP-adresser, personuppgifter (GDPR) eller opublicerade sårbarheter i en publik AI-tjänst, eftersom man då tappar kontrollen över var informationen hamnar och riskerar att bryta mot sekretessavtal (NDA) och lagkrav.
2. **Hallucinationer och ansvarsfrågan:** AI kan formulera svar som låter mycket säkra och professionella även när de innehåller sakfel. Om en säkerhetskonsult levererar en rapport med felaktiga rekommendationer eller kör ett skadligt kommando är det alltid konsulten själv – aldrig AI-verktyget – som bär det fulla professionella ansvaret gentemot kunden och ledningen.

### Professionell slutsats
Min slutsats är att AI är ett utmärkt hjälpmedel i säkerhetsarbetet när det används som stöd för struktur, sammanfattning, rapportskrivning och idégenerering, men att det kräver strikt informationssäkerhet (ingen känslig kunddata i externa verktyg) och att en människa med teknisk förståelse alltid verifierar fakta och står för bedömningarna.

---

## 8. Prioriterade rekommendationer

#### Prioritet 1 (Omgående): Säkra inloggning och införa serverbaserad behörighetskontroll
- **Vad som bör göras:** Bygg om inloggningsfunktionen så att den använder parametriserade SQL-frågor (prepared statements) och ta bort tilliten till den osignerade rollcookien `role=admin`. Inför serverbaserad sessions- och behörighetskontroll på samtliga skyddade sidor (inklusive `/admin` och `/profile`).
- **Varför (koppling till evidens och risk):** Våra tester visade att vem som helst kan logga in som admin med `' OR '1'='1'` eller nå `/admin` genom att skicka med `-b "role=admin"`. Detta är den enklaste vägen in för en angripare och måste stängas direkt för att stoppa obehörig åtkomst.
- **Förväntad effekt och uppföljning:** Obehöriga utestängs från administrationsgränssnittet och andra användares profiler. Följs upp genom ett nytt test av inloggning och cookies samt genom KPI 1 (Åtgärdstid för kritiska sårbarheter).

#### Prioritet 2 (Omgående): Stänga möjligheten till kommandoexekvering och begränsa serverrättigheter
- **Vad som bör göras:** Ta bort skal-exekveringen i `/admin/diag` och ersätt den med ett säkert API eller strikt validering som endast tillåter giltiga IP-adresser. Ta bort SUID-rättigheter på systembinärer som `/usr/bin/find`, kör webbapplikationen med ett lågprivilegierat tjänstekonto och kräv autentisering mot det interna API:et `/export`. Flytta samtidigt ut lösenord och nycklar ur `secret_config.py` och Git-historik till en Secret Manager och strama åt brandväggsreglerna i `main.tf`.
- **Varför (koppling till evidens och risk):** Kommandoinjektionen i `/admin/diag` gjorde det möjligt för oss att läsa ut databaslösenordet, hämta anställdas personnummer och löner från `/export` och eskalera till `root` för att läsa `/root/flag5.txt`. Genom att både ta bort kommandokörningen och begränsa rättigheterna minskar vi konsekvensen (blast radius) drastiskt.
- **Förväntad effekt och uppföljning:** Även om en sårbarhet skulle uppstå i webbapplikationen i framtiden kan angriparen varken köra systemkommandon, läsa `/root/` eller hämta ut personuppgifter och nycklar. Följs upp via KPI 2 (Least Privilege) och KPI 3 (Exponerade hemligheter).

#### Prioritet 3 (Kort till medellång sikt): Införa central loggövervakning, automatiska larm och säkerhetskontroller i CI/CD
- **Vad som bör göras:** Koppla loggarna (`web.jsonl`, `db.jsonl`, `internal.jsonl`) till en central övervakningslösning med automatiska larm vid misstänkta SQL-fel, obehöriga anrop mot `/admin` och `/export` samt oväntade kommandokörningar. Lägg in automatisk secret scanning och konfigurationskontroll i CI/CD-pipelinen före driftsättning.
- **Varför (koppling till evidens och risk):** Post-mortem-analysen visade att alla våra fem angrepp loggades men att ingen märkte dem i realtid. Samtidigt visade Blue Team-fasen att hemligheter och felkonfigurationer i Terraform lätt hamnar i Git om de inte fångas automatiskt.
- **Förväntad effekt och uppföljning:** Förkortad tid till upptäckt vid intrångsförsök och färre sårbarheter som når produktion. Följs upp genom simulerade larmtester och månadsvis granskning av KPI 1–3.

---

## 9. Individuell reflektion

### Vad kan jag nu som jag inte kunde när kursen började?
- Jag har fått en mycket tydligare helhetsbild av hur olika sårbarheter hänger ihop i en **attackkedja** – hur ett till synes enkelt fel i en cookie eller ett inloggningsfält kan kedjas vidare till kommandoinjektion, åtkomst till interna API:er i ett Docker-nätverk, molnresurser och till slut `root`-behörighet på servern.
- Jag har också lärt mig hur man använder **CWE och MITRE ATT&CK** tillsammans på ett genomtänkt sätt: att CWE beskriver själva den tekniska svagheten i koden medan ATT&CK beskriver hur angriparen faktiskt agerar, och hur man översätter tekniska fynd och KPI:er så att en ledningsgrupp kan fatta beslut utifrån dem.

### Vilket moment har förändrat mitt sätt att arbeta med cybersäkerhet mest?
Kombinationen av att först arbeta i **Blue Team** (hitta brister i kod, Git-historik och Terraform-infrastruktur) och sedan göra **Red Team med efterföljande post-mortem i loggarna**. Att först själv genomföra attackerna mot målmiljön och sedan öppna `web.jsonl`, `db.jsonl` och `internal.jsonl` för att se exakt hur våra egna kommandon såg ut ur försvararens perspektiv gav en riktig aha-upplevelse kring hur viktigt det är att loggar inte bara samlas in, utan faktiskt övervakas och larmas på.

### Vad skulle jag göra annorlunda om projektet började idag?
Jag skulle föra ännu mer strukturerade anteckningar med exakta tidsstämplar, kommandon och skärmdumpar direkt i samma sekund som vi testar något. När vi satt med post-mortem-analysen och skulle matcha våra tester mot över 34 000 loggrader i `web.jsonl` märkte vi hur mycket enklare arbetet blir när man har dokumenterat exakt när och hur varje anrop gjordes från början.

### Vad tar jag med mig till LIA och min framtida yrkesroll?
- **Försvar på djupet (Defense in Depth) och Least Privilege i praktiken:** Man får aldrig lita på att en enskild säkerhetsgräns håller. Även om någon tar sig förbi inloggningen ska servern, tjänstekontona och de interna API:erna vara så strikt avgränsade att angriparen inte kommer längre.
- **Vikten av noggrann dokumentation och spårbarhet:** I yrkesrollen är ett penetrationstest eller en säkerhetsgranskning bara så bra som den rapport och de bevis man lämnar efter sig. Reproducerbara steg, tydliga loggbevis och respekt för Rules of Engagement (RoE) och TLP-klassificering är helt avgörande.
- **Att kunna tala både med utvecklare och med ledningen:** Utvecklare behöver konkreta tekniska beskrivningar (som vilken fil eller rad som är sårbar och hur den ska fixas), medan en ledningsgrupp behöver förstå vad bristen innebär för verksamheten, ekonomin och lagkrav som GDPR, samt vilka åtgärder som ger störst riskminskning först.

---

## 10. Källor och referenser
- [kpi.md](./kpi.md) – Mina tre säkerhets-KPI:er och samlad ledningsrapportering.
- [lessons-learned.md](./lessons-learned.md) – Post-mortem, tre lessons learned och professionell sammanfattning.
- **Team 6 – Penetrationstestrapport (Red Team)** – Teknisk dokumentation av rekognoscering, fynd 1–5, reproduktion, logganalys (`web.jsonl`, `db.jsonl`, `internal.jsonl`) och bilagor med skärmdumpar samt Rules of Engagement.
- **Team 6 – Säkerhetsrapport (Blue Team)** – MITRE ATT&CK-mappning, CWE-analys, Threat Intelligence-analys, TLP-klassificering och infrastrukturgranskning (`main.tf`).
- [MITRE ATT&CK Framework](https://attack.mitre.org/) – Referensramverk för angripartaktiker och tekniker (bl.a. T1190, T1059.004, T1059.006, T1213, T1005, T1552.001, T1552.005, T1530).
- [MITRE Common Weakness Enumeration (CWE)](https://cwe.mitre.org/) – Klassificering av tekniska sårbarheter (bl.a. CWE-78, CWE-89, CWE-94, CWE-200, CWE-312, CWE-540, CWE-565, CWE-639, CWE-798, CWE-862).
- [FIRST Traffic Light Protocol (TLP) v2.0](https://www.first.org/tlp/) – Standard för informationsdelning och klassificering (`TLP:AMBER`).
- [OWASP Top 10 / Authorization & Injection Prevention](https://owasp.org/www-project-top-ten/) – Riktlinjer för åtgärder mot Broken Access Control och Injection.

---

## Slutkontroll
- [x] Jag har tydligt beskrivit mitt eget bidrag och länkat till relevant underlag.
- [x] Mina viktigaste resultat stöds av evidens och källor.
- [x] Mina KPI:er är anpassade för en icke-teknisk beslutsfattare.
- [x] Mitt post-mortem leder till konkreta förbättringar och uppföljning.
- [x] Jag har beskrivit konkret AI-användning och självständigt värderat när AI är lämpligt eller olämpligt.
- [x] Jag har beskrivit hur AI-genererade resultat verifierades.
- [x] Mina rekommendationer följer av analysen och är prioriterade.
- [x] Den professionella sammanfattningen fungerar fristående.
- [x] Min individuella reflektion innehåller konkreta exempel.
- [x] Källor, länkar och referenser fungerar.
- [x] Inga lösenord, tokens, API-nycklar, personuppgifter eller andra skyddsvärda uppgifter finns i repositoryt.
- [x] Alla ändringar är committade och pushade till GitHub.
