# Individuell slutrapport

**Namn:** Julia Persson
**GitHub-konto:** JuliaChas
**Projekt/team:** Team 6 – Blue Team och Red Team – ITSX25 Avancerad IT-säkerhet
**Datum:** 2026-11-15

---

## Syfte och kursmål

Rapporten sammanställer projektets resultat och mina egna reflektioner för en fiktiv ledningsgrupp. Den knyter an till kursmål 4 och 9 och 10 och 11 och 13.

Rapporten bygger vidare på två andra filer i samma mapp:

- [`kpi.md`](./kpi.md) – tre KPI:er för uppföljning
- [`lessons-learned.md`](./lessons-learned.md) – post-mortem och tre lärdomar

---

## 1. Professionell sammanfattning

**Vad gjordes?** Team 6 byggde och härdade en molnmiljö i Blue Team. I Red Team testade teamet sedan en intern webbportal med tillstånd. Testet pågick i två veckor i oktober 2026.

**Viktigaste resultat:** Teamet tog sig in i portalen utan giltigt konto. Därifrån gick det att nå administratörsfunktioner och köra kommandon på servern. Det gav åtkomst till anställdas löner och personnummer. Tre av fem brister bedömdes som kritiska.

**Största risken:** Ingen märkte angreppen. Alla fem angreppstyper syntes i loggarna. Men inget larm kunde verifieras. Angreppen pågick under en vecka utan att någon reagerade. I en verklig verksamhet hade det varit en personuppgiftsincident enligt GDPR.

**Positivt:** Blue Team-arbetet visar att identitet och åtkomst kan förbättras med små medel. Statiska nycklar togs bort och inloggning till servrar styrs nu centralt.

**Vad KPI:erna följer upp:** Tre KPI:er mäter om organisationen stänger hålen och märker angrepp och begränsar skadan. Det viktigaste nuläget är att **0 av 5** testade angrepp gav ett verifierat larm.

**Rekommendationer i prioritetsordning:**

1. Åtgärda de tre kritiska bristerna i portalen inom en vecka.
2. Inför larm för de angrepp som redan syns i loggarna inom en månad.
3. Inventera konton och ta bort onödiga rättigheter och gamla nycklar inom ett kvartal.

**Beslut som behövs:** Ledningen behöver utse en ägare för portalens säkerhet och en ägare för övervakningen. Ledningen behöver också godkänna ett omtest efter åtgärderna.

---

## 2. Kontext och avgränsning

**Syfte:** Projektet skulle ge praktisk erfarenhet av både försvar och angrepp. Resultatet skulle sedan omsättas till uppföljning och förbättring.

**Miljöer:** Rapporten bygger på två olika miljöer.

- **Blue Team:** en molnmiljö i Google Cloud som teamet byggde som kod med Terraform. Fokus låg på identitet och åtkomst och CI/CD.
- **Red Team:** en intern webbportal som tilldelades av Comerit. Testet följde en undertecknad Rules of Engagement med ändringsstopp. Inga brister åtgärdades under testet.

**Avgränsningar:** Inga nya tester har gjorts för denna rapport. Miljöerna är labbmiljöer. Personuppgifterna i portalen är fiktiva. Fysisk säkerhet och social engineering ingick inte.

**Underlag:** Teamets två rapporter och repot med Git-historik och pull requests. Även mina egna skärmdumpar från testet ingår. Red Team-rapporten är märkt TLP:AMBER. Den ligger därför inte i detta publika repo. Den är inlämnad i Canvas.

---

## 3. Mitt individuella bidrag

Teamet arbetade till stor del tillsammans. Vi tog beslut gemensamt på möten och arbetspass. Jag deltog i samtliga.

**Blue Team – kodgranskning.** Min tydligaste roll var granskare. Jag granskade och godkände 11 pull requests innan de fördes in i main. De syns i repots [PR-historik](https://github.com/ITSX25-Chas/blueteam-team6/pulls?q=is%3Apr+reviewed-by%3AJuliaChas). De flesta gällde identitet och åtkomst:

- **Inloggning utan statiska nycklar** för CI/CD via Workload Identity Federation – PR #2 och #4
- **Borttagen nyckel** för ett tjänstekonto – PR #12
- **OS Login** och SSH-hantering – PR #8 och #14 och #16
- **Egna tjänstekonton per person** för spårbarhet – PR #5 och #6 och #7

**Red Team – egen testning.** Parallellt med teamets gemensamma arbete testade jag själv i samma miljö.

- Jag kartlade portalen med ffuf. Jag begränsade hastigheten för att inte bryta mot RoE.
- Jag fångade inloggningen i Burp Suite. Ett citattecken i användarnamnet gav ett serverfel. Det tydde på SQL-injektion.
- Jag verifierade själv SQL-injektionen och åtkomsten till adminsidan och att andra användares profiler gick att läsa.

Skärmdumparna finns i avsnitt 7 i Red Team-rapporten.

**Slutrapportering.** Jag har tagit fram denna slutrapport och de två andra filerna i mappen. Hur AI användes beskrivs i avsnitt 7.

---

## 4. Viktigaste projektresultat och evidens

### Resultat 1 – En kedja av brister gav åtkomst till personuppgifter

#### Resultat

Portalen gick att ta sig in i utan konto på två sätt. Det ena var att ändra en cookie så att den sa "admin". Det andra var SQL-injektion i inloggningen. Ett adminverktyg kunde sedan köra kommandon på servern. Det gav åtkomst till löner och personnummer i en intern tjänst och till en skyddad fil på servern.

#### Evidens

- Red Team-rapporten avsnitt 4 och 5 med fem fynd och attackkedja. Inlämnad i Canvas.
- Mina skärmdumpar i avsnitt 7 i samma rapport visar SQL-injektionen och profilåtkomsten.
- Målmiljöns loggar visar att hela kedjan genomfördes på mindre än en timme den 20 oktober.

#### Verksamhets- och säkerhetsbetydelse

En angripare utan konto kan läsa känsliga personuppgifter. Det är en GDPR-risk med möjliga sanktioner och skadat förtroende. Bristerna är vanliga och välkända. De hade kunnat fångas tidigt.

#### Rekommendation

Utvecklingsteamet åtgärdar de tre kritiska bristerna. Behörighet ska kontrolleras på servern. Databasfrågor ska vara parametriserade. Adminverktyget ska inte kunna köra kommandon.
**Prioritet:** P1 inom en vecka. **Ägare:** systemägaren för portalen.

---

### Resultat 2 – Angreppen syntes i loggarna men ingen märkte dem

#### Resultat

Alla fem angreppstyper lämnade tydliga spår i loggarna. Inget larm eller någon åtgärd kunde verifieras. Det första angreppet skedde den 20 oktober. Den skyddade filen lästes den 27 oktober.

#### Evidens

Red Team-rapporten avsnitt 6 med loggutdrag från webbserver och databas och intern tjänst.

#### Verksamhets- och säkerhetsbetydelse

Organisationen har informationen men använder den inte. Ett verkligt intrång kan pågå i veckor. Ju längre en angripare får hålla på desto större blir skadan.

#### Rekommendation

Skapa larmregler för de fem angreppstyperna. Samla loggarna på ett ställe. Utse en ansvarig som tar emot larmen.
**Prioritet:** P1 inom en månad. **Ägare:** IT-chef eller ansvarig för övervakning.

---

### Resultat 3 – Identitet och åtkomst förbättrades men helheten har luckor

#### Resultat

I Blue Team togs statiska nycklar bort. CI/CD loggar nu in utan sparad nyckel. Inloggning till servrar styrs via OS Login. Samtidigt finns kvarstående brister. Ett tjänstekonto har för brett åtkomstområde. Gamla SSH-nycklar ligger kvar bredvid OS Login.

#### Evidens

- [PR #2](https://github.com/ITSX25-Chas/blueteam-team6/pull/2) och [PR #4](https://github.com/ITSX25-Chas/blueteam-team6/pull/4) – Workload Identity Federation
- [PR #12](https://github.com/ITSX25-Chas/blueteam-team6/pull/12) – borttagen nyckel
- [PR #8](https://github.com/ITSX25-Chas/blueteam-team6/pull/8) och [PR #14](https://github.com/ITSX25-Chas/blueteam-team6/pull/14) – OS Login och SSH-nycklar
- Blue Team-rapporten avsnitt 10.3 och 10.4. Inlämnad i Canvas.

#### Verksamhets- och säkerhetsbetydelse

Färre nycklar betyder färre saker som kan läcka. Men motsägelsefull konfiguration gör det oklart vilken inloggningsväg som gäller. Om OS Login stängs av kan de gamla nycklarna börja fungera igen.

#### Rekommendation

Ta bort de gamla SSH-nycklarna. Minska tjänstekontots rättigheter. Inför en checklista för granskning av säkerhetsändringar.
**Prioritet:** P2 inom ett kvartal. **Ägare:** plattformsteamet.

---

## 5. KPI:er och ledningsrapportering

De fullständiga definitionerna finns i [`kpi.md`](./kpi.md).

| KPI | Ledningsfråga | Nuläge | Mål |
|---|---|---|---|
| 1. Åtgärdstakt för allvarliga sårbarheter | Stänger vi hålen tillräckligt snabbt? | 0 av 5 åtgärdade på grund av ändringsstopp | Minst 95 % inom tidsgräns |
| 2. Detektionsgrad för testade angrepp | Märker vi när någon angriper oss? | 0 av 5 med verifierat larm | Minst 80 % inom sex månader |
| 3. Minsta behörighet och inga statiska nycklar | Hur långt kommer en angripare? | Delvis förbättrat men inte inventerat | 100 % |

**Hur de kommunicerar säkerhetsstatus:** KPI:erna följer samma kedja som ett angrepp. Den första mäter om vi förebygger. Den andra mäter om vi upptäcker. Den tredje mäter om vi begränsar skadan. En ledningsgrupp kan alltså se var i kedjan svagheten finns.

**Varför de passar en icke-teknisk beslutsfattare:** varje KPI är en andel i procent med ett tydligt mål. Varje KPI svarar på en enkel fråga. Man behöver inte förstå tekniken för att se om värdet går åt rätt håll.

**Beslut de stödjer:** KPI 1 stödjer beslut om resurser för att åtgärda brister. KPI 2 stödjer beslut om övervakning. KPI 3 stödjer beslut om att städa behörigheter.

**Begränsningar:** Underlaget är litet med bara fem fynd och fem angrepp. Ett enda fynd flyttar värdet 20 procentenheter. KPI 2 bygger på loggfiler. Teamet kunde inte se något larmsystem. KPI 3 saknar ett exakt nuläge eftersom ingen fullständig inventering finns.

---

## 6. Incidentrespons, post-mortem och förbättringsarbete

Det fullständiga underlaget finns i [`lessons-learned.md`](./lessons-learned.md).

**Vad analyserades?** Red Team-testets simulerade intrång i webbportalen.

**Hur upptäcktes det?** Intrånget upptäcktes inte medan det pågick. Det hittades i efterhand när teamet gick igenom målmiljöns loggar. Det fanns ingen incidentrespons under testet. Det är i sig det viktigaste resultatet.

**Orsaker och konsekvenser:**

| Orsak | Konsekvens |
|---|---|
| Servern litade på det användaren skickade | En angripare utan konto nådde personuppgifter |
| Loggar samlades in men ingen bevakade dem | Angreppen pågick i en vecka utan reaktion |
| Säkerhetsändringar granskades en och en | Motsägelsefull konfiguration med dolda risker |

**Förbättringar:**

1. Kontrollera behörighet på servern och inför automatiska säkerhetstester i CI/CD.
2. Inför larm och central loggsamling och en ansvarig för övervakning.
3. Inför en checklista för granskning och en helhetsgranskning varje kvartal.

**Uppföljning:** Ett omtest av hela kedjan görs efter åtgärderna. Kedjan ska inte fungera. Varje angrepp ska ge ett larm inom 15 minuter. Resultatet följs med de tre KPI:erna.

**Vad organisationen behöver lära sig:** Säkerhet fungerar inte som en lista med enskilda åtgärder. Den måste fungera som en helhet. Att samla in loggar är inte samma sak som att upptäcka angrepp.

---

## 7. AI i säkerhetsarbetet

### Konkreta användningsfall

Jag har använt Claude som stöd i två delar av projektet.

**Under Blue Team och Red Team** använde jag Claude för att förstå begrepp och verktyg. Exempel är Workload Identity Federation och OS Login och verktygen ffuf och Burp Suite. Jag använde det också för att slå upp MITRE ATT&CK-tekniker. Själva testerna utförde jag utan AI.

**Under slutrapporteringen** använde jag Claude som stöd för struktur och formulering. Underlaget var uppgiftsbeskrivningen och mallarna och projektets rapporter. Innehållet och slutsatserna har jag granskat och anpassat.

### När AI var lämpligt

AI var lämpligt för att tolka resultat under testet. Ett exempel är när servern svarade med fel 500 efter ett citattecken i inloggningen. AI hjälpte mig att snabbt koppla det till SQL-injektion. Jag kunde sedan läsa vidare i källorna.

AI var också användbart för att få överblick. Kursen hade flera inlämningar med olika mallar. AI gjorde det lättare att se hur delarna hänger ihop.

### När AI var olämpligt eller otillräckligt

AI kan inte avgöra vad som faktiskt hände i ett projekt. Uppgifter om mitt bidrag och om projektets resultat måste komma från projektets egen evidens. Exempel är Git-historiken och rapporterna och skärmdumparna.

AI är heller inte lämpligt att lita på ensamt för klassificering. En ATT&CK-teknik kan låta rimlig men ändå inte beskriva det som faktiskt hände.

AI kan dessutom formulera sig mer säkert än underlaget tillåter. Därför skiljer rapporterna på vad som är verifierat och vad som inte är det. Ett exempel är KPI 2. Där står att inget larm kunde *verifieras*. Det står inte att larm saknas.

### Verifiering och mänsklig kontroll

ATT&CK-tekniker kontrollerade jag på attack.mitre.org. Siffror och datum kontrollerades mot teamets rapporter och loggutdrag. AI-genererad text lästes igenom och rättades där den inte stämde med underlaget eller med min egen roll.

### Risker och professionella gränser

**Hallucinationer:** AI kan ge felaktiga säkerhetsråd som låter övertygande. Allt behöver kontrolleras mot en oberoende källa.

**Informationsläckage:** Projektets rapporter innehåller information klassad som TLP:AMBER. I labbmiljön var data fiktiva. I en verklig organisation får sådan information bara delas med verktyg som organisationen har godkänt.

**Bristande spårbarhet:** Det kan bli oklart vad som kommer från projektet och vad som kommer från AI. Därför skiljer källförteckningen på projektets evidens och AI-stöd.

**Övertro:** En välskriven text är inte samma sak som en korrekt text.

### Professionell slutsats

I yrkeslivet kommer jag att följa fyra principer:

1. AI är ett stöd för förståelse och struktur. Det är aldrig en källa.
2. Allt AI föreslår kontrolleras mot originalkällan innan det används.
3. Känslig information delas bara med AI-verktyg som organisationen har godkänt.
4. Jag står själv för allt jag lämnar ifrån mig. Även när AI har hjälpt till.

---

## 8. Prioriterade rekommendationer

| Prioritet | Vad | Varför | Risk som adresseras | Ägare |
|---|---|---|---|---|
| **P1** inom en vecka | Åtgärda de tre kritiska bristerna i portalen. Stäng av adminverktyget tills det är säkrat. | En angripare utan konto når personuppgifter idag | Dataintrång och GDPR-incident | Systemägare för portalen |
| **P1** inom en månad | Inför larm för de fem angreppstyperna. Utse en ansvarig för övervakning. | Angrepp pågick i en vecka utan reaktion | Långvariga intrång som inte upptäcks | IT-chef |
| **P2** inom ett kvartal | Inventera konton. Ta bort gamla nycklar och onödiga rättigheter. Inför en checklista för granskning. | Ett intrång kan spridas om konton har för många rättigheter | Större skada vid intrång | Plattformsteamet |
| **P2** efter åtgärderna | Genomför ett omtest av hela angreppskedjan. | Bekräftar att åtgärderna fungerar | Falsk trygghet | Systemägare och testansvarig |

---

## 9. Individuell reflektion

### Vad kan jag nu som jag inte kunde när kursen började?

Innan kursen kände jag till SQL-injektion i teorin. Nu har jag sett det hända. I Burp Suite fångade jag inloggningen och lade till ett enda citattecken. Servern svarade med ett felmeddelande. Då förstod jag att det jag skriver faktiskt hamnar i databasfrågan. Jag kan nu kartlägga en webbapplikation med ffuf och analysera trafik i Burp Suite. Jag kan också koppla ett fynd till rätt ATT&CK-teknik.

### Vilket moment har förändrat mitt sätt att arbeta med cybersäkerhet mest?

Red Team med Burp Suite. Det var roligt och väldigt lärorikt. Det gjorde att jag kunde se trafiken visuellt. Jag kunde se exakt vad webbläsaren skickade och vad servern svarade. Säkerhet blev något konkret och inte bara teori. Jag förstod också varför servern aldrig får lita på det användaren skickar. I Burp Suite ser man hur enkelt det är att ändra allt.

### Vad skulle jag göra annorlunda om projektet började idag?

Jag skulle dokumentera löpande och mer noggrant. När rapporterna skulle skrivas var det svårt att visa vad just jag hade gjort. Vi arbetade mycket tillsammans. Jag fick leta i GitHub efter mina granskade pull requests. Jag fick också gå igenom mina skärmdumpar i efterhand. Om jag hade skrivit korta anteckningar efter varje arbetspass hade det varit enklare. Anteckningarna skulle innehålla vad jag testade och vad resultatet blev och vilket datum det var.

### Vad tar jag med mig till LIA och min framtida yrkesroll?

**Tänka minsta behörighet.** I Blue Team såg jag hur många vägar in det finns till en miljö. Det gäller tjänstekonton och nycklar och SSH. Varje väg är en risk. På LIA vill jag fråga vem som behöver åtkomst och varför. Jag vill också fråga hur länge åtkomsten behövs.

**Följa ramar och regler.** I Red Team fick inget test börja innan Rules of Engagement var undertecknad. Jag begränsade hastigheten i ffuf för att inte bryta mot reglerna. I ett riktigt jobb gäller samma sak. Jag måste veta vad jag får göra innan jag gör det. Om något går utanför ramarna ska jag stoppa och säga till direkt.

---

## 10. Källor och referenser

### Projektets evidens

1. Team 6. *Säkerhetsrapport – Blue Team*. Inlämnad i Canvas. Avsnitt 10.3 och 10.4 används.
2. Team 6. *Penetrationstestrapport – Red Team*. Testperiod 19–30 oktober 2026. TLP:AMBER. Inlämnad i Canvas. Avsnitt 4 och 5 och 6 och 7 används.
3. Team 6. Repot [`blueteam-team6`](https://github.com/ITSX25-Chas/blueteam-team6) med pull requests #2 och #4 och #5 och #6 och #7 och #8 och #12 och #14 och #16.
4. Julia Persson. Egna skärmdumpar från ffuf och Burp Suite och portalen. Datum 20 oktober 2026. Finns i avsnitt 7 i Red Team-rapporten.

### Externa källor

5. MITRE ATT&CK. [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/).
6. MITRE ATT&CK. [T1059.004 Command and Scripting Interpreter: Unix Shell](https://attack.mitre.org/techniques/T1059/004/).
7. MITRE ATT&CK. [T1213 Data from Information Repositories](https://attack.mitre.org/techniques/T1213/).
8. MITRE ATT&CK. [T1552.001 Unsecured Credentials: Credentials In Files](https://attack.mitre.org/techniques/T1552/001/).
9. OWASP. [OWASP Top 10](https://owasp.org/Top10/).
10. FIRST. [Traffic Light Protocol 2.0](https://www.first.org/tlp/).
11. Google Cloud. [OS Login](https://cloud.google.com/compute/docs/oslogin).
12. Google Cloud. [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation).

### Kursmaterial

13. Uppgiftsbeskrivning *Avslutande arbete: från säkerhetsarbete till professionell rapportering med KPI:er* och mallarna `kpi-3.md` och `lessons-learned-3.md` och `individual-report-3.md`.

### AI-stöd

14. Claude från Anthropic. Använt under projektet för att förstå begrepp och verktyg. Använt för struktur och utkast till slutrapporten. AI-text är inte en källa till projektets resultat. Alla fakta kommer från källorna ovan.

### Egna bedömningar

KPI-mål och tidsgränser och prioriteringar är mina egna förslag. De bygger på projektets resultat men är inte uppmätta värden.
