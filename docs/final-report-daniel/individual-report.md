# Individuell slutrapport
**Namn:** Daniel Wagenius
**Projekt/team:** Team 6
**Datum:** 2026-11-13

---

## Syfte och kursmål
I rapporten ska du sammanställa projektets resultat och dina egna reflektioner för en fiktiv ledningsgrupp. Du ska omsätta tekniskt säkerhetsarbete till mätbar
uppföljning, förbättringsarbete och prioriterade rekommendationer. Du ska också
värdera när AI är lämpligt eller olämpligt i professionellt säkerhetsarbete.
Rapporten knyter an till:
- **Kursmål 4:** säkerhetsmätningar, KPI:er och compliance-rapportering till
ledningsnivå.
- **Kursmål 9:** incidentrespons, post-mortem och förbättringsarbete.
- **Kursmål 10:** KPI-rapportering för en icke-teknisk beslutsfattare.
- **Kursmål 11:** integrerad användning och professionell värdering av generativ AI och AI-assisterade säkerhetsverktyg.
- **Kursmål 13:** självständigt leda och leverera ett komplext säkerhetsprojekt med kvalitet, dokumentation och professionell kommunikation.
Bedömningen utgår från målgruppsanpassning, källanvändning, struktur, motivering
och din förmåga att visa förståelse för helheten.

---

## 1. Professionell sammanfattning
Skriv rapportens sammanfattning sist, men placera den först. Den ska kunna läsas
fristående av exempelvis en CISO, IT-chef, systemägare eller ledningsgrupp.
Sammanfatta:
- projektets syfte och viktigaste resultat,
- mest betydelsefulla risker och förbättringsområden,
- vad KPI:erna visar eller bör följa upp,
- viktigaste rekommendationerna,
- vilka beslut eller prioriteringar som behövs.
**Omfattning:** cirka 200–300 ord.

---

## 2. Kontext och avgränsning
Vårt projekt var ett penetrationstest mot en målmiljö med en webbtjänst och dess tillhörande databas. Testet utfördes för att undersöka webbtjänstens säkerhet och motståndskraft, och hade även potential att visa på regulatoriska brister.
Vi gick igenom webbtjänstens användargränssnitt för att försöka hitta en väg in. När vi väl vara inne kunde vi utforska serverns interna system med hjälp av administrationsverktyget som vi fick tillgång till.
Målmiljön var endast webbtjänsten som körde på IP-adressen vi blev tilldelade. Vi fick inte gå utanför den och röra andra teams målmiljöer. När vi hade tillgång fick vi inte göra några ändringar eller hindra driften.
Vi har byggt våra rapporter på resultatet av de tester vi utfört mot målmiljön och observationer av dessa.

---

## 3. Mitt individuella bidrag
Jag utsågs inofficiellt till gruppledare. Därför har jag lett teamets möten. Under våra möten har jag varit den som delat skärm och genomfört stora delar av penetrationstestet. Det var fördelaktigt när vi satte upp miljön, då det hindrade dubbelarbete. Därför är de flesta commits från mig. Jag fick ha hand om det mesta av skötseln på vår GitHub.
Under testet var skärmdelningen hjälpsam för att kunna demonstrera teknikerna för övriga gruppmedlemmar. Till viss del gjorde min skärmdelning att jag såg och visade flaggorna i målmiljön för de andra, även om någon hade utfört samma sak innan mig. Jag hittade bland annat en instruktion för att göra SQL-injektioner i inloggningsfältet.
Jag hoppas att det var värdefullt för övriga medlemmar och att de har förstått processen, och att de hade stoppat mig om det var något de inte förstod och behövde gå in djupare på.
När vi fastnade var det i huvudsak jag som sökte hjälp av utomstående när vi kände att vi verkligen inte kom vidare.
Anledningen till att jag fick en ledarroll har troligtvis med att göra att jag har tidigare utbildningar inom mjukvaruutveckling, kan programmera hyggligt och har använt GitHub en del.

---

## 4. Viktigaste projektresultat och evidens
### Bristande autentisering i inloggningsfunktionen
Penetrationstestet visade på stora brister i hur indatan behandlades i webbtjänstens inloggningsfunktion. Det är möjligt att kringgå inloggningen utan giltiga inloggningsuppgifter.
#### Evidens
Vi har i penetrationstestet demonstrerat hur administratörsinloggningen enkelt kan passeras med en välkänd SQL-injektionsattack i inloggningsformuläret.
Se [RedTeam6.pdf](./RedTeam6.pdf) med bilagor.
#### Verksamhets- och säkerhetsbetydelse
En angripare kan få full åtkomst till webbtjänsten med administratörsbehörigheter och all data som finns på webbtjänsten. Detta äventyrar hela systemets konfidentialitet och integritet.
Se [lessons-learned.md](./lessons-learned.md).
#### Rekommendation
Prioritet: Omgående, detta är en kritisk sårbarhet.
DevOps/utvecklingsteamet behöver bygga om inloggningsfunktionen med strikt indatavalidering och parametriserade databasfrågor så att injektioner blockeras.

### Avsaknad av åtkomstkontroll
Webbtjänsten saknar tillräcklig behörighets- och åtkomstkontroll på servern och litar blint på informationen från klienten.
#### Evidens
Vi har visat att en obehörig användare kan få administratörsrättigheter och komma åt skyddade funktioner genom att enbart manipulera rollvärdet i en kaka.
Se [RedTeam6.pdf](./RedTeam6.pdf) med bilagor.
#### Verksamhets- och säkerhetsbetydelse
Obehöriga kan manipulera en kaka och därmed eskalera sina behörigheter och komma åt känslig data och funktioner. Detta kan dessutom ske utan att gå via inloggningsfunktionen med hjälp av ett systemskal.
#### Rekommendation
Prioritet: Omgående. Detta är en kritisk sårbarhet.
DevOps/utvecklingsteamet måste flytta sessions- och behörighetshanteringen från klientsidan till servern, och införa användarverifiering vid varje anrop.

### Exponerad attackyta
Stora delar av företagets miljö är lättåtkomlig utifrån. Detta dessutom utan tillräcklig kontroll av indatan.
#### Evidens
Genom att anropa webbtjänsten från ett systemskal med verktyget *curl* och manipulera ett värde i en kaka kunde vi exekvera kommandon direkt på servern. Vi kunde se interna filer och eskalera våra privilegier.
#### Verksamhets- och säkerhetsbetydelse
En angripare kan ansluta från utsidan av företagets nätverk och ta över servern. Detta kan leda till bland annat driftstopp, ekonomiska förluster och dataläckage.
Se [lessons-learned.md](./lessons-learned.md).
#### Rekommendation
Prioritet: Omgående.
DevOps och driftansvariga måste begränsa synligheten och åtkomsten för känsliga delar av webbtjänsten. Det ska krävas VPN eller IP-vitlistning för att komma åt dessa delar. Administrationsverktyget som använts måste byggas om så att det inte kommer åt serverns system.

---

## 5. KPI:er och ledningsrapportering
Sammanfatta de tre KPI:erna från [`kpi.md`](./kpi.md). Beskriv:
- vilka KPI:er du valde,
- hur de kommunicerar säkerhetsstatus,
- varför de är relevanta för en icke-teknisk beslutsfattare,
- vilka beslut eller prioriteringar de kan stödja,
- eventuella begränsningar i data eller tolkning.
Du behöver inte upprepa alla fullständiga definitioner från `kpi.md`.

#### Säkerhetsefterlevnad i infrastruktur
Visar hur väl företagets system följer etablerade standarder och interna säkerhetskrav. Kan berätta om delar av ett system inte lever upp till kraven och behöver förbättras. Visar endast hur systemet är uppsatt när koden pushas. Förlorar i värde om systemet senare ändras utan att gå igenom granskningar.

#### Säkerhetsloggning och övervakning
Ger en bild av vad som övervakas och vilka händelser som loggas. Talar om ifall hotbilden ökar genom ökad loggintesitivitet, förutsatt att systemet i övrigt är säkert. Utan tillräcklig loggning är det svårt att upptäcka incidenter. Kan kombineras med automatiska larm och skyddsåtgärder. Kan dock logga för mycket så att det kritiska försvinner i mängden.

#### Publika och exponerade molnresurser
Ger en bild av företagets externa attackyta, det vill säga de delar av systemet som är mest utsatta. De resurser som är exponerade måste skyddas och säkras upp. De exponerade resurserna är ofta vägen in för angripare. Ju färre osäkra resurser det finns desto mindre risk att angrepp lyckas.

---

## 6. Incidentrespons, post-mortem och förbättringsarbete
Sammanfatta de viktigaste slutsatserna från [`lessons-learned.md`](./lessons-
learned.md).
Beskriv:
- vilket verkligt eller simulerat intrångsförsök som analyserades,
- hur incidenten upptäcktes och hanterades,
- de viktigaste orsakerna och konsekvenserna,
- vilka förbättringar som bör genomföras,
- hur förbättringarna bör följas upp.
Fokusera på vad organisationen behöver lära sig och förändra, inte bara på
händelseförloppet.


---

## 7. AI i säkerhetsarbetet
Beskriv och värdera din användning av generativa AI-assistenter och AI-assisterade
säkerhetsverktyg i projektets arbetsflöde.
### Konkreta användningsfall
[Vilka verktyg använde du, i vilka arbetsmoment och för vilket syfte? Ge konkreta
exempel.]
### När AI var lämpligt
[Beskriv minst ett konkret tillfälle där AI tillförde värde. Förklara varför AI var
lämpligt och vilken nytta användningen gav.]
### När AI var olämpligt eller otillräckligt
[Beskriv minst ett konkret tillfälle där AI inte borde användas ensamt, eller där
användningen krävde särskild försiktighet.]
### Verifiering och mänsklig kontroll
[Hur kontrollerade du AI-genererade förslag, analyser, kod eller slutsatser? Vilka
oberoende källor eller tester använde du?]
### Risker och professionella gränser
[Resonera om exempelvis hallucinationer, felaktiga säkerhetsråd,
informationsläckage, bias, bristande spårbarhet, upphovsrätt eller övertro på
resultat.]
### Professionell slutsats
[Formulera principer för hur du skulle använda, begränsa eller avstå från AI i
motsvarande yrkesmässigt säkerhetsarbete.]
**Viktigt:** En verktygslista räcker inte. Du ska självständigt motivera när AI är
lämpligt eller olämpligt och visa hur du säkerställde kvalitet och ansvar.
**Omfattning:** cirka 400–600 ord.

---

## 8. Prioriterade rekommendationer
Prioritera ett begränsat antal åtgärder utifrån projektresultat, KPI:er och post-
mortem.
För varje rekommendation ska det framgå:
- vad som bör göras,
- varför det bör göras,
- vilken risk eller möjlighet rekommendationen adresserar,
- föreslagen prioritet,
- vem eller vilken funktion som bör äga nästa steg.
Undvik en lång önskelista. Fokusera på åtgärder med tydlig säkerhets- och
verksamhetseffekt.

---

## 9. Individuell reflektion
### Vad kan jag nu som jag inte kunde när kursen började?
[Ge minst ett konkret exempel.]
### Vilket moment har förändrat mitt sätt att arbeta med cybersäkerhet mest?
[Förklara varför.]
### Vad skulle jag göra annorlunda om projektet började idag?
[Koppla svaret till något du faktiskt har upplevt eller lärt dig.]
### Vad tar jag med mig till LIA och min framtida yrkesroll?
[Beskriv minst två konkreta saker du kommer att kunna använda.]
**Omfattning:** cirka 300–500 ord totalt. AI-värderingen görs separat i avsnitt 7
och behöver inte upprepas här.

---

## 10. Källor och referenser
Lista de källor du faktiskt har använt, exempelvis:
- standarder och ramverk,
- kursmaterial,
- teknisk dokumentation,
- loggar och projektartefakter,
- AI-genererat material när det har påverkat analysen eller texten.
Använd ett konsekvent sätt att ange källorna. Det ska gå att skilja mellan externa
fakta, projektets evidens, AI-genererade förslag och dina egna bedömningar.

---

## Slutkontroll
- [ ] Jag har tydligt beskrivit mitt eget bidrag och länkat till relevant underlag.
- [ ] Mina viktigaste resultat stöds av evidens och källor.
- [ ] Mina KPI:er är anpassade för en icke-teknisk beslutsfattare.
- [ ] Mitt post-mortem leder till konkreta förbättringar och uppföljning.
- [ ] Jag har beskrivit konkret AI-användning och självständigt värderat när AI är
lämpligt eller olämpligt.
- [ ] Jag har beskrivit hur AI-genererade resultat verifierades.
- [ ] Mina rekommendationer följer av analysen och är prioriterade.
- [ ] Den professionella sammanfattningen fungerar fristående.
- [ ] Min individuella reflektion innehåller konkreta exempel.
- [ ] Källor, länkar och referenser fungerar.
- [ ] Inga lösenord, tokens, API-nycklar, personuppgifter eller andra skyddsvärda
uppgifter finns i repositoryt.
- [ ] Alla ändringar är committade och pushade till GitHub.