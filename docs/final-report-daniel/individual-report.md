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
I penetrationstestet granskade vi bland annat hanteringen av inlogg, behörigheter och least privilege. Det mest kritiska vi hittade var att det inte fanns några hinder att komma åt systemet utifrån. Det finns flera sätt att göra detta på.
De tre största sårbarheterna är att sessionerna och identiteten hanteras hos klienten, att inloggningsfunktionen går kringå för lätt, och att det går komma åt ett verktyg utifrån utan någon identitetskontroll. Den fjärde största sårbarheten är att administrationsverktyget kan användas för att exekvera kod på servern. Tillsammans gör dessa sårbarheter att systemet är väldigt känsligt för yttre påverkan. Det kan leda till dataförluster och att företaget tappar kontrollen över systemet.
Det första som måste göras är att åtgärda åtkomsten. Inloggningsverktyget måste bli strikt och hindras från att göra SQL-anrop. Sessionerna måste flyttas från klienten till servern. Sedan bör företaget se över administrationsverktyget. Om det finns skäl att tillåta åtkomst utifrån, med korrekt identitetshantering, så ska verktyget inte tillåtas köra kod på servern.
Åtgärderna för att stänga åtkomsten för obehöriga måste genomföras omgående, och följas upp så fort de införts. Administrationsverktyget bör kopplas ifrån tills beslut tagits om det ska gå att nå utifrån, eller tills det byggts om och inte längre har rätt att köra kod på servern.
I fortsättningen rekommenderas företaget att implementera automatiska säkerhetsskanningar i CI/CD-flödet så att kritiska sårbarheter upptäcks och hindras från att komma i produktion.

---

## 7. AI i säkerhetsarbetet
Beskriv och värdera din användning av generativa AI-assistenter och AI-assisterade säkerhetsverktyg i projektets arbetsflöde.
### Konkreta användningsfall
Gemini för att lära mig mer om KPI:er.
### När AI var lämpligt
I mitt fall var det en enkel utväg för att förstå KPI:er bättre. Jag hittade ett par artiklar med förslag på flera KPI:er som jag tyckte passade, och bad om en förklaring av några stycken.
### När AI var olämpligt eller otillräckligt
Jag har försökt använda AI fyra gånger under utbildningens gång. Den här uppgiften var den fjärde.
Den första var i en tidigare kurs där jag på grund av yttre omständigheter behövde få ihop något fort. Det blev inte bra, och jag var inte nöjd. Jag försökte faktakolla det jag fick, men trots att jag inte hittade det som AI:n påstod så använde jag det för att jag kände att jag inte hade tid.
Andra gången var i en tidigare kurs där vi satte upp en enklare AI-skanning i Wazuh. Det är ett bra användningsområde.
Den tredje gången var under Blue Team-delen av denna kurs. Då satt jag i drygt tre timmar och gick i cirklar med att försöka hitta en flagga med hjälp av AI innan jag gav upp.
AI bör inte användas under tidspress, för det tar tid att kontrollera uppgifterna och göra ändringar. Det måste ifrågasättas hela tiden. Jag anser att jag oftast hittar korrekt information snabbare på egen hand än med "hjälp" av en AI.
### Verifiering och mänsklig kontroll
Jag gick igenom det AI:n påstod, och jämförde noga med artiklarna jag hittat så att det verkligen stämde överens. Om någonting hade skilt mellan artikeln och vad AI:n påstod, så hade jag gått på artikeln.
### Risker och professionella gränser
Båda de tidigare gångerna jag använt AI under utbildningen så har den hallucinerat. Den har påstått saker som jag enkelt kan se är felaktigt.
Jag vet inte säkert, men det är väldigt troligt att flera i klassen tyvärr inte granskar det de får av en AI. Det är även möjligt att de har lagt upp hela infrastrukturkoden i en AI, och i det läget kan känslig data ha läckt.
Man kan säga att AI är ett bra verktyg om det används på rätt sätt, men det är väldigt svårt att hålla sig inom "rätt" gräns. Däremot är det lätt att bara slänga in en prompt och ta det man får tillbaka.

### Professionell slutsats
AI har i en tidigare kurs varit ett väldigt användbart verktyg i cybersäkerhet, specifikt när vi tränade en enklare AI att upptäcka avvikelser i nätverkstrafik. Skanningen kunde upptäcka händelser snabbt och antingen larma eller avvärja eventuella hot.
I övrigt har mitt användningsområde bestått av chatt-bottar, och för mig har det inte fungerat bra att använda AI. Svaren jag fått från dem har inte varit användbara. Jag gissar att orsaken är en blandning av att jag inte promptar på rätt sätt, och att AI har inbyggda säkerhetsramar som hindrar vissa svar när det handlar om känsliga ämnen som cybersäkerhet. AI-bolagen har ett ansvar att inte lära ut hacking på ett skadligt sätt.

#### AI för automation och kodstöd
Automatiska AI-skanningar kan hjälpa till att säkra system. Utan dessa måste människor kontrollera loggar med jämna mellanrum, och därefter reagera på incidenter. En AI kan automatiskt blockera trafik och låsa ner system om det ser ut som att ett angrepp pågår. Det minskar responstiden och den mänskliga faktorn.
Människor bör fortfarande kontinuerligt granska AI:n för att säkerställa att den håller sig inom bestämda ramar.

Att använda chatt-bottar som Gemini, ChatGPT, Claude och Copilot som stöd när man till exempel skriver kod, stavningskontroll, och behöver enklare vägledning är även det lämpliga användningsområden.

#### AI och känslig data
AI ska aldrig användas för att hantera känslig data. Det kan vara personuppgifter, lösenord, säkerhetsnycklar och liknande. Vi som användare har ingen kontroll eller översikt över var sådan data tar vägen efter att en AI får tag i den. Den kan användas för att träna AI, eller användas av AI-bolaget på ett otillåtet sätt.

Om man använder AI till att skriva kod finns även en risk att man hårdkodar in nycklar och lösenord i kod som sedan blir publik. Det är viktigt att lära sig hur man använder säkerhetshantering med till exempel GitHubs Secrets Management.

#### Kvalitetssäkring och ansvar
Om man använder chatt-bottar så måste man granska precis allt. Om en AI påstår någonting, gör en sökning på det och dubbelkolla. Det är okej att inte kunna en sak, men när en AI säger något så vet du vad du ska lära dig mer om utanför AI. Det är väldigt vanligt att AI hittar på saker om den inte kan svaret. Som användare vet du inte om det du får ut stämmer eller inte. Chatt-bottar skriver även ofta fel i kod, och det kan vara besvärligt att felsöka när det inte är du själv som skrivit koden.

När det gäller automatiska verktyg så är det viktigt att granska resultat med jämna mellanrum. Om verktyget tränas kontinuerligt så finns det en risk att en angripare långsamt ökar trafiken, och att verktyget då tror att det är normalt och inte larmar. Det kan även behöva justeras för trafik som är ovanlig men påvisat säker. Det måste alltid finnas en människa som tar besluten.

I båda fallen är det viktigt att komma ihåg att allt ansvar för det som levereras till kund eller företagsledning ligger på den som levererar, och då även ansvaret för att arbetet är kvalitetssäkrat.

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

#### Prioritet 1: Åtgärda att inloggningen går kringgå
I nuläget går det att logga in som admin med en enkel SQL-injektion. Inloggningen är oerhört osäker. Det gör företaget sårbart för intrång från inte bara främmande makt och hacker-grupper, utan även från ungdomar som leker runt oansvarigt och kan orsaka skada.

Utvecklarna måste omedelbart införa indatavalidering i inloggningsformuläret och parametriserade frågor i SQL-anropet.

Det här är en så pass kritisk sårbarhet att det inte går vänta, det måste göras nu.

#### Prioritet 2: Flytta sessions- och identitetshanteringen till serversidan
Servern litar på att det klienten påstår stämmer utan verifiering. Det gör så att klienten kan manipulera värdet som anger klientens roll, och därmed få behörigheter utan att vara inloggad.

DevOps måste flytta hanteringen av sessioner och identitet från klienten till servern, och införa validering vid varje anrop.

Även detta måste ha högsta prioritet. Det är marginellt svårare att utföra en attack på det här sättet än genom inloggningen då angriparen antingen måste vara inloggad på webbtjänsten eller ha tillgång till vissa endpoints, men det är ändå en kritisk sårbarhet.

#### Prioritet 3: Begränsa administrationsverktygets tillgång till systemet
Administrationsverktyget är en stor sårbarhet, men av lägre prioritet än de två första. Om de två första punkterna åtgärdas så är det svårare för en angripare att utnyttja sårbarheten i verktyget. Det bör ändå prioriteras högt.

Verktyget ska ändå inte ha möjlighet att exekvera kod mot servern. Vid ett intrång där angriparen får tillgång till verktyget så riskerar företaget att systemet tas över då verktyget möjliggör att skadlig kod körs på servern. Utvecklarna av verktyget måste därför bygga om det så att det istället använder till exempel en API för att utföra sina uppgifter. Om verktyget absolut måste vara kopplat till systemet så behöver det hindras från att exekvera kod. Det ska endast kunna göra den administrative uppgift den är gjord för.

---

## 9. Individuell reflektion
### Vad kan jag nu som jag inte kunde när kursen började?
- Jag förstår nu bättre varför behörighetskontroll ska vara på serversidan och inte hos klienten. När man skapar en webbtjänst så ska man inte lita på klienterna. Det har dessutom fått mig att börja kolla på ett gammalt projekt där jag har inloggning, och som jag vill sätta upp igen och testa säkerheten på.
- Efter att ha läst på om SQL-injektioner så vill jag testa även det på mitt gamla projekt som nyss nämnts.

### Vilket moment har förändrat mitt sätt att arbeta med cybersäkerhet mest?
[Förklara varför.]
Blue Team. Vi fokuserade visserligen mest på att leta flaggor, men målmiljön patchades under tiden och det blev svårare att komma vidare. Det var kul att bli utmanad lite mer varje vecka.

### Vad skulle jag göra annorlunda om projektet började idag?
Mycket bättre och tydligare dokumentation. Vi tog oss igenom Red Team för snabbt utan att dokumentera ordentligt, och därför saknar vi till exempel dokumentation om när och vad vi gjorde i penetrationstestet. Det leder till en sämre slutrapport.

### Vad tar jag med mig till LIA och min framtida yrkesroll?
- Det behöver oftast inte gå så fort när man testar och granskar. Det är viktigare att vara noggrann, tydlig och dokumentera allt man gör och allt man hittar. Om man skyndar sig igenom en uppgift så kommer man missa saker. Det märkte vi när vi skulle skriva rapporten till Red Team. Vi saknade dokumentation om när vi utförde attacker och kunde därför inte granska loggarna så väl som i alla fall jag hade önskat.
- Om utvecklarna eller DevOps slarvar med indatavalidering eller behörigheter så sätts säkerheten helt ur spel nästan direkt. Det har vi sett med inloggningen i först Blue Team och sedan Red Team. Den var lätt att kringå, och anledningen var att formuläret skrevs på ett av (vad jag hittat) två sätt. Sättet som använts i inloggningen ser enklare ut, så det kan vara så att utvecklare skriver inloggningar på det sättet utan att förstå hur det kan utnyttjas. Om vi hamnar på ett företag med utvecklare så bör vi diskutera såna saker med dem.
- Att kommunicera med ledningsgrupp och klienter är svårt. Då de inte alltid är på samma tekniska nivå som utvecklare eller säkerhetsteamet måste man försöka undvika tekniska termer och istället hålla sig på en enkel nivå. Men man måste ändå få fram poängen så att beställaren förstår, och kunna visa på sårbarheter, risker, åtgärder och vilka resurser som krävs. Det är viktigt både för att kunna säkra upp systemen och för att faktiskt få de resurser som behövs i arbetet.
- Det är viktigt att lyssna eller läsa och förstå kravspecifikationer och Rules Of Engagement. När det gäller kravspecifikationer så måste man säkerställa att man uppfyller det beställaren vill ha. Har man Rules of Engagement så får man inte gå utanför den, och om man går utanför måste man stoppa direkt.
- Se till att alltid lära mig någonting nytt. Utvecklingen inom cybersäkerhet går framåt hela tiden, och det är viktigt att hänga med när något förändras. Gör jag mig tillgänglig för extra uppgifter så mycket som möjligt så lär jag mig mer om rollen. Jag kan dessutom få uppgifter som inte hör till rollen, och därför lära mig om andra intressanta saker. Det är viktigt att vara öppen för sådana möjligheter.

---

## 10. Källor och referenser
Lista de källor du faktiskt har använt, exempelvis:
- standarder och ramverk,
- kursmaterial,
- teknisk dokumentation,
- loggar och projektartefakter,
- AI-genererat material när det har påverkat analysen eller texten.
Använd ett konsekvent sätt att ange källorna. Det ska gå att skilja mellan externa fakta, projektets evidens, AI-genererade förslag och dina egna bedömningar.
[MITRE ATT&CK](https://attack.mitre.org/)
Ett ramverk för att förstå hur angripare attackerar system.
[CWE](https://cwe.mitre.org/index.html)
Ramverk med kända sårbarheter.
[CISA: TLP](https://www.cisa.gov/news-events/news/traffic-light-protocol-tlp-definitions-and-usage)
CISAs förklaring av hur TLP fungerar.

[LinkedIn: Cloud Security Key Performance Indicators...](https://www.linkedin.com/pulse/cloud-security-key-performance-indicators-kpis-risk-kris-singh-4jtkc/)
En artikel om KPI:er som jag tyckte hjälpte mycket.
[Reddit: What KPIs have you found to be useful...](https://www.reddit.com/r/cybersecurity/comments/16a5dqr/what_kpis_have_you_found_to_be_useful_when/)
Reddit-tråd med förslag på KPI:er.
[Trustview: Compliance-KPI:er](https://trustview.se/akademi/compliance-kpier/)
Sida med förklaringar av KPI:er på svenska.

[SQL injection - Authentication](https://medium.com/@elkassimiabderrahmane2001/sql-injection-authentication-21caedf5e4cb)
En artikel som förklarar SQL-injektioner.
[SQL injection](https://developer.mozilla.org/en-US/docs/Glossary/SQL_Injection)
En artikel som förklarar SQL-injektioner.
[W3: Prevent SQL Injection](https://www.w3schools.com/sql/sql_parameterized_queries.asp)
Artikel som visar hur man förhindrar SQL-injektioner.

[kpi.md](./kpi.md)
KPI:erna jag har valt ut.
[lessons-learned.md](./lessons-learned.md)
Lärdomar från projektet.
[RedTeam6.pdf](./RedTeam6.pdf)
Teamets rapport från Red Team.

Källor och referenser utan länk eller utanför GitHub-repot:
Blue Team-rapporten för Team 6
Tidigare utbildare har visat hur man bygger ett kommando med en payload.
Malcolm tipsade om att modifiera kakorna.

---

## Slutkontroll
- [x] Jag har tydligt beskrivit mitt eget bidrag och länkat till relevant underlag.
- [x] Mina viktigaste resultat stöds av evidens och källor.
- [x] Mina KPI:er är anpassade för en icke-teknisk beslutsfattare.
- [x] Mitt post-mortem leder till konkreta förbättringar och uppföljning.
- [x] Jag har beskrivit konkret AI-användning och självständigt värderat när AI är
lämpligt eller olämpligt.
- [x] Jag har beskrivit hur AI-genererade resultat verifierades.
- [x] Mina rekommendationer följer av analysen och är prioriterade.
- [x] Den professionella sammanfattningen fungerar fristående.
- [x] Min individuella reflektion innehåller konkreta exempel.
- [x] Källor, länkar och referenser fungerar.
- [x] Inga lösenord, tokens, API-nycklar, personuppgifter eller andra skyddsvärda
uppgifter finns i repositoryt.
- [x] Alla ändringar är committade och pushade till GitHub.