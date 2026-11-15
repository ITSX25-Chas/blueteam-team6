# KPI:er och säkerhetsmätningar

**Namn:** Julia Persson
**Projekt/team:** Team 6 – Blue Team och Red Team – ITSX25 Avancerad IT-säkerhet
**Datum:** 2026-11-15

---

## Syfte och kursmål

Den här delen tar fram tre KPI:er för att följa upp säkerhetsläget efter Team 6:s projekt. KPI:erna bygger på resultat som redan finns i projektet:

- **Blue Team:** teamets molnmiljö i Google Cloud. Den är byggd som kod i repot [`blueteam-team6`](https://github.com/ITSX25-Chas/blueteam-team6). Här gjordes härdningar av identitet och åtkomst.
- **Red Team:** ett penetrationstest mot en tilldelad webbapplikation. Testperioden var 19–30 oktober 2026. Testet gav fem fynd och en post-mortem baserad på målmiljöns loggar.

Arbetet knyter an till kursmål 4 och 10. KPI:erna är skrivna för en icke-teknisk beslutsfattare. Varje KPI svarar på en enkel ledningsfråga.

---

## KPI 1

### Namn

**Åtgärdstakt för allvarliga sårbarheter**

*Ledningsfråga: Stänger vi de farligaste säkerhetshålen tillräckligt snabbt?*

### Syfte och mottagare

KPI:n visar om organisationen åtgärdar allvarliga brister innan de hinner utnyttjas. En hittad sårbarhet minskar inte risken förrän den är åtgärdad.

**Mottagare:** systemägare och IT-chef. Systemägaren ansvarar för att åtgärder får resurser. IT-chefen följer upp att tidsgränserna hålls.

### Definition och beräkning

`Antal kritiska och höga sårbarheter åtgärdade inom tidsgräns / totalt antal identifierade kritiska och höga sårbarheter × 100`

Tidsgränser:

- **Kritisk:** åtgärdad inom 7 dagar
- **Hög:** åtgärdad inom 30 dagar

En sårbarhet räknas som åtgärdad först när ett omtest visar att den inte längre går att utnyttja.

### Datakälla och datakvalitet

**Datakälla:** fyndlistan i avsnitt 4 i penetrationstestrapporten och ett ärendesystem där varje fynd får en ägare och ett datum. Omtester ger bevis på att åtgärden fungerar.

**Nuläge:** Red Team-testet gav fem fynd. Tre bedömdes som kritiska och två som höga. Under testet gällde ändringsstopp. Därför är inga fynd åtgärdade ännu och mätningen startar efter testfönstret.

**Begränsningar:** Fem fynd är ett litet underlag. Ett enda fynd ändrar värdet med 20 procentenheter. Allvarlighetsgraden är teamets egen bedömning och inte en extern granskning.

### Målvärde eller önskad riktning

- **Mål:** minst 95 %. Alla kritiska fynd ska vara åtgärdade inom 7 dagar.
- **Gränsvärde:** under 80 % kräver beslut.

### Mätfrekvens

Följs upp **varje vecka** så länge kritiska fynd är öppna. Därefter **varje månad** och efter varje nytt penetrationstest.

### Tolkning och åtgärd vid avvikelse

**Bra värde:** organisationen stänger allvarliga hål snabbt. Fönstret där en angripare kan utnyttja dem är kort.

**Dåligt värde:** allvarliga brister ligger öppna längre än acceptabelt. Det är en direkt verksamhetsrisk. Red Team-testet visade att fynden kunde kombineras till åtkomst till personuppgifter och skyddade serverfiler.

**Vid avvikelse:**

1. Systemägaren eskalerar öppna kritiska fynd till IT-chefen.
2. Om en åtgärd inte kan göras i tid ska en tillfällig skyddsåtgärd beslutas. Exempel: stänga av den sårbara funktionen.
3. Om värdet ligger under gränsvärdet två mätningar i rad ska resursbehovet tas upp i ledningsgruppen.

---

## KPI 2

### Namn

**Detektionsgrad för testade angrepp**

*Ledningsfråga: Märker vi när någon angriper oss?*

### Syfte och mottagare

KPI:n visar om försvaret upptäcker angrepp medan de pågår. Att något syns i en logg i efterhand räcker inte. Någon måste få ett larm och kunna agera.

**Mottagare:** CISO eller IT-chef. KPI:n visar om investeringar i övervakning ger effekt.

### Definition och beräkning

`Antal testade angreppsscenarier som utlöste ett larm inom 15 minuter / totalt antal testade angreppsscenarier × 100`

Ett angreppsscenario är en angreppstyp från ett penetrationstest. Exempel: SQL-injektion i inloggningen eller kommandoinjektion i adminverktyget.

Som stödmått redovisas även **loggtäckning**. Det är andelen scenarier som lämnade spår i loggarna.

### Datakälla och datakvalitet

**Datakälla:** penetrationstestets lista över scenarier med tidpunkter och larmhistoriken från övervakningssystemet. Loggarna jämförs med testets tidpunkter.

**Nuläge från post-mortem i avsnitt 6 i Red Team-rapporten:**

| Mått | Nuläge |
|---|---|
| Loggtäckning | **5 av 5** scenarier syntes i loggarna = 100 % |
| Detektionsgrad | **0 av 5** scenarier kunde verifieras som larm = 0 % |

**Begränsningar:** Teamet hade bara tillgång till loggfiler. Teamet kunde inte se något larmsystem. Värdet 0 % betyder därför att inget larm kunde *verifieras*. Det bevisar inte att larm helt saknas. Fem scenarier är också ett litet underlag.

### Målvärde eller önskad riktning

- **Mål inom sex månader:** minst 80 %.
- **Mål för kritiska scenarier:** 100 %. Exempel: kommandoinjektion och åtkomst till personuppgifter.
- **Önskad riktning:** stigande vid varje omtest.

### Mätfrekvens

Mäts **efter varje penetrationstest eller omtest**. Det ska ske minst **en gång per kvartal**. Rapporteras till ledningen **kvartalsvis**.

### Tolkning och åtgärd vid avvikelse

**Bra värde:** organisationen upptäcker angrepp i tid och kan begränsa skadan.

**Dåligt värde:** angrepp kan pågå obemärkt. Nuläget visar att alla fem angrepp lämnade spår. Men ingen kunde visa att någon fick veta det medan det hände.

**Vid avvikelse:**

1. För varje scenario utan larm skapas en larmregel. Underlaget finns redan i loggarna.
2. Larmet testas igen genom att scenariot upprepas i ett godkänt testfönster.
3. Om målet inte nås ska ledningen besluta om resurser för central loggsamling och bemannad övervakning.

---

## KPI 3

### Namn

**Andel konton med minsta behörighet och utan statiska nycklar**

*Ledningsfråga: Hur svårt är det för en angripare att ta sig vidare om ett system blir hackat?*

### Syfte och mottagare

KPI:n visar hur väl organisationen begränsar skadan när något redan har gått fel. Om varje konto bara har de rättigheter det behöver kommer en angripare inte långt. Statiska nycklar är lösenordsfiler som kan läcka och fungerar tills någon tar bort dem.

**Mottagare:** IT-chef och systemägare.

### Definition och beräkning

`Antal konton som uppfyller båda kraven / totalt antal aktiva konton × 100`

Kontona omfattar personkonton och tjänstekonton. De omfattar även konton som används av automatiserade CI/CD-flöden. Ett konto uppfyller kraven när båda punkterna stämmer:

1. Kontot har bara de rättigheter som krävs för dess uppgift. Det kallas minsta behörighet.
2. Kontot loggar inte in med en statisk nyckel som sparas i en fil.

### Datakälla och datakvalitet

**Datakälla:** molnplattformens behörighetsöversikt IAM och infrastrukturkoden i repot. Koden visar vilka rättigheter som är *avsedda*. IAM visar vilka som *faktiskt gäller*.

**Nuläge från projektet:**

- **Förbättrat i Blue Team:** CI/CD-flödet loggar in utan statisk nyckel via Workload Identity Federation. En statisk nyckel för ett tjänstekonto togs bort. Inloggning till servrarna styrs via OS Login.
- **Kvarstående avvikelser:** jumphostens tjänstekonto har ett brett åtkomstområde. Gamla SSH-nycklar finns kvar i konfigurationen bredvid OS Login. Det beskrivs i avsnitt 10.3–10.4 i Blue Team-rapporten.
- **Red Team:** webbapplikationen kunde läsa en fil som bara root ska kunna läsa. Det tyder på att applikationen har för höga rättigheter på servern.

Ett exakt procentvärde kräver en fullständig inventering av alla konton. Den finns inte i projektet. **Första steget är därför att inventera.**

**Begränsningar:** Koden kan skilja sig från det som faktiskt körs. Någon kan ha ändrat behörigheter direkt i molnportalen. Därför ska båda källorna jämföras.

### Målvärde eller önskad riktning

- **Mål:** 100 %.
- **Gränsvärde:** under 90 % kräver åtgärd.
- **Statiska nycklar:** inga nya får skapas utan godkänt undantag.

### Mätfrekvens

Kontrolleras **automatiskt vid varje ändring** i infrastrukturkoden. Sammanställs **varje månad**.

### Tolkning och åtgärd vid avvikelse

**Bra värde:** ett hackat system eller ett läckt konto ger angriparen liten möjlighet att ta sig vidare.

**Dåligt värde:** ett enda intrång kan sprida sig till fler system och data.

**Vid avvikelse:**

1. Kontots ägare får i uppdrag att minska rättigheterna eller byta till inloggning utan nyckel.
2. Läckta eller onödiga nycklar spärras direkt.
3. Om ett undantag behövs ska det godkännas av systemägaren och ha ett slutdatum.

---

## Samlad ledningsrapportering

De tre KPI:erna följer samma kedja som ett verkligt angrepp:

| KPI | Fråga | Del av försvaret |
|---|---|---|
| 1. Åtgärdstakt | Stänger vi hålen? | **Förebygga** |
| 2. Detektionsgrad | Märker vi angreppet? | **Upptäcka** |
| 3. Minsta behörighet | Hur långt kommer angriparen? | **Begränsa** |

Tillsammans ger de ledningen en enkel bild. Hur svårt är det att ta sig in? Hur snabbt märker vi det? Hur stor kan skadan bli?

**Trend att uppmärksamma:** KPI 2 är viktigast just nu. Alla fem testade angrepp syntes i loggarna men inget larm kunde verifieras. Organisationen samlar alltså in rätt information men använder den inte. Det är ett tydligt och relativt billigt förbättringsområde.

**Beslut som KPI:erna stödjer:**

- KPI 1 stödjer beslut om resurser för att åtgärda de tre kritiska fynden.
- KPI 2 stödjer beslut om övervakning och larm.
- KPI 3 stödjer beslut om att inventera och städa behörigheter.

**Vad KPI:erna inte visar:** de mäter inte säkerheten i applikationens kod direkt. De säger inte heller hur medveten personalen är om säkerhet. Underlaget är dessutom litet och kommer från en labbmiljö. KPI:erna behöver därför kompletteras med regelbundna penetrationstester och kodgranskning.
