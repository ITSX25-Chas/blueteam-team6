# Individuell slutrapport

**Namn:** Sebastian Ilia  
**GitHub-konto:** Sebastianilia7  
**Projekt/team:** Team 6 – Blue Team och Red Team  
**Datum:** 2026-11-14  
**Underlag:** Gruppens dokumenterade resultat och min egen beskrivning av deltagandet. AI-stödet beskrivs i avsnitt 7.

## 1. Professionell sammanfattning

Projektunderlaget omfattar en koddefinierad molnmiljö, dokumenterade säkerhetsfynd och ett penetrationstest mot en separat tilldelad webbmiljö. Granskningen visar att gruppen har använt Terraform och automatiserade kontroller samt konfigurerat förbättringar för åtkomst och CI/CD-autentisering. Detta kan styrkas i repositoryt. Att lösningen fungerar i drift och att alla identifierade brister är åtgärdade kan däremot inte fastställas från koden ensam.

Red Team-rapporten visar en kedja från bristande behörighetskontroll till kommandoexekvering och åtkomst till intern information. Den visar också att angreppsrelaterad aktivitet finns i loggar. Automatiska larm och incidentåtgärder är inte verifierade i materialet. Organisationens viktigaste förbättringsbehov är därför både förebyggande skydd och spårbar uppföljning: stäng de dokumenterade angreppsvägarna och kontrollera att försvarsfunktionerna reagerar.

Tre KPI:er föreslås: andel godkända konfigurationskontroller, andel testade angreppsscenarier med spårbara loggar och antal oavsiktligt internetexponerade resurser. De ska hjälpa ledningen att prioritera resurser och följa resultat. Samtliga numeriska nulägesvärden är ännu okända eftersom fullständiga kontrollistor, testförteckning och aktuell resursinventering saknas. Detta är en databegränsning, inte ett bevis för att alla kontroller misslyckats eller att exponering saknas.

Systemägaren bör utse ansvariga, besluta tidsplan och säkerställa ett godkänt åtgärds- och omtestfönster. Applikationsskydd och begränsad åtkomst till interna resurser bör prioriteras tillsammans med verifierad detektion. Rapporten skiljer mellan gruppens dokumenterade resultat och mitt bekräftade deltagande i gemensamt flaggletande under Blue Team. Egen medverkan i Red Teams incidentrespons har inte kunnat preciseras och tillskrivs mig därför inte.

## 2. Kontext och avgränsning

Arbetet bygger på Team 6:s Blue Team-projekt och efterföljande Red Team-dokumentation. Blue Team-underlaget omfattar ett GitHub-repository med Terraform, pipelines och ändringshistorik samt en säkerhetsrapport med tio flaggfynd och infrastrukturella observationer. Red Team-underlaget beskriver fem fynd och en efteranalys av loggar från en tilldelad webbmiljö. Miljöerna ska inte behandlas som en och samma installation utan ytterligare underlag.

Källorna är gruppens rapporter, skärmdumpar, återgivna loggutdrag, RoE, repositoryversionen `6e3449f` och ett begränsat chattutdrag. Originalens loggfiler, aktuell molndrift och lyckade pipelinekörningar har inte kontrollerats. Inga nya tekniska tester har genomförts för slutrapporten. Chattutdraget ger stöd för samarbetskontext men bevisar inte vem som utförde varje tekniskt steg. Rapportens uppgift är att omsätta befintliga resultat till uppföljning och beslut, med tydliga begränsningar där evidensen inte räcker.

## 3. Mitt individuella bidrag

Under Blue Team deltog jag i flaggletandet och i gruppens möten. Vi gick in på olika sidor i labbmiljön och undersökte dem tillsammans. Jag försökte själv hitta saker under letandet. Vid ett tillfälle hittade jag något som andra i gruppen hade hittat precis före mig. Jag minns inte exakt vilket fynd det var, så jag kopplar det inte till en bestämd flagga eller sårbarhet i rapporten.

Vi hade möten ofta och hjälptes åt när någon fastnade. Jag bidrog också till att ordna ett möte inför inlämningarna. I gruppchatten den 11 november frågade jag om vi skulle ha ett nytt möte och föreslog torsdag efter lektionen. Den 12 november föreslog jag klockan 13. Detta är ett konkret exempel på hur jag deltog i samordningen, utan att ha ansvar för hela projektet.

Gruppens rapport beskriver våra samlade resultat. Jag tillskriver mig inte andra deltagares tester eller kodändringar. Mitt GitHub-konto är `Sebastianilia7`. Inför rapporteringen har jag samlat material och bett ChatGPT om hjälp att förstå uppgiften. Jag kan inte säkert ange vilka Red Team-tester jag själv utförde; resultaten därifrån redovisas som gruppens dokumenterade arbete.

## 4. Viktigaste projektresultat och evidens

### Resultat 1 – Infrastruktur och vissa säkerhetskontroller finns dokumenterade som kod

**Resultat:** Repositoryt definierar två instanser, nätverksresurser och trafikvägar. OS Login är konfigurerat på båda instanserna. Deploy-workflow använder Workload Identity Federation. En historisk ändring tar bort en Terraform-resurs för en service account-nyckel.

**Evidens:** [main.tf vid granskad version](https://github.com/ITSX25-Chas/blueteam-team6/blob/6e3449fb85e83555014bc812ef80e6a457e9a9ed/main.tf), [deploy-workflow](https://github.com/ITSX25-Chas/blueteam-team6/blob/6e3449fb85e83555014bc812ef80e6a457e9a9ed/.github/workflows/deploy.yml), commits [49ed83c](https://github.com/ITSX25-Chas/blueteam-team6/commit/49ed83ccc92498f0f3ec137720de23ee0f122a09) och [fabd32e](https://github.com/ITSX25-Chas/blueteam-team6/commit/fabd32ea8cf3a1dea9af98ae575a09e4e7193e10).

**Verksamhets- och säkerhetsbetydelse:** Versionshanterad konfiguration ger ett spårbart underlag för granskning och återuppbyggnad. Det är däremot inte bevis för att ändringarna är driftsatta eller att gamla nycklar har återkallats. Brandväggsregeln är fortfarande brett tillåtande i granskad kod.

**Rekommendation:** Plattformsteamet bör verifiera driftstatus, faktisk åtkomst och nyckelstatus samt granska brandväggsregeln. Prioritet P1 för verifiering av åtkomst och exponering. Jumphostens routingfunktion ska beaktas så att nödvändig trafik inte bryts av en ogenomtänkt härdning.

### Resultat 2 – Webbbrister kunde kombineras till åtkomst till intern information

**Resultat:** Red Team-rapporten beskriver åtkomst till administration, kommandoexekvering, konfigurationsläsning, intern datahämtning och läsning av en fil under `/root/`.

**Evidens:** *Redteam team 6 (1).pdf*, avsnitt 4–5 och bilagor 2–6, sidorna 20–23. Skärmdumparna visar flera testinmatningar och resultat. Fullständig rotorsaksanalys av filrättigheter och processidentitet saknas.

**Verksamhets- och säkerhetsbetydelse:** Brister i en ingång kan möjliggöra åtkomst till flera informationsresurser. Flera flaggor ska inte tolkas som lika många oberoende rotorsaker. Läsning av filen under `/root/` visar inte ensam hur rättigheter erhölls.

**Rekommendation:** Applikationsansvarig och plattformsteam bör prioritera serverbaserad behörighetskontroll, säker indatahantering och begränsad process- och tjänsteåtkomst. Prioritet P1, med omtest av hela kedjan inom ett godkänt fönster efter att åtgärderna genomförts. De tekniska åtgärdsprinciperna stöds av OWASP-källorna i avsnitt 10.

### Resultat 3 – Loggspår finns, men aktiv detektion är inte verifierad

**Resultat:** Gruppens post-mortem beskriver spår från fem angreppstyper. Automatisk larmning och incidentåtgärder kunde inte verifieras.

**Evidens:** *Redteam team 6 (1).pdf*, avsnitt 6, sidorna 13–15. Detta är återgivna utdrag och gruppens analys; originalfilerna har inte granskats här. Antalen loggrader från två miljöer räcker inte för att jämföra deras säkerhetsnivå.

**Verksamhets- och säkerhetsbetydelse:** Organisationen har visst underlag för efteranalys men kan inte bedöma förmågan att upptäcka och hantera ett pågående intrång utifrån dessa uppgifter.

**Rekommendation:** Övervakningsansvarig bör samla larmunderlag och koppla definierade scenarier till larm, mottagare och hantering. Prioritet P1 för att klarlägga detektionskedjan och besluta om nödvändiga förbättringar.

## 5. KPI:er och ledningsrapportering

De fullständiga definitionerna finns i [kpi.md](./kpi.md).

| KPI | Ledningsfråga | Beslut som måttet kan stödja |
|---|---|---|
| Andel godkända konfigurationskontroller | Följer miljön våra beslutade säkerhetskrav? | Prioritera kritiska avvikelser och kontroll av drift. |
| Andel testade angreppsscenarier med spårbara loggar | Går det att undersöka de testade angreppen i efterhand? | Förbättra insamling och kvalitet på loggar. |
| Antal oavsiktligt internetexponerade resurser | Finns åtkomst från internet som inte borde finnas? | Begränsa åtkomst och granska verksamhetsbehov. |

Måtten knyter projektets konfiguration och loggspår till beslut som en IT-chef kan följa upp. Målen är bättre kravuppfyllelse, spårbar loggning av prioriterade scenarier och noll oavsiktliga exponeringar. Inget av dessa mål presenteras som ett redan uppnått resultat.

Fullständiga nulägesvärden saknas. Första steget är att fastställa kontrollista, testförteckning och aktuell resursinventering. Miljö, tidsperiod och okontrollerade delar behöver alltid redovisas. Måtten ersätter inte omtester av applikationsbrister eller kontroll av larm och incidenthantering. Loggspår visar exempelvis inte att någon fick ett larm och reagerade på det.

## 6. Incidentrespons, post-mortem och förbättringsarbete

Analysen i [lessons-learned.md](./lessons-learned.md) gäller Red Teams dokumenterade angreppskedja. Gruppen beskriver bristande behörighetskontroll och kommandoinjektion som gav vidare åtkomst. Angreppsspår identifierades i loggar i efterhand. Det finns inte tillräckligt underlag för att beskriva en genomförd kedja av larm, triage, begränsning och återställning.

Tre slutsatser är centrala. Behörighet måste kontrolleras även när klienten manipuleras. En komprometterad applikations åtkomst till andra resurser behöver begränsas. Logginsamling måste slutligen kopplas till verifierad larmning och ansvarig hantering. Förbättringarna följs upp genom omtester och kontroll av loggspår, larm och ansvarig hantering. KPI:erna kompletterar detta med regelbunden uppföljning av konfiguration och extern exponering.

En organisatorisk lärdom finns också i gruppchatten: i oktober diskuterades oklarheter om vad som skulle lämnas in och behovet av ett gemensamt möte. Det stödjer behovet av en gemensam leveranslista med ansvar och evidens. Det bevisar inte att arbetet saknade planering eller att ett möte aldrig hölls. Samma försiktighet behövs när min egen roll beskrivs: efteranalysen ska inte framställas som min ursprungliga incidentrespons.

## 7. AI i säkerhetsarbetet

### Konkreta användningsfall

Jag bad ChatGPT om hjälp för att förstå uppgiften. Vid rapportförberedelsen har jag även använt ChatGPT/Codex för struktur och formulering. Jag lämnade uppgiftsbeskrivningar, mallar och gruppens rapportmaterial. AI hjälpte till att formulera textutkast, föreslå KPI:er och sammanställa lärdomar utifrån underlaget. Jag har själv beskrivit min medverkan och förtydligat att mitt bidrag var deltagandet i flaggletandet och gruppmötena. Det tekniska resultatet kommer från gruppens arbete och dokumentation. AI-stödet har inte ersatt de tester som rapporterna beskriver.

### När AI var lämpligt

AI var användbart när jag behövde få överblick över flera olika inlämningar. Det hjälpte mig att skilja Blue Team, Red Team och slutarbetets tre filer åt. Ett annat exempel är KPI-delen, där AI föreslog hur gruppens fynd skulle kunna följas upp. Förslagen fick definitioner, datakällor och mål i stället för att bara bli en lista med säkerhetsbegrepp. Det gjorde innehållet mer strukturerat, men måtten behöver fortfarande kunna förklaras och motiveras av mig.

### När AI var olämpligt eller otillräckligt

AI kunde inte avgöra exakt vilka tekniska steg jag själv hade utfört. Den informationen måste komma från mig och från dokumentation av arbetet. AI kunde inte heller räkna fram korrekta KPI-värden när fullständiga kontrollistor, testförteckning och inventering av aktuell drift saknades. Därför står värdena som okända. Att en formulering låter övertygande är inte tillräckligt för att den ska användas som ett resultat eller som bevis för min arbetsinsats.

### Verifiering och mänsklig kontroll

Jag har kontrollerat beskrivningen av mitt eget bidrag och korrigerat den så att den inte omfattar andra deltagares ansvar. Assistenten har jämfört delar av rapportmaterialet med repositoryts kod och granskat bildbilagorna. Den har också kontrollerat rekommendationer mot OWASP:s vägledningar, som finns i källförteckningen. Detta ska skiljas från min egen tekniska verifiering: jag har inte genomfört nya tester för slutrapporten. En viktig avgränsning är att loggspår inte används som bevis för automatiska larm. Rapporteringen skiljer också mellan föreslagna åtgärder och åtgärder vars effekt har kontrollerats.

### Risker och professionella gränser

AI kan misstolka projektmaterial, blanda ihop miljöer eller skriva mer säkert än underlaget tillåter. Det kan även göra det svårt att se vilka slutsatser som kommer från en källa och vilka som är förslag. Därför behöver källor och begränsningar framgå. I en yrkesroll behöver jag även följa organisationens regler för vilken information som får lämnas till ett AI-verktyg. Lösenord, tokens och andra känsliga uppgifter ska inte delas utan stöd i godkända rutiner. Personliga uppgifter som inte behövs för analysen bör lämnas utanför.

### Professionell slutsats

Jag ser AI som ett stöd för struktur, förklaringar och formulering. Ansvaret för det jag lämnar in ligger fortfarande hos mig. Jag behöver kunna skilja mellan gruppens resultat, min egen medverkan och AI:s förslag. Jag har också äldre ChatGPT-chattar från Blue Team som inte har gåtts igenom här. Den här redogörelsen beskriver därför främst stödet vid rapportförberedelsen och gör inga detaljerade påståenden om AI-användningen under de tidigare labbarna.

## 8. Prioriterade rekommendationer

Prioriteringarna är förslag för en fiktiv ledningsgrupp. De är inte instruktioner att ändra gruppens miljö nu.

| Prioritet | Åtgärd och motiv | Föreslagen ägare | Verifiering |
|---|---|---|---|
| P1 | Fastställ godkänd målmiljö, kvarstående brister och åtgärdsfönster. RoE och rapport innehåller oklarheter som måste lösas innan nya tester. | Systemägare och testansvarig | Samstämmig scopebeskrivning och godkänt underlag. |
| P1 | Åtgärda behörighetsbrister, SQL-injektion och kommandoinjektion. De öppnar de dokumenterade angreppsvägarna. | Applikationsansvarig | Omtest av både otillåtna och legitima användningsfall. |
| P1 | Begränsa åtkomst till interna tjänster, filer och nätverk efter verksamhetsbehov. Granska faktisk drift och brandvägg utan att bryta gatewayfunktionen. | Plattformsteam och tjänsteägare | Dokumenterad åtkomstmatris och godkända funktionstester. |
| P1 | Verifiera kopplingen mellan test, logg, larm och ansvarig hantering. | Övervakningsansvarig | Loggspår enligt KPI 2 samt separat verifiering av larm och mottagare. |
| P2 | Inför kontrollista, testförteckning och resursinventering för KPI:erna samt ett fyndregister med ägare och omtestbevis. | Systemägare och respektive tekniskt ansvarig | Underlag för alla tre KPI:er och separat uppföljning av åtgärder. |

## 9. Individuell reflektion

### Vad kan jag nu som jag inte kunde när kursen började?

Jag har lärt mig lite mer om flaggletande genom att själv vara med och leta i labbmiljön. Vid ett tillfälle hittade jag något som andra i gruppen hade hittat precis före mig. Jag minns inte vilket fynd det var, men det är ett exempel på att jag också provade själv och kom fram till ett resultat. Att någon annan hann före ändrar inte att jag fick öva på letandet.

När rapporten skulle skrivas märkte jag också hur svårt det var att i efterhand beskriva de exakta stegen. Jag behöver därför dokumentera bättre och fortsätta träna på att förklara vad jag gör, inte bara komma fram till ett resultat.

### Vilket moment har förändrat mitt sätt att arbeta med cybersäkerhet mest?

Det gemensamma flaggletandet är det moment jag främst kan utgå från. Vi hade möten ofta och nästan alla i gruppen brukade vara med. När någon fastnade hjälptes vi åt. För mig var det lättare att komma vidare tillsammans med gruppen än att försöka lösa allt själv. Det är den erfarenhet jag tydligast vill ta med mig i mitt fortsatta arbete. Samtidigt behöver jag själv förstå det vi gör, även när vi löser en uppgift tillsammans.

### Vad skulle jag göra annorlunda om projektet började idag?

När jag skulle sammanställa arbetet märkte jag att det var svårt att komma ihåg exakt vad jag själv hade gjort. Om jag gjorde om projektet skulle jag skriva korta anteckningar efter varje arbetspass. Då skulle jag lättare kunna följa mitt eget arbete och se vad jag behövde förstå bättre. Anteckningarna skulle till exempel kunna innehålla vilken sida vi undersökte, vad vi provade och vilket resultat vi fick. Jag skulle då ha något konkret att återvända till när det var dags att skriva rapporten.

### Vad tar jag med mig till LIA och min framtida yrkesroll?

Jag vill ta med mig två arbetssätt: att dokumentera löpande och att samarbeta när en uppgift är svår. Korta anteckningar skulle hjälpa mig att minnas vad jag gjort och hålla reda på frågor som återstår. Erfarenheten från gruppen visar också att det hjälper att kunna ta stöd av andra när man fastnar. Under LIA vill jag kunna fråga kollegor när jag behöver hjälp och bidra när jag själv kan hjälpa någon annan. Det är arbetssätt jag vill utveckla vidare, samtidigt som jag fortsätter bygga min tekniska förståelse.

## 10. Källor och referenser

### Projektets evidens

1. Team 6, *Säkerhetsrapport – MITRE ATT&CK-mapping, Threat Intelligence-analys och TLP-klassificering*. Tillhandahållen som *Blueteam team 6.pdf*, särskilt avsnitt 4, 9 och 10. Beskriver gruppens fynd och rekommendationer, inte fullständig verifiering efter åtgärd.
2. Team 6, *Redteam team 6 (1).pdf*, 23 sidor, testperiod 19–30 oktober 2026. Avsnitt 4–6, bilaga 1 och bilagor 2–6. Tillhandahållen PDF, visuellt granskade bildbilagor. Gruppens originalrapporter har lämnats in separat i Canvas under Blue Team respektive Red Team.
3. [Team 6:s repository](https://github.com/ITSX25-Chas/blueteam-team6), granskad commit `6e3449fb85e83555014bc812ef80e6a457e9a9ed`, hämtad 2026-11-12. Fil- och commitlänkar finns vid resultat 1.
4. Gruppchattutdrag tillhandahållna av Sebastian: 18 augusti för kontonamn, 17 september för Tailscale-kontext, 9–10 oktober för diskussion om inlämning samt 11 november kl. 10.00–10.02 och 12 november kl. 10.03 för egna förslag om gruppmöte. De daterade meddelandena stödjer samordningsbidraget. Uppgiften om det egna letandet kommer från min egen beskrivning, inte från chatten. Chattens bilder och övriga bilagor har inte granskats. Hela chatten återges inte här.

### Kursmaterial och extern vägledning

5. Kursens instruktion *Avslutande arbete: från säkerhetsarbete till professionell rapportering med KPI:er* samt mallarna `kpi-3.md`, `lessons-learned-3.md` och `individual-report-3.md`.
6. [Praetorian: Cybersecurity Metrics & KPIs](https://www.praetorian.com/security-101/cybersecurity-metrics-and-kpis/), läst 2026-11-12. Använd för mätprinciper, inte för att styrka projektresultat. Övriga delade KPI-länkar anges inte som lästa källor.

### Teknisk vägledning för rekommendationerna

- [OWASP: SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html). Parameteriserade frågor skiljer inmatade värden från SQL-koden.
- [OWASP: Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html). Behörighet ska kontrolleras vid varje anrop till skyddade resurser.
- [OWASP: OS Command Injection Defense](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html). Undvik direkt kommandoexekvering när en avgränsad biblioteksfunktion kan användas.

Dessa källor kontrollerades av AI-assistenten 2026-11-14. De ger stöd för åtgärdsförslagen, inte bevis för att åtgärderna genomförts i projektet. Kontrollen utfördes med AI-stöd och ska inte likställas med egna omtester.

### AI-stöd och författarens bedömning

7. Generativ AI-assistent i Codex, arbetsdialog 2026-11-12–2026-11-14. Stöd för struktur, källjämförelse, KPI-förslag och utkast. AI-text är inte en oberoende källa till projektets händelser. Prioriteringar och KPI-mål är förslag för fortsatt uppföljning. KPI-definitionerna har bearbetats med AI-stöd. Mina personliga erfarenheter bygger på min egen beskrivning, medan projektresultaten bygger på gruppens dokumentation.
