# KPI:er och säkerhetsmätningar
---
## KPI 1
### Säkerhetsefterlevnad i infrastruktur (Configuration Compliance Rate)
### Syfte
Ge ledningen och IT-avdelningen en tydlig bild av hur väl företagets molninfrastruktur och konfigurationer följer interna säkerhetskrav och etablerade branschstandarder.
### Mottagare
Företagsledningen, IT-avdelningen, DevOps, säkerhetsavdelningen
### Definition och beräkning
Denna KPI mäter andelen molnresurser och konfigurationer som följer företagets fastställda säkerhetsregler i förhållande till det totala antalet granskade resurser.
(Antal resurser och konfigurationer som lever upp till säkerhetskrav
/ Totalt antal granskade resurser och konfigurationer)
x 100
### Datakälla och datakvalitet
Datakällan är företagets konfigurationer, policies, kod och brandväggsregler. Eftersom det företaget kan titta på är den egna konfigurationen och koden så är datakvaliteten väldigt exakt. Begränsningen är att den bara visar hur den är inställd när den pushas, och därefter kan ändringar göras i molnkonsolen. Datan jämförs med företagets fastställda säkerhetslinje och best practices.
### Målvärde eller önskad riktning
Långsiktigt ska målvärdet vara 100%. Det är svårt och tar tid att uppnå. Ett realistiskt gränsvärde på 95% bör införas och efterlevas. Om värdet sjunker under 95% ska åtgärder sättas in.
### Mätfrekvens
Ska mätas automatiskt vid varje ändring i CI/CD-flödet, samt månadsvis.
### Tolkning och åtgärd vid avvikelse
Ett bra värde (över 95%) betyder att infrastrukturen är säkert uppsatt. Om en allvarlig incident inträffar när värdet är högt kan konsekvenserna hållas nere. Om värdet skulle sjunka ska pipelinen larma och IT-avdelningen måste åtgärda konfigurationen eller den skadliga ändringen.
---
## KPI 2
### Säkerhetsloggning och övervakning (Security Logging and Monitoring Coverage)
### Syfte
Ge företaget en bild av vad som övervakas och vilka typer av attacker som loggas och kan hanteras av säkerhetsteamet.
### Mottagare
Företagsledningen, IT-avdelningen, DevOps, säkerhetsavdelningen
### Definition och beräkning
Denna KPI visar hur stor andel attacker som upptäcks och loggas. Detta görs bäst genom kontrollerade penetrationstest.
(Antal identifierade och loggade angreppsförsök
/ totalt antal genomförda kända angreppsförsök under penetrationstest)
x 100
### Datakälla och datakvalitet
Datakällan är loggning och övervakning under ett penetrationstest. Loggarna det kan handla om är till exempel systemloggar, databasloggar, webbserverloggar. Under testet bör så stora områden som möjligt testas så utförligt som möjligt för att kvaliteten ska vara hög. Om precis allting loggas kan dock viktiga händelser försvinna i mängden. Därför bör det bestämmas i förväg vad övervakningen bör titta på och logga.
### Målvärde eller önskad riktning
Målet är att 100% av kritiska säkerhetsrelaterade händelser loggas och i bästa fall larmas.
### Mätfrekvens
Vid varje ändring av autentisering, inloggning, personaländringar,
samt kvartalsvis.
### Tolkning och åtgärd vid avvikelse
Ett högt värde visar att företaget har en god överblick över sina system och snabbt kan se om någonting händer. I vår post-mortem-analys såg vi att våra tester lämnade spår i loggarna. Vi kan dock inte med säkerhet säga att precis allt loggades, och om systemen inte larmar så visar loggarna bara vad som har hänt och inte vad som aktivt sker. Om värdet är lågt så har inte företaget tillräcklig övervakning. Oavsett värde i vårt test så bör centraliserad logghantering (SIEM) och automatiska larm införas.
---
## KPI 3
### Publika och exponerade molnresurser (Publicly Exposed Cloud Resources)
### Syfte
Ge företagsledningen och säkerhetsteamet en tydlig bild av företagets externa attackyta genom att dokumentera de exponerade resurserna.
### Mottagare
Företagsledningen, säkerhetsavdelningen
### Definition och beräkning
Denna KPI tittar på antalet molnresurser som är tillgängliga utifrån utan tillräckligt skydd som VPN och korrekt konfigurerad brandvägg.
Beräkningen görs som ett tal:
Totalt antal oavsiktligt exponerade resurser.
### Datakälla och datakvalitet
Datakällan är molnplattformens tillgångsinventering (Cloud Asset Inventory), konfigurationer i Terraform, säkerhetsskanningar.
Datakvaliteten är mycket hög eftersom vi kan läsa nätverksregler och publika IP-adresser direkt från konfigurationen.
### Målvärde eller önskad riktning
Målet ska vara 0 oavsiktligt exponerade resurser.
### Mätfrekvens
Kontinuerligt via automatiserade skanningar i CI/CD-pipelinen samt månadsvis uppföljning.
### Tolkning och åtgärd vid avvikelse
Ett lågt värde, helst 0, innebär att attackytan är minimerad och obehöriga utifrån har svårt att komma åt tjänsterna.
Vid avvikelse ökar risken för intrång, vilket vi visar i både Blue Team- och Red Team-rapporterna.
Pipelinen ska blockera publicering, och om avvikelse upptäcks i drift ska resurserna isoleras omedelbart.
---
## Samlad ledningsrapportering
De tre KPI:erna jag har valt hänger ihop med varandra. Infrastrukturen som beskrivs i KPI 1 säkras upp av KPI 3, och med hjälp av KPI 2 får man en bild av hur hela infrastrukturen övervakas. Tillsammans ger de en heltäckande bild av hur motståndskraftig företagets infrastruktur är. När vi vet hur motståndskraftiga vi är i nuläget kan vi se om vi lever upp till den motståndskraft vi vill ha och behöver.

Företaget bör sträva efter att nå målvärdena på KPI:erna. Efterlevnaden ska gå mot 100%. Alla kritiska säkerhetsrelaterade händelser ska loggas, larmas och åtgärdas. Företaget ska påbörja arbetet mot att stänga ner åtkomsten till alla molnresurser som inte måste nås utifrån. KPI:erna ger företagsledningen underlag för var insatser gör mest nytta och vad som ska prioriteras.

Jag har valt tre KPI:er som täcker infrastrukturen. Det innebär att de inte visar sårbarheter i den mänskliga faktorn, eller sårbarheter i företagets applikationer. För att få en fullständig bild måste även applikationer testas och analyseras, och personalen måste få träning i säkerhetsmedvetenhet.

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