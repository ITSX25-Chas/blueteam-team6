# Post-mortem, lessons learned och professionell sammanfattning

**Namn:** Julia Persson
**Projekt/team:** Team 6 – Blue Team och Red Team – ITSX25 Avancerad IT-säkerhet
**Datum:** 2026-11-15

---

## Syfte och kursmål

Den här delen är ett post-mortem över det simulerade intrånget i Red Team-testet. Den tar också med lärdomar från Blue Team-arbetet. Målet är att visa vad som hände och varför det hände. Den visar också vad organisationen bör ändra.

Arbetet knyter an till kursmål 9 och 13.

---

## Händelse och avgränsning

**Vad simulerades?** Team 6 genomförde ett penetrationstest mot en intern webbportal mellan 19 och 30 oktober 2026. Testet följde en undertecknad Rules of Engagement. Teamet tog sig förbi inloggningen och nådde administratörsfunktioner. Därifrån gick det att köra kommandon på servern. Det gav åtkomst till personuppgifter i en intern tjänst och till en skyddad fil på servern.

**Vad påverkades?** Webbportalen och servern den körs på. Även en intern tjänst med personuppgifter påverkades.

**Hur upptäcktes det?** Angreppen upptäcktes inte medan de pågick. De hittades i efterhand när teamet gick igenom målmiljöns loggar. Alla fem angreppstyper syntes i loggarna. Inget larm kunde verifieras.

**Min roll:** Jag deltog i testet på angriparsidan. Jag kartlade portalen och verifierade själv flera av sårbarheterna. Post-mortem genomfördes i teamet och jag deltog i arbetet. Logganalysen finns dokumenterad i avsnitt 6 i Red Team-rapporten. Här analyserar jag orsaker och förbättringar utifrån den.

---

## Lesson learned 1 – Servern litade på det användaren skickade

### Observation

Två enkla knep gav full administratörsåtkomst. Det första var att ändra en cookie i webbläsaren så att den sa "admin". Det andra var att skriva en kort SQL-kod i lösenordsfältet. Väl inne kunde ett adminverktyg köra valfria kommandon på servern. Hela kedjan från inloggning till personuppgifter tog teamet mindre än en timme den 20 oktober.

### Orsak

**Teknisk orsak:** applikationen kontrollerade inte behörighet på servern. Den litade på en roll som klienten själv skickade. Inmatningen i inloggningen och i adminverktyget skickades vidare utan kontroll.

**Bakomliggande orsak:** det fanns ingen säkerhetskontroll i utvecklingsflödet som kunde ha fångat bristerna innan portalen gick i drift. Felen är vanliga och välkända. De finns med på OWASP Top 10.

### Konsekvens

En angripare utan konto kunde nå administratörsfunktioner. Därifrån gick det att läsa löner och personnummer. I en riktig verksamhet är det en personuppgiftsincident enligt GDPR. Det innebär risk för sanktioner och skadat förtroende.

### Förbättring

1. Behörighet ska alltid kontrolleras på servern. Roller får aldrig sparas i något som användaren kan ändra.
2. Databasfrågor ska byggas med parametriserade frågor. Då kan inmatning aldrig tolkas som kod.
3. Adminverktyget ska bara acceptera giltiga IP-adresser. Det ska inte kunna köra kommandon på servern.
4. Automatiska säkerhetstester ska läggas in i CI/CD-flödet. Då fångas liknande fel innan de når drift.

### Uppföljning

Ett omtest av hela angreppskedjan görs efter att åtgärderna är införda. Kedjan ska inte längre fungera. Uppföljningen sker med **KPI 1 Åtgärdstakt** i [`kpi.md`](./kpi.md). De tre kritiska fynden ska vara åtgärdade inom 7 dagar.

---

## Lesson learned 2 – Loggarna såg allt men ingen fick veta

### Observation

Alla fem angreppstyper lämnade tydliga spår i loggarna. SQL-koden syntes i databasloggen. Kommandona syntes i webbserverns logg. Ändå kunde inget larm eller någon incidentåtgärd verifieras. De första angreppen skedde den 20 oktober. Åtkomsten till den skyddade filen skedde den 27 oktober. Angriparen kunde alltså fortsätta i en vecka utan att någon reagerade.

### Orsak

**Teknisk orsak:** det fanns inga larmregler kopplade till loggarna. Loggarna låg dessutom i separata filer för webb och databas och intern tjänst.

**Bakomliggande orsak:** loggning sågs som något man samlar in. Det fanns ingen tydlig ägare som skulle bevaka loggarna och agera på dem.

### Konsekvens

Organisationen hade informationen men kunde inte använda den. Ett verkligt intrång hade kunnat pågå länge. Skadan blir större ju längre en angripare får hålla på. Loggarna räcker för att utreda i efterhand. De räcker inte för att stoppa angreppet medan det pågår.

### Förbättring

1. Skapa larmregler för de fem angreppstyperna. Underlaget finns redan i loggarna.
2. Samla alla loggar på ett ställe. Då kan händelser från olika system kopplas ihop.
3. Utse en ansvarig som tar emot larmen och har en enkel rutin för vad som ska göras.

### Uppföljning

Angreppen upprepas i ett godkänt testfönster efter att larmen är införda. Varje angrepp ska ge ett larm inom 15 minuter. Uppföljningen sker med **KPI 2 Detektionsgrad** i [`kpi.md`](./kpi.md). Nuläget är 0 av 5.

---

## Lesson learned 3 – Varje ändring granskades men inte helheten

### Observation

I Blue Team granskades alla ändringar i pull requests innan de godkändes. De flesta gällde identitet och åtkomst. Varje ändring var rimlig för sig. Men helheten blev motsägelsefull. OS Login infördes den 14 september för att styra inloggning centralt i [PR #8](https://github.com/ITSX25-Chas/blueteam-team6/pull/8). En vecka senare byttes en SSH-nyckel i den gamla nyckellistan i [PR #14](https://github.com/ITSX25-Chas/blueteam-team6/pull/14). Teamet underhöll alltså två inloggningsmodeller samtidigt. Det beskrivs i avsnitt 10.4 i Blue Team-rapporten. Red Team-testet visade samma mönster i större skala. En enda brist i behörigheten räckte för att ta sig hela vägen till en skyddad fil på servern.

### Orsak

**Process:** teamets granskning fokuserade på om varje ändring fungerade. Jag var en av granskarna och godkände båda ändringarna. Det fanns ingen checklista som frågade om gamla lösningar togs bort. Den frågade inte heller om ändringen passade ihop med resten av arkitekturen.

**Bakomliggande orsak:** det saknades en gemensam bild av hotet. Utan den blir säkerhetsarbetet en lista med enskilda åtgärder i stället för en sammanhängande lösning.

### Konsekvens

Motsägelsefull konfiguration gör det svårt att veta vilken inloggningsväg som faktiskt gäller. Om OS Login stängs av i framtiden kan de gamla nycklarna börja fungera igen. Det skapar en risk som ingen har tänkt på.

### Förbättring

1. Inför en kort checklista för granskning av säkerhetsändringar. Den ska fråga om ändringen följer minsta behörighet. Den ska fråga om statiska nycklar används. Den ska fråga om den gamla lösningen är borttagen.
2. Gör en helhetsgranskning av infrastrukturkoden varje kvartal. Den ska utgå från hotbilden och inte från enskilda ändringar.

### Uppföljning

Andelen säkerhetsändringar som granskats med checklistan följs upp varje månad. Resultatet syns också i **KPI 3 Minsta behörighet** i [`kpi.md`](./kpi.md). Den ska stiga när gamla nycklar och för breda rättigheter städas bort.

---

## Professionell sammanfattning

**Vad simulerades?** Ett säkerhetsteam testade företagets interna webbportal under två veckor i oktober. Testet gjordes med tillstånd och inom avtalade ramar.

**Vad blev resultatet?** Teamet tog sig in utan giltigt konto. Därifrån nådde de administratörsfunktioner. De kunde läsa anställdas löner och personnummer. De kunde också läsa skyddade filer på servern. Tre av fem brister bedömdes som kritiska.

**Vad betyder det?** En verklig angripare hade kunnat orsaka en personuppgiftsincident enligt GDPR. Det allvarligaste är att ingen märkte något. Alla angrepp syntes i loggarna. Men inget larm gick och ingen agerade. Angreppen pågick i en vecka utan reaktion.

**Grundorsaken** är densamma i båda fallen. Säkerheten byggdes som enskilda åtgärder och inte som en helhet. Servern litade på det användaren skickade. Loggarna samlades in men ingen bevakade dem.

**Rekommendationer i prioritetsordning:**

1. **Inom en vecka:** åtgärda de tre kritiska bristerna i portalen. Stäng av adminverktyget tills det är säkrat.
2. **Inom en månad:** inför larm för de angrepp som redan syns i loggarna. Utse en ansvarig som tar emot larmen.
3. **Inom ett kvartal:** inventera alla konton och ta bort onödiga rättigheter och gamla nycklar. Inför en checklista för säkerhetsgranskning.

**Beslut som behövs:** ledningen behöver utse en ägare för portalens säkerhet och en ägare för övervakningen. Ledningen behöver också godkänna ett omtest efter åtgärderna. Utvecklingen följs med tre KPI:er som rapporteras kvartalsvis. De beskrivs i [`kpi.md`](./kpi.md).
