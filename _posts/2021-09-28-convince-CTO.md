---
layout: post
title: Inlägg 28-09-2021
subtitle: Närverk i molnet
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [Azure-Service-Bus, Azure-Private-Link]
comments: true
readtime: true
---

# Dear CTO;

God dag, 

Jag förstår att det kanske känns osäkert att lagra den klassificerade informationen
uppe i molnet. Men efter lite undersökning och testning så kan jag försäkra dig om att
det är minst lika säkert (om inte mer) som att ha allt lagrat lokalt. Och förhoppningsvis kan
jag förklara på ett bra sätt precis varför.

Som vi tidigare diskuterade är det specifikt **Azure Private Link** i samband med **Azure Service Bus** jag vill implementera i ledet att modernisera systemet.

Först vill jag förklara hur en service bus fungerar:

När man använder sig av service bus så skickas all data eller *messages* som det heter inom området till en *queue* eller ett *topic*. 

Queue:
![image](_posts\Images\QueueBus.png)

I en queue så skickas ett message till en tjänst/applikation, när det sedan har konsumerats av någon av mottagarna så tas det sedan bort ur queuen. 

När input i form av ett message skickas lagras det i vår queue, och skickas sedan vidare enligt [FIFO](https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)) modellen (first-in-first-out). Detta kan däremot ändras genom att tilldela properties till våra messages, ju lägre ju högre upp på listan. Se det som nummerlappar hos tandläkaren, oavsett hur länge ett message har väntat i vår queue så kommer det med motsvarande priority att skickas först. 

Det går även att använda något som kallas *Message Scheduling*, något som många idag gör för att skicka ut mail vid en bestämd tid. Man sätter en parameter som bestämmer när ett message skall skickas. Detta är smidigt om man har icke-kritiska messages att skicka ut, då man kan färdigställa arbetet och gå på nästa uppgift utan att behöva vänta på processering eller att något mer kritiskt skall färdigställas.

Om man vill säkerställa att alla messages från ett visst ursprung eller med ett visst syfte processeras samtidigt kan man använda sig av det som kallas *Message Sessions*. I grund och botten är detta ett enkelt sätt att särskilja en viss mängd av en *message stream* så de kan hanteras av en viss message handler. För att göra detta skapar man upp ett session och tilldelar sedan den sessionens ID till de messages som skall hanteras av den.

En viktig property jag vill nämna innan nästa steg är *TTL* (time-to-live). Här bestämmer man hur länge ett message får "leva", när det bestämda tiden har gått ut så kommer det inte längre skickas, med undantag för messages som redan har blivit låste för att skickas.

Hur ser det då ut med felhantering inom en service bus queue?

I en queue finns det inbyggt ett par kritiska funktioner som leder till en väldigt hög pålitlighet. Den första biten kallas för *DLQ* (dead lettering queue), vilket är ytterligare en queue, fast det enda sättet att posta till den är genom ett message som antingen inte kunde skickas till någon mottagare eller som inte kunde processeras, detta går att kontrollera genom skräddarsydd felhantering. Detta ger oss möjligheten att undersöka orsaken till varför just detta message inte gick igenom och hur vi kan fixa problemet.

När ett message har placerats i *DLQ* så gäller inte längre *TTL*, det betyder att det går att inspektera ett message som tekniskt sätt borde ha tagits bort ur vår queue.

Det finns även stöd för att hantera dubletter, om queuen upptäcker ett message som har ett *messageId* som redan har loggats under tidsfönstret så skickas den första som vanligt, medans kopian ignoreras och släpps från vår queue.

Tidsfönstret för detta kan ställas in manuellt, och skiftar mellan 20 sekunder upp till 7 dygn. Har man en stor mängd messages så är det viktigt att detta fönstret är så litet som möjligt då alla Id'n inom samma tid måste loggas och jämföras med samtliga messages innan de kan skickas vidare. Detta leder då naturligtvis till en högre mängd processering.

En queue kan som sagt skicka ett message till en konsument, men om vi skulle vilja skicka samma message till ett flertal mottagare? Hur gör vi då?

Det är då **Topics** kommer in i bilden:

Topic:
![image](_posts\Images\TopicBus.png)

Ett topic fungerar på ett liknande sätt som en queue, den stora skillnaden är att ett topic har stöd för att skicka ut samma message till ett flertal mottagare. Detta ser lite annorlunda ut än en queue i praktiken, men resultatet är precis detsamma som i en queue. 

Det som är annorlunda processen är att applikationerna inte direkt kommer åt vårt topic som de hade gjort med en queue. Istället använder man sig av subscriptions för att ta emot en kopia av ett message, när alla subscriptions sedan har fått sin kopia så tas vårt message bort ur topics. 

Topics har precis som queues anpassningsbarhet, du vill antaligen inte att varje message skall levereras till samtliga subscriptions på din service bus, då de inte innehåller relevant information för samtliga subscriber. Det går då att ställa in filter på messages, topics har stöd för tre olika vilkor:

- SQL Filter: Detta använder sig av ett SQL-likt vilkorsuttryck för att kvalificera eller diskvalificera en subscriber.
- Boolean Filters: **TrueFilter** gör att alla inkommande messages tas väljs för en specifik subscription, medans **FalseFilter** ser till att inga messages väljs.
- Correlation Filters: Med correlation filters kan man välja att undersöka ett visst/eller flera properties på ett message och sedan avgöra om det skall väljas för en subscriber. Det mest vanliga är att använda sig av **CorrelationId* men det går att använda sig av vilken lagrad property som helst, förutsatt att den existerar på vårt message.

Med filter kan man även inlkudera actions, detta betyder att ett message som har kavlificerats och kopierats kan modfieras innan det tas emot av mottagaren, och eftersom det är en kopia så påverkas inte orginalet. 

Från en systemutvecklares synpunkt så är fördelarna med Azure Service Bus väldigt påtagliga, men för att sätta de i klartext så har jag målat upp två grafer:

![image](_posts\Images\API-requests.png)

Här ser du antalet requests vårt API tar emot under en given tidsperiod. DÅ det är ojämt så betyder det att vissa tider av dagen kräver mer processering än andra, detta leder till att kapaciteten måste anpassas till den högsta toppen för att klara av dessa requests, om man inte använder sig av Azure Service Bus. Nedan kan du se hur processeringen ser ut med APIet kopplat till en service bus.

![image](_posts\Images\queue-processing.png)

Eftersom alla messages placeras i en kö, och hanteras i samma takt oavsätt mängd så betyder det att vi kan lägga vår kapacitet på en median av det behov som uppstår. Med detta så slösar vi mindre kapacitet under de timmarna som den inte behövs, det tåls att säga att under *peak hours* så kommer hanteringen gå långsammare. Men kapaciteten möter behovet på ett helt annat sätt.




-----

Nu vet jag att inget av det jag har sagt hittils har tagit itu med de farhågor du har angående att använda sig av molnet för vår klassificerade information. Därför kan vi nu kika lite på säkerhet.

Molnet är ett ganska skrämmande koncept på grund av många anledningar, för mig har det alltid vart frågon om att kunna komma åt sin information från vartsomhelst. Om jag kan göra det så kan väl vemsomhelst göra det?

Med Azure Private Link så ser verkligheten lite annorlunda ut, och jag ska förklara varför.

Den första uppenbara fördelen är att vi får tillgång till resurser vi inte har för tillfället. Automatisk skalning, garantier när det gäller upptid och professionell support vid behov (utan att behöva anställa någon för samma syfte).

Här är en graf som lite snabbt beskriver förloppet med en Azure Private Link:

![image](_posts\Images\Azure-private-link.png)

För att bryta ner vad allt detta betyder kan vi börja med ExpressRoute.

ExpressRoute tillåter oss att förlänga våra lokala nätverk till Microsofts moln genom en säker anslutning, expressroute går inte över det *publika* nätet utan använder sig av det som kallas *Microsoft backbone*. Vilket är ett sorts internet som körs via hundratals network POP's (points of presence) och datacenter. ExpressRoute är även [säkerthetscertifikerat](https://docs.microsoft.com/en-us/compliance/regulatory/offering-home?view=o365-worldwide) av många pålitliga källor, anställer ungefär 3500 personer och har en årlig budget på 1 miljard USD eller 874 miljoner kronor. 

Med expressroute kan man skapa säkra länkar mellan sin lokala infrastruktur och Azure datacenter, de accepterar enbart publika IP addresser, men med hjälp av NAT (network adress translation) så kan en privat IP address användas.

När vi väl är framme i molnet via vår säkra länk så sköter vi allting inuti en virtuell maskin som befinner sig inuti ett subnet. Även här kan man styra sin säkerhet genom något som kallas för **NSG** (Network security groups), med NSG så kan vi introducera en rad olika säkerhetsfunktioner:

- Åtkomstkontroller lindade runt vårt subnet och vår VM.
- Inspektera inkommande traffik för att godkänna eller underkänna inkommande paket.
- Variabler på paket:
    - RULE NAME
    - Source IP
    - Source Port
    - Destination IP
    - Destination port
    - Direction (inkommande eller utgående)
    - Action (godkänd/icke godkänd)
    - Prioritet (Lägre = högre prioritet, som med en queue eller topic)
- Isolera traffik mellan specifika subnet.
- Kontrollera traffik från det offentliga nätet, ge en viss IP läs- eller skrivåtkomst

Fortfarande inuti vårt subnet har vi det som kallas **Service Endpoints**, detta ger oss en route via Backbone-nätet. Och ger oss möjlighet att helt exkludera det publika nätet från att ha åtkomst till våra kritiska resurser genom att använda oss av en privat IP adress för att ansulta till en Azure service offentliga endpoint.

Det går även att skapa regler för dessa anslutningar i t.ex form av subscriptions och mängd dataöverförning tillåten.

För att sammanfatta allt jag precis gick igenom så finns det i dagsläget väldigt bra möjligheter för en molnlösning som har en hög säkerhetsnivå. Och med hjälp av alla de verktyg som finns tillgängliga så ser jag ingenting förutom möjligheter när det kommer till att använda sig av molnet för vårt framtida arbete.


# References
https://www.youtube.com/watch?v=HrK1UlPBkEY - Azure service bus
https://www.youtube.com/watch?v=vVDql7IKneg - Azure private link
https://www.youtube.com/watch?v=gxsitRRgylI - Azure virtual network
https://www.youtube.com/watch?v=vgnSLuf_zj0 - Azure network security groups
https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview - Vnet
https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues - DLQ
https://docs.microsoft.com/en-us/azure/service-bus-messaging/private-link-service - Private Link




