# Post-mortem, lessons learned och professionell sammanfattning
## Syfte och kursmål
I den här delen ska du genomföra ett kort post-mortem över ett verkligt eller
simulerat intrångsförsök i projektet. Du ska identifiera tre viktiga lärdomar och
omsätta projektets resultat till information för en professionell mottagare.
Arbetet knyter an till:
- **Kursmål 9:** utföra incidentrespons, post-mortem och förbättringsarbete efter
ett verkligt eller simulerat intrångsförsök.
- **Kursmål 13:** självständigt leda och leverera ett komplext säkerhetsprojekt med
höga krav på kvalitet, dokumentation och professionell kommunikation.
En lesson learned är inte bara något som gick bra eller dåligt. Den ska visa vad
som hände, varför det hände, vilken betydelse det fick och hur framtida
säkerhetsarbete kan förbättras.
---
## Händelse och avgränsning
Beskriv kort vilket intrångsförsök, vilken incident eller vilken del av projektet
ditt post-mortem gäller.
Ta med:
- vad som inträffade eller simulerades,
- vilken miljö eller förmåga som påverkades,
- hur händelsen upptäcktes,
- vilka delar av incidentresponsen du själv deltog i.
**Omfattning:** cirka 100–150 ord.
---
## Lesson learned 1
### Observation
SQL Injection i inloggningen. Vem som helst kan logga in som admin utan giltigt lösenord genom att injicera SQL-kod i lösenordsfältet.
### Orsak
Inloggningsformuläret saknar indatavalidering och tillåter därför att databasanropet modifieras. Detta sker via osäker strängkonkatenering. Databasen tolkar anropet som ett logiskt OR-villkor, och eftersom ett villkor alltid är sant så godkänns hela inloggningen.
### Konsekvens
Att någon utan vidare kan logga in som administratör är oerhört allvarligt och riskerar att samma person sedan kan få tillgång till andra av företagets system.
### Förbättring
Inför indatavalidering i inloggningsformuläret samt parametriserade frågor i SQL-anropet.
### Uppföljning
Företaget bör utföra penetrationstester med jämna mellanrum, och framför allt utföra ett test när åtgärder vidtagits för att hindra otillåten inloggning.
---
## Lesson learned 2
### Observation
### Orsak
### Konsekvens
### Förbättring
### Uppföljning
---
## Lesson learned 3
### Observation
### Orsak
### Konsekvens
### Förbättring
### Uppföljning
---
## Professionell sammanfattning
Föreställ dig att mottagaren är en CISO, IT-chef, systemägare eller annan icke-
teknisk beslutsfattare. Mottagaren ska förstå vad som är viktigt utan att först
läsa hela den tekniska dokumentationen.
Skriv en sammanfattning som besvarar:
1. Vad inträffade eller simulerades?
2. Vilka är de viktigaste resultaten och lärdomarna?
3. Vilka risker eller förbättringsområden identifierades?
4. Vad bör organisationen prioritera härnäst?
5. Finns något som kräver beslut eller fortsatt uppföljning?
Använd principen: **resultat → betydelse → rekommendation**.
Undvik en kronologisk aktivitetslista över vad projektgruppen gjorde.
**Omfattning:** cirka 200–300 ord.
---
## Kontroll före commit
- [ ] Jag har avgränsat vilket intrångsförsök eller vilken incident mitt post-
mortem gäller.
- [ ] Jag har beskrivit min egen roll i incidentresponsen.
- [ ] Jag har formulerat tre tydliga lessons learned.
- [ ] Jag skiljer på observation och bakomliggande orsak.
- [ ] Jag beskriver konsekvensen av varje lärdom.
- [ ] Varje lesson learned leder till en konkret förbättring och uppföljning.
- [ ] Den professionella sammanfattningen fokuserar på resultat, risk och
betydelse.
- [ ] Texten är begriplig utan alla tekniska detaljer från projektet.