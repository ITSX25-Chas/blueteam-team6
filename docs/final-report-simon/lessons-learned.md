# Post-mortem, lessons learned och professionell sammanfattning

## Händelse och avgränsning
I det här dokumentet har jag valt att fokusera på hur en angripare efter en första ingång i webbapplikationen kan komma åt interna personuppgifter, eskalera sina rättigheter till högsta systemnivå (`root`) och varför inget av detta stoppades trots att angreppen syntes i målmiljöns loggar.

Under vårt penetrationstest i Red Team undersökte vi webbapplikationen på `10.0.0.9:8080`, dess bakomliggande databas, ett internt API (`http://internal:8000/export`) samt själva Linux-servern som tjänsten kördes på. Samtliga tester hölls inom överenskomna Rules of Engagement utan att ändra eller skada målmiljön. Efter testerna genomförde vi en post-mortem-analys där vi granskade försvarssidans loggar (`web.jsonl`, `db.jsonl` och `internal.jsonl`) för att se vilka spår vi lämnat efter oss.

Arbetet genomfördes i samarbete inom Team 6, där jag bidrog aktivt både i det praktiska testandet:
- Jag var med och kartlade applikationens sidor och testade hur inloggning, vanliga användarkonton och profilsidor (`/profile?id=1`) fungerade, vilket hjälpte oss att förstå hur behörigheter och sessioner hanterades i applikationen.
- Jag var med och testade hur administrationssidan och diagnostikverktyget (`/admin/diag`) kunde nås genom att skicka med en egen rollcookie (`role=admin`) och hur vi via SQL-injektion kunde logga in som administratör.
- När vi väl nådde diagnostikverktyget var jag med och testade kommandoinjektion för att läsa ut `secret_config.py`, anropa det interna API:et `/export` med personuppgifter.

---

## Lesson learned 1
### Observation
Exponering av hemligheter i konfigurationsfiler och oskyddade interna API:er leder till läckage av känsliga personuppgifter. Genom att läsa filen `secret_config.py` på webbservern hittade vi både databaslösenordet i klartext och adressen till ett internt API (`http://internal:8000/export`). När vi anropade denna adress inifrån servern fick vi direkt ut anställdas namn, roller, löner och personnummer utan någon inloggning eller behörighetskontroll. Samma typ av problem såg vi även under Blue Team-fasen där hemligheter och nycklar fanns kvar i Git-historik, dolda branches och konfigurationsfiler.
### Orsak
Applikationen är byggd utifrån antagandet att allt som ligger på insidan av servern eller det interna nätverket är säkert. Därför har man hårdkodat känsliga uppgifter direkt i källkodsfiler och låtit det interna API:et lämna ut personuppgifter till alla anrop som kommer inifrån miljön utan autentisering.
### Konsekvens
Om en angripare hittar en enda väg in i webbapplikationen faller skyddet för resten av de interna tjänsterna direkt. Läckage av personnummer och löneuppgifter innebär en mycket allvarlig integritetsrisk för de anställda och kan för företaget leda till anmälningsplikt och höga sanktionsavgifter enligt GDPR, utöver förlorat förtroende.
### Förbättring
Ta bort alla lösenord, nycklar och känsliga konfigurationer ur källkoden och hantera dem i en säker hemlighetshanterare (Secret Manager). Inför krav på autentisering och behörighetskontroll även mellan interna tjänster och API:er, och begränsa hur mycket persondata som kan exporteras i ett och samma anrop.
### Uppföljning
Inför automatisk skanning efter hemligheter (secret scanning) vid varje kodändring i CI/CD-pipelinen (kopplat till KPI 3) och genomför en säkerhetsgranskning av samtliga interna API-endpoints innan de driftsätts.

---

## Lesson learned 2
### Observation
Bristande rättighetsbegränsning på servern gjorde det möjligt att gå från en vanlig webbapplikation till fullständiga administratörsrättigheter (`root`) på operativsystemnivå. Genom diagnostikfunktionen på hemsidan kunde vi inte bara köra kommandon som webbserverns användare, utan även använda systemverktyget `/usr/bin/find` (med växeln `-exec cat {} \;`) för att läsa filen `/root/flag5.txt` i serverns mest skyddade katalog.
### Orsak
Principen om minsta möjliga behörighet (Least Privilege) har inte följts. Dels tillät diagnostikverktyget att användarinmatning skickades direkt till operativsystemets skal utan validering, och dels hade systemverktyget `find` förhöjda rättigheter (SUID/root-behörighet) som gjorde att kommandon som kördes via `find` fick administratörsrättigheter på hela servern.
### Konsekvens
Istället för att ett intrång stannar i själva webbapplikationen kan en angripare ta över hela den underliggande servern. Med `root`-åtkomst kan angriparen läsa och ändra samtliga filer på maskinen, installera bakdörrar, radera eller manipulera lokala loggar och använda servern som språngbräda mot andra delar av infrastrukturen.
### Förbättring
Bygg om diagnostikfunktionen så att den aldrig anropar operativsystemets skal med användarinmatning, och kräv serverbaserad behörighetskontroll istället för en enkel klientcookie (`role=admin`). Kör webbapplikationen med ett strikt begränsat tjänstekonto och ta bort onödiga förhöjda rättigheter (SUID-bitar) på systemprogram som `find`.
### Uppföljning
Kontrollera regelbundet att inga tjänstekonton eller systembinärer har högre behörigheter än nödvändigt (kopplat till KPI 2) och verifiera genom ett uppföljande test att webbapplikationen inte längre kan nå filer utanför sin egen katalog.

---

## Lesson learned 3
### Observation
Samtliga fem angreppssteg vi genomförde registrerades tydligt i målmiljöns loggar. När vi i efterhand analyserade `web.jsonl`, `db.jsonl` och `internal.jsonl` (totalt över 34 000 loggrader i den ursprungliga miljön) såg vi klara bevis på både SQL-injektionen, kommandona mot `/admin/diag`, anropet mot `/export` och läsningen av `/root/flag5.txt`, men inga larm eller motåtgärder hade aktiverats.
### Orsak
Målmiljön hade passiv loggning påslagen men saknade automatisk övervakning, larmregler och aktiv incidentrespons. Loggarna sparades lokalt i filer utan att något system analyserade avvikande beteenden, såsom databasfel vid inloggning eller att webbservern plötsligt körde shell-kommandon och läste filer under `/root/`.
### Konsekvens
Utan aktiva larm ger loggarna bara nytta i efterhand när man ska utreda vad som redan har gått fel. I ett skarpt läge innebär det att en angripare kan prova sig fram, stjäla personuppgifter och ta över servern under lång tid utan att säkerhetsteamet märker att ett intrång pågår.
### Förbättring
Koppla loggarna från webbserver, databas och interna tjänster till en central logghanteringslösning (SIEM) och sätt upp automatiska larm vid tydliga angreppsmönster, till exempel upprepade SQL-fel, obehöriga anrop till `/admin` och `/export` eller om webbserverprocessen startar ett systemskal.
### Uppföljning
Testa larmreglerna praktiskt genom simulerade anrop efter att de har införts för att säkerställa att säkerhetsteamet faktiskt får en varning direkt när misstänkt aktivitet uppstår.

---

## Professionell sammanfattning
Under vårt penetrationstest och den efterföljande logganalysen granskade vi hur väl företagets webbtjänst och bakomliggande server står emot ett riktat intrångsförsök, samt hur god förmåga miljön har att upptäcka ett pågående angrepp.

Resultatet visar att miljön har flera allvarliga säkerhetsbrister som i kombination gör det möjligt för en utomstående angripare att ta sig hela vägen från startsidan till företagets mest skyddsvärda data. De tre viktigaste slutsatserna för ledningen är:
- **Enkel väg in till administratörsfunktioner:** Brister i både inloggningen och behörighetskontrollen gör att en angripare snabbt kan få administratörsåtkomst utan giltigt konto.
- **Känsliga personuppgifter och hela servern ligger oskyddade på insidan:** Väl inne i administrationsverktyget kunde vi köra kommandon direkt på servern, hämta ut en lista med anställdas personnummer och löner från ett internt system och slutligen få högsta möjliga behörighet (`root`) på servern.
- **Angreppen loggas men ingen larmas:** Vår genomgång av loggarna efter testet visade att alla våra steg syntes tydligt i loggfilerna, men eftersom det saknas automatiska larm kunde hela angreppet genomföras ostört.

För verksamheten innebär detta en akut risk för såväl driftstopp och förlorad kontroll över IT-miljön som allvarliga personuppgiftsincidenter med rättsliga och ekonomiska konsekvenser (GDPR) samt skadat anseende.

Företaget bör omgående säkra inloggningen och flytta behörighetskontrollen till servern så att obehöriga stängs ute. Samtidigt behöver administrationsverktyget stängas av eller byggas om, de interna personuppgifterna och lösenorden skyddas, och automatiska larm införas så att säkerhetsansvariga får veta direkt om någon försöker ta sig in.

---

## Kontroll före commit
- [x] Jag har avgränsat vilket intrångsförsök eller vilken incident mitt post-mortem gäller.
- [x] Jag har beskrivit min egen roll i incidentresponsen.
- [x] Jag har formulerat tre tydliga lessons learned.
- [x] Jag skiljer på observation och bakomliggande orsak.
- [x] Jag beskriver konsekvensen av varje lärdom.
- [x] Varje lesson learned leder till en konkret förbättring och uppföljning.
- [x] Den professionella sammanfattningen fokuserar på resultat, risk och betydelse.
- [x] Texten är begriplig utan alla tekniska detaljer från projektet.
