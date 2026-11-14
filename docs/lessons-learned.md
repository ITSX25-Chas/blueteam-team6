# Post-mortem, lessons learned och professionell sammanfattning

**Författare:** Sebastian Ilia, Team 6  
**Datum:** 2026-11-14  
**Underlag:** Efteranalys av gruppens dokumentation med AI-stöd. Min egen roll avgränsas nedan.

## Händelse och avgränsning

Detta post-mortem gäller den attackkedja som Team 6 beskriver i Red Team-rapporten för testperioden 19–30 oktober 2026. I den tilldelade webbmiljön dokumenterades bristande behörighetskontroll, SQL-injektion och kommandoinjektion. Administrativa funktioner kunde enligt rapporten användas för att läsa konfiguration, hämta uppgifter från en intern tjänst och läsa en fil under `/root/`. Bilagorna visar flera av dessa resultat.

Händelserna analyserades i efterhand med webb-, databas- och internserviceloggar. Rapporten visar angreppsspår men verifierar inte automatiska larm, kvittering eller incidentåtgärder under testet. Det går därför inte att rekonstruera en fullständig incidentrespons. Jag kan inte säkert beskriva vilka delar av Red Teams incidentrespons jag själv deltog i. Gruppens tester och ursprungliga logganalys redovisas därför som gruppens arbete. Detta dokument är en efteranalys av det delade underlaget. Blue Teams flaggletande och infrastruktur behandlas som separat projektkontext. Inga nya angrepp har genomförts för rapporten.

## Lesson learned 1 – Behörighetskontroll måste hålla även när klienten manipuleras

### Observation

Red Team-rapporten beskriver att en klientstyrd rollcookie gav åtkomst till administrationssidan. Bilaga 2 visar ett anrop med denna cookie och ett svar som innehåller den dokumenterade flaggan. Rapporten beskriver även åtkomst till en annan användares profil och en separat väg via SQL-injektion. Det är olika beteenden som behöver hållas isär vid verifiering.

### Orsak

Den dokumenterade cookie-vägen tyder på att servern accepterade en klientstyrd roll utan tillräcklig kontroll av användarens rättigheter. Detta är en bedömning utifrån testresultatet; den fullständiga serverimplementationen för Red Team-målet har inte granskats här. SQL-injektionen ska utredas och åtgärdas separat och inte förklaras som ett cookieproblem.

### Konsekvens

Administrativa funktioner blev en ingång till mer långtgående angrepp. För en verksamhet innebär det risk för obehörig informationsåtkomst och missbruk av funktioner som normalt ska vara begränsade till ett fåtal användare. Att en vanlig inloggningssida finns är därför inte tillräckligt som skydd.

### Förbättring

Applikationsansvarig bör kontrollera behörigheten på servern vid varje anrop till en skyddad funktion eller resurs. Detta följer [OWASP:s vägledning om behörighetskontroll](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html). Testfall bör omfatta oinloggad användare, vanlig användare, ändrad cookie och åtkomst till någon annans profil. SQL-injektionen behöver åtgärdas separat med parameteriserade frågor, där inmatade värden hålls skilda från SQL-koden. Se [OWASP:s vägledning om SQL-injektion](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html). Åtgärder och omtester ska ske efter ändringsstopp och med godkännande för den aktuella miljön.

### Uppföljning

Dokumentera förväntat och faktiskt svar för varje behörighetsfall. Kontrollera både att obehöriga nekas och att legitima användare fortfarande kan arbeta. Koppla godkända omtester till respektive fynd i ett åtgärdsregister. Dessa applikationstester kompletterar konfigurationskontrollerna i KPI 1; de ska inte räknas in utan att ingå i den fastställda kontrollistan i [kpi.md](./kpi.md).

## Lesson learned 2 – En applikationsbrist kan ge åtkomst till flera interna resurser

### Observation

Bilaga 4 visar inmatning i diagnostikfunktionen och resultat från läsning av en konfigurationsfil. Bilaga 5 visar hämtning från en intern tjänst. Bilaga 6 visar läsning av den dokumenterade filen under `/root/`. Dessa resultat stödjer att kommandoinjektionen kunde användas för flera steg i kedjan. Filens placering bevisar däremot inte ensam att en separat rättighetseskalering skedde.

### Orsak

Den rapporterade diagnostikfunktionen lät användarinmatning påverka kommandoexekvering. Processens åtkomst till konfiguration, interna tjänster och filer gjorde konsekvensen större. Exakt processidentitet, filrättigheter och mekanism för åtkomsten till `/root/` är inte fastställda i underlaget. Dessa ska inte ersättas av antagandet att angriparen automatiskt blev root.

### Konsekvens

En brist i en webbkomponent kunde påverka skyddet för information som låg bakom andra gränser. I en verklig verksamhet kan motsvarande kedja exponera känsliga data och göra återställning och riskbedömning mer omfattande. Flera flaggor innebär här inte nödvändigtvis flera oberoende orsaker; samma grundbrist kan ge flera konsekvenser.

### Förbättring

Applikationsansvarig bör undvika att skicka användarinmatning till ett systemskal och i första hand använda en avgränsad biblioteksfunktion för uppgiften. Se [OWASP:s vägledning om kommandoinjektion](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html). Plattformsteamet bör granska processrättigheter, filåtkomst och nätverksåtkomst till den interna tjänsten. Tjänsteägaren bör införa egen behörighetskontroll så att intern placering inte är det enda skyddet. Prioritera den gemensamma ingången och begränsa samtidigt följdskadorna.

### Uppföljning

Genomför godkända omtester av den dokumenterade kedjan, inte bara av en enskild inmatning. Spara bevis för att vanlig diagnostik fungerar och att läsning av konfiguration, intern export och skyddade filer stoppas. Spara verifieringen i fyndregistret med ansvarig och datum. Relevanta konfigurationskrav kan följas med KPI 1. KPI 3 kompletterar detta genom att följa exponering från internet, men mäter inte hela den interna attackkedjan.

## Lesson learned 3 – Loggspår behöver kopplas till larm och ansvar

### Observation

Rapportens avsnitt 6 innehåller tidsstämplade loggutdrag för de fem angreppstyperna. Samma avsnitt anger att automatiska säkerhetslarm och incidentåtgärder inte kunde verifieras. Det visar en gräns för vad materialet kan säga: aktivitet går att analysera i efterhand, men aktiv upptäckt under testet är inte styrkt.

### Orsak

Underlaget innehåller loggar men saknar kopplade larm-id, kvitteringar och incidentärenden. Det kan bero på att sådan dokumentation inte samlats in eller på att delar av detektionskedjan saknades. Det går inte att avgöra vilket utifrån rapporten. Skillnaden mellan dessa förklaringar behöver utredas innan organisationen bestämmer åtgärd.

### Konsekvens

Ledningen kan inte bedöma hur snabbt organisationen skulle ha reagerat. Det finns risk att logginsamling ger en överdriven bild av försvarsförmågan. Rapportens två loggmängder från olika miljöer kan inte jämföras som säkerhetsresultat utan samma tidsperiod, belastning och insamlingsomfattning.

### Förbättring

Övervakningsansvarig bör koppla prioriterade testscenarier till detektionsregler, mottagare och en enkel rutin för kvittering och eskalering. Säkerställ spårbarhet från test-id och tidpunkt till logg, larm och eventuell incidentåtgärd. Börja med att efterfråga befintliga larmunderlag innan slutsatsen dras att nya verktyg behövs.

### Uppföljning

Använd KPI 2 för att följa om definierade scenarier lämnar spårbara loggar. Kontrollera larm och hantering separat: dokumentera om larm uppstod, när det kom och vem som tog emot det. Redovisa saknade uppgifter som okända och gör godkända uppföljningstester där bevis saknas.

## Professionell sammanfattning

Team 6:s dokumenterade test visar hur brister i en webbapplikation kunde kombineras till åtkomst till administrativa funktioner, konfigurationsinformation och en intern tjänst. Resultaten stöds av rapportens skärmdumpar och loggutdrag. Den centrala risken är att ett inledande fel kunde få konsekvenser utanför den funktion där angreppet började. Enskilda åtgärder behöver därför bedömas mot hela den dokumenterade kedjan.

Tre förbättringsområden bör prioriteras. För det första behöver applikationens behörighetskontroll fungera även när klientdata manipuleras. För det andra behöver osäker kommandohantering tas bort och tjänstens åtkomst till andra resurser begränsas. För det tredje behöver loggning kopplas till verifierade larm, ansvariga mottagare och dokumenterad hantering. De befintliga loggarna visar spår av testerna men räcker inte för att säga att försvaret upptäckte och hanterade angreppen under testet.

Systemägaren bör besluta vem som ansvarar för varje åtgärd och när den ska vara verifierad. Applikationsansvarig bör äga kodåtgärderna, plattformsteamet åtkomstbegränsningarna och övervakningsansvarig larmkedjan. Förändringar i målmiljön ska respektera det avtalade ändringsstoppet och genomföras först i ett godkänt åtgärdsfönster.

Ledningen bör följa säker konfiguration, spårbar loggning och oavsiktlig exponering mot internet enligt de tre föreslagna KPI:erna. Måtten ska kompletteras med uppföljning av applikationsbrister och faktisk larmhantering. Fullständiga nulägesvärden saknas eftersom kontrollistor, testförteckning och aktuell resursinventering inte är kompletta. Nästa steg är att samla underlaget och besluta uppföljningen. Rapporten visar ett konkret förbättringsbehov, men stödjer inte ett påstående om att alla brister redan är lösta eller att någon viss individ genomförde samtliga moment.

## Källor och begränsningar

- Team 6, *Redteam team 6 (1).pdf*, avsnitt 4–6 och bilagor 2–6, tillhandahållen PDF med 23 sidor. Bilagorna granskades visuellt vid rapportens framtagning.
- RoE i samma PDF, bilaga 1: anger testfönster och ändringsstopp. Måladress, kontaktväg och kursansvarigs underskrift är inte ifyllda i visad version. Rapportens tunneladress skiljer sig dessutom från angiven måladress. Fullständig överensstämmelse med godkänd omfattning kan därför inte intygas utifrån dessa dokument ensamma.
- Loggutdrag har lästs i rapporten; originalfilerna har inte granskats oberoende. Ett HTTP-svar med status 200 bevisar inte ensamt att en exploatering lyckades.
- OWASP:s tre vägledningar ovan kontrollerades av AI-assistenten 2026-11-14. De stödjer rekommendationerna, inte påståenden om vad som faktiskt genomfördes i labben.
- Kursens mall för lessons learned. AI-assistent har hjälpt till med analys, avgränsning och språk. Detta är inte en oberoende verifiering av de ursprungliga testerna.
