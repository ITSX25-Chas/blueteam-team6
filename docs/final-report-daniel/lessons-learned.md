# Post-mortem, lessons learned och professionell sammanfattning

## Händelse och avgränsning
Jag har valt att fokusera på åtkomstkontroll, behörigheter och kodexekvering.
I vårt penetrationstest tog vi oss in i företagets system via hemsidan. Efter det använde vi oss av administrationsverktyget för att exekvera kod på företagets server. Vi lyckades även injicera kod med högre privilegier genom ett Linux-skal.
I vårt test påverkades hemsidan, serverns infrastruktur och databasen med användardata. Vi undersökte och utforskade utan att göra någon skada.
Under delen av kursen som hanterade Blue Team fick vi grundläggande kunskaper i SQL-injektioner och hur man kan använda verktyget *curl* för att anropa en server.
Gruppen jobbade tillsammans. Jag gjorde flera delar individuellt utan hjälp från övriga i gruppen, även om de hade gjort det steget innan mig.
- Jag loggade in med SQL-injektionen.
- Med administrationsverktyget hittade jag den dolda filen med flagga tre.
- Efter instruktioner av utbildaren under Blue Team skrev jag en kommandorad med en payload för att köra kommandon på målservern.
- Efter en mindre knuff i rätt riktning av Malcolm kunde jag hitta information om Privilege Escalation och läsa den sista flaggan som var dold i en lässkyddad fil.
När det gäller de lärdomar jag skrivit om i det här dokumentet så stod jag för inloggningen som admin, inloggningen som användare som inte var admin (byggt på Simons upptäckter), tillvägagångssättet att använda administrationsverktyget och kak-manipulationen i lärdom 3.
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
Command Injection i administrationsverktyg. Om en obehörig kommer åt verktyget så kan den personen komma åt systemets insida, antingen via hemsidan eller med terminalverktyg. Vi lyckades exekvera serverkommandon genom verktyget med hjälp av specialtecken.
### Orsak
Indatavalidering saknas i administrationsverktyget. Genom att skriva vissa specialtecken i textrutan så kör verktyget eventuella efterföljande kommandon mot servern. Likt SQL-injektionen tidigare skickas kommandon efter vissa specialtecken vidare till serverns systemskal utan sanering.
### Konsekvens
En angripare kan komma åt data och manipulera serverns system genom Remote Code Execution. Angriparen kan läsa data, installera bakdörrar och skadlig programvara, och i värsta fall kan en utomstående ta över hela servern.
### Förbättring
Indatavalidering. Verktygets enda funktion är att pinga IP-adresser. Därför behöver användare inte kunna använda specialtecken utöver punkt och kolon. Verktyget behöver inte heller kommunicera med systemet direkt. Istället bör säkra API:er användas, om det inte finns väldigt goda skäl att inte göra det.
### Uppföljning
Åtgärder bör utföras och bedömas omgående. Därefter bör verktyget testas årligen samt vid uppdateringar och ändringar.
---
## Lesson learned 3
### Observation
Broken Access Control. Privilegieeskalering genom manipulering av kakor. En inloggad användare med låga privilegier kan ändra sin roll till admin och få tillgång till det en admin kommer åt. En ej inloggad användare kan skicka modifierade kak-värden till servern och kringå inloggning.
### Orsak
Systemet litar blint på rollen som anges i klientens kaka utan att verifiera användarens rättigheter.
### Konsekvens
Att obehöriga kan få samma rättigheter som admin äventyrar hela systemets integritet. Det är samma risker som om någon hade loggat in som admin med en SQL-injektion. Ännu värre är att kod kan exekveras på servern via ett systemskal utan att en angripare behöver vara inloggad.
### Förbättring
Lagra inte känsliga roller eller behörigheter i klientens kakor. Verifiera användarens behörigheter vid varje anrop. Inför strikt auktorisering på alla känsliga endpoints. Sessioner körs enbart på servern där de lättare kan kontrolleras.
### Uppföljning
Inför automatiserade säkerhetstester i CI/CD-pipelinen. Granska regelbundet hur sessioner och behörigheter hanteras på servern.
---
## Professionell sammanfattning
När vi genomförde vårt penetrationstest mot företagets hemsida undersökte vi motståndskraften i inloggningssystemet, om en obehörig användare kan ta sig runt i systemet vid ett intrång, och hur skyddade eventuella känsliga filer och uppgifter var.
Företagets webbtjänst uppvisar omfattande och kritiska brister, men vi bedömde att följande tre sårbarheter har högst prioritet:
- Obehöriga kan enkelt logga in som administratör på hemsidan.
- Administrationsverktyget på hemsidan tillåter alla som har tillgång till verktyget att köra kod direkt på servern.
- Det går att komma åt servern och köra kod på den även utan att vara inloggad.
Dessa sårbarheter måste åtgärdas omgående för att förhindra att utomstående får tillgång till företagets system. Om det sker riskerar företaget att läcka eller förlora känslig data, förlora tillgången till sina egna system, och att tjänsten som företaget erbjuder blir otillgänglig. Förutom de ekonomiska förlusterna kan detta komma att resultera i försämrat rykte och tillit från företagets kunder. Det finns även regulatoriska ramverk som företaget kan bedömas ha misslyckats följa.
Det som måste göras omedelbart är att flytta autentiseringen från klientens enheter till företagets server. Servern måste verifiera användarens identitet vid varje känslig aktivitet. Detta måste göras för att hindra att utomstående får tillgång till systemet. Därefter bör administrationsverktyget göras om så att det inte har möjlighet att köra kod på servern.
---
## Kontroll före commit
- [x] Jag har avgränsat vilket intrångsförsök eller vilken incident mitt post-
mortem gäller.
- [x] Jag har beskrivit min egen roll i incidentresponsen.
- [x] Jag har formulerat tre tydliga lessons learned.
- [x] Jag skiljer på observation och bakomliggande orsak.
- [x] Jag beskriver konsekvensen av varje lärdom.
- [x] Varje lesson learned leder till en konkret förbättring och uppföljning.
- [x] Den professionella sammanfattningen fokuserar på resultat, risk och
betydelse.
- [x] Texten är begriplig utan alla tekniska detaljer från projektet.