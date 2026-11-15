# KPI:er och säkerhetsmätningar

**Namn:** Sebastian Ilia  
**Projekt/team:** Team 6  
**Datum:** 2026-11-14

## Utgångspunkt

Jag har valt tre områden som hör ihop med projektet: säker konfiguration, loggning av angrepp och oavsiktlig exponering mot internet. KPI:erna är förslag på hur säkerheten kan följas upp. Målvärdena är inte uppmätta resultat från våra tester.

Gruppens rapporter och repository ger exempel på brister och loggspår. Underlaget innehåller däremot inte en fullständig kravlista, testförteckning eller inventering av aktuell drift. Därför redovisas inga beräknade nulägesvärden. Blue Team-miljön och Red Team-målet ska följas upp separat.

## KPI 1 – Andel godkända konfigurationskontroller

### Syfte och mottagare

IT-chefen och systemägaren behöver veta om miljön följer beslutade säkerhetskrav. I projektet finns exempelvis inställningar för inloggning, autentisering i leveransflödet och brandväggsregler. Måttet hjälper ansvariga att se vilka inställningar som behöver förbättras.

### Definition och beräkning

`Antal godkända kontroller / totalt antal planerade kontroller i den fastställda kontrollistan × 100`.

En kontroll gäller ett bestämt krav på en bestämd resurs. Listan och omfattningen fastställs före mätningen. Varje kontroll får status godkänd, underkänd eller ej kontrollerad. En ej kontrollerad punkt räknas inte som godkänd och antalet sådana punkter visas separat. Om ingen lista finns anges ”saknar underlag”; om inga kontroller är tillämpliga anges ”ej tillämpligt”.

### Datakälla och datakvalitet

Terraform-kod, pipelinekonfiguration, beslutade säkerhetskrav och inställningar i den aktuella molnmiljön. Koden visar avsedd konfiguration men behöver jämföras med drift, där manuella ändringar kan ha gjorts. En kodgranskning är därför inte ensam bevis för att miljön följer kraven.

**Nuläge:** Kan inte beräknas. Projektunderlaget saknar en komplett kontrollista och verifiering av aktuell drift. Blue Team-rapportens infrastrukturfynd kan användas för att välja relevanta kontroller.

### Målvärde eller önskad riktning

Andelen godkända kontroller ska öka mot 100 procent. Kritiska avvikelser ska alltid uppmärksammas, även om totalvärdet är högt. Målet är ett förslag för uppföljningen.

### Mätfrekvens

Efter relevanta konfigurationsändringar och vid en månadsvis jämförelse mellan kod och drift.

### Tolkning och åtgärd vid avvikelse

Ett lågt värde innebär att krav inte är uppfyllda eller att verifiering saknas. Plattformsteamet behöver skilja dessa orsaker åt, prioritera kritiska avvikelser och dokumentera åtgärderna. Ett högt värde gäller bara kontrollistan och bevisar inte att hela systemet är säkert.

## KPI 2 – Andel testade angreppsscenarier med spårbara loggar

### Syfte och mottagare

Säkerhetsansvarig och IT-chefen behöver veta om relevanta angrepp lämnar användbara spår. Gruppens post-mortem visar att loggar kan användas för att undersöka händelser i efterhand. Måttet följer upp den förmågan.

### Definition och beräkning

`Antal genomförda scenarier med verifierade loggspår / totalt antal giltigt genomförda scenarier i den beslutade testomgången × 100`.

Varje scenario får ett id och räknas en gång, oavsett antal loggrader. Före testet bestäms vilka händelser som ska loggas. Ett godkänt loggspår ska kunna kopplas till scenariot genom tidpunkt, berörd komponent och den testade handlingen. Om testförteckning eller nödvändiga loggar saknas anges ”saknar underlag”. Om inga scenarier genomförts anges ”ej tillämpligt”.

### Datakälla och datakvalitet

Testprotokoll samt webb-, databas- och tjänsteloggar. Tidszoner, klockor och testperioder behöver stämma överens. Rapportutdrag kan visa att vissa spår finns men inte att samtliga genomförda tester loggades. Loggning ska också skiljas från automatiska larm och faktisk incidenthantering.

**Nuläge:** Kan inte beräknas säkert. Red Team-rapportens avsnitt 6 beskriver loggspår från fem angreppstyper, men underlaget räcker inte för att kontrollera täckningen för hela testomgången.

### Målvärde eller önskad riktning

Alla prioriterade scenarier ska lämna de loggspår som beslutats i förväg. Det motsvarar ett föreslaget mål på 100 procent för just dessa scenarier, inte för alla möjliga angrepp.

### Mätfrekvens

Vid godkända uppföljningstester och efter ändringar som påverkar loggningen. Befintliga resultat kan sammanställas månadsvis.

### Tolkning och åtgärd vid avvikelse

Vid saknade spår ska övervakningsansvarig undersöka logginställningar, insamling och lagring. Därefter verifieras förbättringen. Även vid full loggtäckning behöver organisationen separat kontrollera om viktiga händelser ger larm och om någon hanterar dem.

## KPI 3 – Antal oavsiktligt internetexponerade resurser

### Syfte och mottagare

Systemägaren och IT-chefen behöver veta om tjänster kan nås från internet utan att detta är avsett och godkänt. Blue Team-underlagets breda brandväggsregler gör området relevant. Måttet stödjer beslut om vilka åtkomstvägar som behövs och vilka som bör stängas.

### Definition och beräkning

`Antal unika resurser med verifierad åtkomst från internet som saknar godkänt verksamhetsbehov eller bryter mot beslutad åtkomstpolicy`.

Varje resurs räknas en gång; berörda portar och tjänster anges i underlaget. En publik IP-adress räcker inte ensam för att avgöra om resursen är åtkomlig eller felaktigt exponerad. Bedömningen behöver omfatta trafikväg, brandvägg, tjänst och beslutad åtkomst. Resurser som ännu inte kontrollerats redovisas separat. Vid ofullständig inventering får ett observerat antal inte presenteras som ett fullständigt totalvärde.

### Datakälla och datakvalitet

Molnplattformens resursförteckning, Terraform, brandväggsregler, godkänd åtkomstlista och resultat från tillåtna åtkomstkontroller. Konfiguration visar möjlig exponering men måste jämföras med faktisk drift och verksamhetens behov.

**Nuläge:** Ett fullständigt antal kan inte fastställas. En bred regel i koden är skäl att kontrollera exponeringen, men är inte ensam bevis för hur många resurser som faktiskt kan nås från internet.

### Målvärde eller önskad riktning

Målet är noll oavsiktligt exponerade resurser. Avsiktligt publika tjänster ska fortfarande ha ändamålsenligt skydd och ingår i den övriga riskbedömningen.

### Mätfrekvens

Efter nätverksändringar och vid månadsvis inventering av miljön.

### Tolkning och åtgärd vid avvikelse

Varje bekräftad oavsiktlig exponering behöver riskbedömas och få en ansvarig. Plattformsteamet bör begränsa åtkomsten och kontrollera resultatet. Nödvändiga funktioner, exempelvis jumphostens trafikförmedling, behöver beaktas så att åtgärden inte bryter avsedd drift. Noll bekräftade fynd betyder inte att okontrollerade resurser är säkra.

## Samlad ledningsrapportering

Jag har valt de här tre KPI:erna eftersom projektet innehåller både konfiguration av en molnmiljö och dokumenterade angrepp med loggspår. Måtten besvarar tre frågor: följer miljön säkerhetskraven, kan angreppen följas i loggarna och finns åtkomst från internet som inte borde finnas? De ger ledningen konkreta områden att följa utan att behöva läsa varje teknisk inställning.

En positiv utveckling är fler godkända konfigurationskontroller, bättre loggtäckning och färre oavsiktligt exponerade resurser. Vid försämring kan IT-chefen prioritera tid för konfigurationsarbete, logginsamling eller åtkomstbegränsning. Kritiska avvikelser behöver hanteras även när ett sammanlagt procentvärde ser bra ut. Varje rapportering bör därför visa både antal, omfattning och allvarliga kvarstående brister.

Underlaget räcker ännu inte för fullständiga nulägesvärden. Första steget är att fastställa kontrollista, testförteckning och resursinventering samt utse ansvariga för uppföljningen. Saknad information ska redovisas tydligt. Den ska inte räknas som att allt är godkänt eller som att inga brister finns.

KPI:erna beskriver inte hela säkerhetsläget. De visar exempelvis inte om alla applikationsbrister är åtgärdade eller om någon faktiskt reagerar på ett larm. Därför behövs även fyndrapporter, godkända omtester och dokumenterad incidenthantering. Blue Team-miljön och Red Team-målet ska redovisas var för sig så att resultat från olika miljöer inte blandas ihop.

## Källor och arbetsunderlag

- Team 6, *Blueteam team 6.pdf*, särskilt avsnitt 9–10: rekommendationer och infrastrukturfynd.
- Team 6, *Redteam team 6 (1).pdf*, avsnitt 6: post-mortem och loggutdrag.
- [Team 6:s repository, granskad version](https://github.com/ITSX25-Chas/blueteam-team6/tree/6e3449fb85e83555014bc812ef80e6a457e9a9ed): konfiguration och pipelines.
- Kursens instruktion och KPI-mall: struktur och krav på uppföljning.
- Definitioner, avgränsningar och formuleringar har bearbetats med AI-stöd. Målvärdena är förslag och inga uppmätta resultat har konstruerats.

AI-användningen beskrivs närmare i [den individuella slutrapporten](./individual-report.md#7-ai-i-säkerhetsarbetet).
