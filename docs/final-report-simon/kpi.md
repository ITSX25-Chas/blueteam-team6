# KPI:er och säkerhetsmätningar

---

## KPI 1
### Åtgärdstid för kritiska sårbarheter (Mean Time to Remediate – MTTR)
### Syfte
Ge företagsledningen och ansvariga chefer ett tydligt mått på hur snabbt organisationen klarar av att åtgärda allvarliga säkerhetsbrister från det att de upptäcks till dess att en verifierad lösning är på plats i drift.
### Mottagare
Företagsledningen, IT-chef, utvecklingsteam (DevOps) och säkerhetsansvariga.
### Definition och beräkning
Denna KPI mäter den genomsnittliga tiden (i timmar eller dagar) det tar att stänga en sårbarhet som klassats som kritisk eller hög.
(Total tid från upptäckt till verifierad åtgärd för samtliga kritiska sårbarheter under perioden / Totalt antal åtgärdade kritiska sårbarheter under perioden)
### Datakälla och datakvalitet
Datakällan är ärendehanteringssystemet (t.ex. GitHub Issues eller Jira), rapporter från penetrationstester samt loggar från CI/CD-pipelinen som visar när en säkerhetsfix har driftsatts. Datakvaliteten är hög under förutsättning att alla upptäckta brister registreras direkt med tidsstämpel när de hittas och när åtgärden verifierats. Begränsningen är att måttet bara fångar kända sårbarheter som faktiskt har rapporterats in.
### Målvärde eller önskad riktning
Önskad riktning är så kort tid som möjligt.
- Målvärde för kritiska sårbarheter (P1, t.ex. RCE eller autentiseringskringgående): under 48 timmar.
- Gränsvärde för sårbarheter med hög allvarlighetsgrad (P2): max 7 dagar.
### Mätfrekvens
Följs upp månadsvis på ledningsnivå samt direkt efter varje genomfört penetrationstest eller säkerhetsgranskning.
### Tolkning och åtgärd vid avvikelse
En kort åtgärdstid visar att organisationen har fungerande rutiner för att snabbt täppa till farliga säkerhetshål innan en angripare hinner utnyttja dem. Under vårt projekt såg vi hur snabbt en enskild brist i inloggningen eller administrationsverktyget kunde utnyttjas för att ta sig vidare in i systemet. Om åtgärdstiden överstiger gränsvärdet (48 timmar för kritiska fynd) ska resurser omfördelas från vanlig nyutveckling till säkerhetsåtgärder, och vid behov ska den berörda funktionen (exempelvis diagnostikverktyget) stängas ner tillfälligt tills bristen är löst.

---

## KPI 2
### Efterlevnad av minsta möjliga behörighet (Least Privilege Compliance Rate)
### Syfte
Ge ledningen och säkerhetsansvariga insyn i hur väl systemkonton, applikationsprocesser och molntjänster är begränsade till endast de rättigheter som krävs för deras arbetsuppgift, så att skadan begränsas vid ett eventuellt intrång.
### Mottagare
Företagsledningen, IT-avdelningen, molnarkitekter och säkerhetsavdelningen.
### Definition och beräkning
Denna KPI mäter andelen systemkonton, tjänsteprocesser och molnidentiteter (Service Accounts) som har granskats och bekräftats följa principen om minsta möjliga behörighet utan onödiga administratörs- eller root-rättigheter.
(Antal systemkonton och tjänsteidentiteter med korrekt begränsad behörighet / Totalt antal granskade systemkonton och tjänsteidentiteter) x 100
### Datakälla och datakvalitet
Datakällan är IAM-konfigurationer i molnet, Terraform-kod (`main.tf`), rättighetskontroller i operativsystemet (t.ex. SUID-bitar och processanvändare) samt resultat från interna säkerhetstester. Datakvaliteten är god eftersom rättigheter i både kod och operativsystem går att kontrollera maskinellt, men kräver att man har definierat vad varje tjänst faktiskt behöver ha tillgång till.
### Målvärde eller önskad riktning
Målvärdet för produktionskritiska tjänstekonton och webbapplikationer ska vara 100 %. Ett lägsta accepterat gränsvärde är 95 %.
### Mätfrekvens
Kontrolleras automatiskt vid infrastrukturändringar i CI/CD samt genom en bredare genomgång kvartalsvis.
### Tolkning och åtgärd vid avvikelse
Ett högt värde innebär att en komprometterad webbtjänst inte automatiskt ger angriparen tillgång till resten av servern eller molnmiljön. I våra tester i både Blue Team och Red Team såg vi konsekvensen av för breda rättigheter: webbapplikationen kunde via systemverktyget `find` läsa filer i `/root/`, och i molnmiljön hade servicekonton så breda rättigheter att en läckt token gav åtkomst till känsliga filer i Cloud Storage. Om värdet sjunker under gränsvärdet ska de överprivilegierade rättigheterna plockas bort omgående och ändringen blockeras i pipelinen.

---

## KPI 3
### Exponerade hemligheter i kod och konfiguration (Unsecured Secrets in Code & Config)
### Syfte
Ge ledningen och utvecklingsteamet kontroll över att lösenord, API-nycklar, krypteringsnycklar och interna systemadresser inte lagras oskyddat i källkod, konfigurationsfiler eller versionshistorik (Git).
### Mottagare
Företagsledningen, utvecklingschef, DevOps och säkerhetsavdelningen.
### Definition och beräkning
Denna KPI mäter det totala antalet oskyddade hemligheter (lösenord, tokens, hemliga nycklar) som identifieras i källkodsrepositories (inklusive tidigare commits och dolda branches) samt i applikations- och infrastrukturfiler.
Beräkningen redovisas som ett absolut tal:
Totalt antal upptäckta exponerade hemligheter i kodbas, Git-historik och konfigurationsfiler.
### Datakälla och datakvalitet
Datakällan är automatiserade verktyg för secret scanning i GitHub/CI-pipelinen, granskning av Terraform-statefiler och källkodsgranskning. Datakvaliteten är hög för kända nyckelformat och lösenordsfält, men kan ibland ge falska träffar på testdata som behöver filtreras bort.
### Målvärde eller önskad riktning
Målvärdet är 0 exponerade hemligheter i produktion och versionshantering.
### Mätfrekvens
Kontinuerligt vid varje commit/push i CI/CD-pipelinen samt månadsvis skanning av hela Git-historiken och samtliga branches.
### Tolkning och åtgärd vid avvikelse
Värdet 0 betyder att en angripare som får tillgång till en fil eller ett kodrepo inte samtidigt får färdiga lösenord eller nycklar till andra system. Under projektet hittade vi exponerade hemligheter på flera ställen: i Red Team låg databaslösenordet och adressen till det interna API:et i `secret_config.py`, och i Blue Team hittades känslig data i såväl `terraform.tfstate`-backup som i äldre Git-commits, gömda branches och en hårdkodad Flask `SECRET_KEY`. Så fort värdet är över 0 måste den berörda nyckeln eller lösenordet bytas ut (roteras) omedelbart, flyttas till en säker Secret Manager och rensas ur Git-historiken.

---

## Samlad ledningsrapportering
De tre KPI:erna är valda för att tillsammans ge ledningen svar på tre centrala frågor om företagets motståndskraft:
1. **Hur snabbt stänger vi farliga hål i våra tjänster?** (KPI 1 – Åtgärdstid för kritiska sårbarheter)
2. **Hur mycket begränsar vi skadan om någon ändå tar sig in?** (KPI 2 – Minsta möjliga behörighet)
3. **Hur väl skyddar vi våra nycklar och lösenord från att läcka?** (KPI 3 – Exponerade hemligheter i kod och konfiguration)

Tillsammans fångar dessa mätetal hela den attackkedja vi såg under projektet. Om vi åtgärdar kritiska webb-sårbarheter snabbt (KPI 1) minskar risken för ett initialt intrång. Om en angripare ändå lyckas ta sig in i webbapplikationen förhindrar minsta möjliga behörighet (KPI 2) och avsaknaden av hårdkodade lösenord i koden (KPI 3) att angriparen kan läsa ut personuppgifter, ta sig vidare till molnlagring eller ta över hela servern som `root`.

För ledningen fungerar dessa tre KPI:er som ett konkret beslutsunderlag för prioritering: när KPI 3 eller KPI 2 avviker krävs strukturella insatser i hur utvecklarna hanterar behörigheter och nycklar i CI/CD-flödet, och om KPI 1 stiger visar det att teamet behöver mer tid eller resurser för att hinna åtgärda kritiska sårbarheter i tid.

En begränsning med dessa tre KPI:er är att de fokuserar på teknisk sårbarhetshantering, behörigheter och kodhygien. De visar däremot inte hur snabbt en pågående attack upptäcks i loggarna i realtid eller hur väl personalen står emot exempelvis nätfiske (social engineering). Därför bör de i det fortsatta säkerhetsarbetet kombineras med övervakningslarm och löpande säkerhetsutbildning.

---

## Kontroll före commit
- [x] Jag har definierat tre relevanta KPI:er.
- [x] KPI:erna mäter mer än enbart genomförda aktiviteter.
- [x] Varje KPI har definition, datakälla och mottagare.
- [x] Varje KPI har ett målvärde, gränsvärde eller en önskad riktning.
- [x] Jag har beskrivit hur KPI:erna ska tolkas och följas upp.
- [x] Jag har beskrivit vad en avvikelse bör leda till.
- [x] Min rapportering är begriplig för en icke-teknisk beslutsfattare.
- [x] Jag har beskrivit KPI:ernas begränsningar.
