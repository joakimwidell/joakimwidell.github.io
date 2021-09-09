---
layout: post
title: Inlägg 07-09-2021
subtitle: The Cloud
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [The-Cloud]
comments: true
readtime: true
---

## Inlämningsuppgift 1



## **Vad är molnet?**


**Molnet har historiskt sätt bestått av tre "huvudtjänster"**: 

 
 
 - **Software-as-a-Service** (**SaaS**)

        **SaaS** innebär att hela applikationen hostas på molnservrar, 
        och användare får åtkomst till dem via internet. 
        
        Man kan säga att **SaaS** är som att hyra ett hus, 
        hyrevärden ansvarar för den övergripande driften av huset. 
        Men hyresgästen får till största del använda huset som att de bor där.

 
 
 - **Platform-as-a-Service** (**PaaS**)

        I **PaaS** modellen så betalar inte företagen för hosting för sin applikation,
        utan de betalar för att få tillgång till de verktyg som kan behövas
        så de kan utveckla applikationer. 
        
        Allt från hårdvara till operativsystem.
        För att bygga på den tidigare metaforen med ett hus, så erbjuder **PaaS** de verktyg
        man skulle behöva för att bygga ett hus, istället för att hyra det.

 
 
 - **Infrastructure-as-a-Service** (**IaaS**)

        Med denna modellen är det som titeln hintar, infrastruktur man betalar för.
        För våra syften innebär det att man hyr servrar och lagringsutrymme från 
        en molntjänst. 
        
        Detta skulle kunna liknas vid att man hyr en bit land, och sedan
        bygger t.ex ett hotell eller en restaurang där. 
        
        Du hyr marken, men alla saker som
        placeras på marken är du själv ansvarig för.



Dessa tre var under en längre tid de kategorierna som man grupperade in molntjänster i,
men på senare år så har en fjärde kategori dykt upp, då pratar vi om **FaaS**:



 - **Function-as-a-Service**

        **FaaS** eller *serverless computing* som det har kommit att kallas byggs på en princip
        som är väldigt simpel men samtidigt komplicerad.
        
        Du betalar enbart för det du använder.
        
        Hur fungerar detta då? På högsta nivån av abstraktion och med hjälp av vår hus-metafor
        kan man säga att du har ett hus som anpassar sig efter ditt beteende.
        
        Använder du badrummet i 10 minuter, då betalar du för badrummet i 10 minuter.
        
        Sitter du bara på en stol i ditt vardagsrum och inte gör något alls 
        så betalar du kanske enbart för den stolen du sitter på.
        
        **FaaS** tjänster även om det kallas för *serverless computing* körs i verkligheten på servrar.
        Precis som alla andra molntjänster, men de kallas för *serverless* eftersom de inte ligger på 
        någon dedikerad maskin, och företagen som bygger applikationerna inte behöver underhålla några servrar.

        Tanken är att man skall kunna bygga en mer modulär applikation där endast de delar som krävs för en operation
        är aktiva, därav öka skalbarheten av applikationerna då de mer sällan behöver oroa sig för konflikter. 
        I princip kan man säga att det är på nögsta nivån av vad vi kallar *SOLID programming*. 
        Då man både optimerar applikationen och den maskinvara som krävs för att applikationen skall fungera.


![SOLID](https://miro.medium.com/max/606/1*yO6YGExWLJl5VOUL61xXvQ.jpeg){: .mx-auto.d-block :}




## **Fördelar och nackdelar med molnet**



## Vi kan väl börja med att lista av några fördelar:

 
 
 - Åtkomst

        Idag har I princip alla företag något behov av internet, mer eller mindre. 
        De som har ett större behov t.ex Google, Amazon eller FaceBook kanske kan se ett större värde i att också 
        investera i maskinvara för att kunna bedriva verksamhet på nätet. 
        Men om man vänder på det och tar t.ex en familjeägd hattaffär i Göteborg, de har rimligtvis inte samma behov
        av de maskiner som krävs för att hosta en hemsida och lagra ett kundregister. 
        Där kommer molnet in, de kan istället för att betala dyra pengar för en maskin 
        som de sedan även kommer behöva underhålla och bemanna. Bara beställa en **SaaS** eller **FaaS** molntjänst
        där de kan för en måntlig kostnad få ut samma funktionalitet. 

 
 - Skalbarhet

        Låt oss säga att ditt företag har införskaffat ett serverrum, 
        och ett gäng dedikerade maskiner för att bedriva verksamheten.
        Helt plötsligt upplever ni antingen enorma framgångar, eller enorma motgångar.
        Med en molntjänst har man alltid möjligheten att
        anpassa sina utgifter och sin kapacitet till den verkliga situationen. 
        Om du sitter med egna fysiska servrar så löper du alltid risken
        att missa antingen inkomster eller att begravas 
        under dina utgifter om företaget fluktuerar.

    Det finns en hel hög med ytterligare fördelar, t.ex Säkerhet, Miljöpåverkan, Samarbetsmöjligheter, Rörlighet osv. Jag tänker inte
    gå in på djupet på dem idag. Men för att sammanställa så har molnet gjort att vi inte bara har mer professionella möjligheter utan även 
    att vi kan vara sammankopplade på ett helt nytt sätt än tidigare. 

## Vad säger du om att kolla på lite *nackdelar*?

 - **Oseriösa aktörer**

        Precis som i allt annat så kommer man stöta på företag som antingen 
        inte riktigt vet vad de gör, eller alternativt har onda avsikter.
        När det kommer till en molntjänst så blir detta kanske ett större 
        problem än vad det tidigare hade varit. Då grundprincipen är att man
        skall kunna ha tillgång till sina system och sin data vartsomhelst betyder 
        det även att med rätt verktyg, kan någon annan även få
        tillgång till dina saker. Jag nämner oseriösa aktörer här istället 
        för säkerhet, då man har sätt att molnet, i rätt händer, faktiskt
        har lett till en ökad säkerhet. Då moderna molntjänster har valt att göra 
        säkerhet till en stor del av deras egna verksamhet. 
        Ingen vill köpa din molntjänst om du inte kan bevisa att deras 
        information och data är säkra hos dig.
    
 
 - **Teknologiskt krävande att förstå**

        Molnet är inom modern historia ganska nytt, och ett relativt svårsmält koncept 
        för de som inte har en djupare föreståelse för det.
        För en lekman så låter det som science fiction, du skickar upp dina filer i molnet, 
        och sedan kan du komma åt dem precis när du vill.
        Det är fullt förståerligt att man kanske inte vågar chansa sin 
        verksamhet på något som man inte förstår sig på. 
        Och just en förståelse av molnet, som vi försöker skaffa oss nu under denna kursen, 
        är inget man kan råka på om man inte är extremt intresserad av teknologi.

 
 - **Mindre kontroll**

        Det är inte överraskande att man har mindre kontroll över sin miljö via molntjänster, 
        jag skulle kunna göra ett argument för motsatsen. Men för att spela djävulens advokat: 
        Om du jobbar på ett större IT-företag där tillgång till både hård- och mjukvara finns, och det uppstår ett
        akut behov av en systemåterställning, eller en utökad kapacitet av lagring. 
        Då skulle du kunna gå ner till företagets serverrum och göra det på studs,
        medans om du förlitar dig på en molntjänst så skulle det krävas kommunikation till dem,
        samt att de går igenom alla steg nödvändiga för att återställa just ert
        system. Detta tror jag inte kommer vara ett större problem längre fram i molnets utveckling, 
        med renässansen av **FaaS**. Men det är något att tänka på.

![ProsAndCons](https://cdn.business2community.com/wp-content/uploads/2014/09/The-Pros-and-Cons-of-Cloud-Technology-Every-Marketer-Should-Know.jpg){: .mx-auto.d-block :}



## Prisundersökning molntjänster


        För att inte bli för långrandig på ämnet så är dagens situation precis som väntat. 
        Vill man få den billigaste möjliga tjänsten så är det någon av
        "jättarna" som gäller. Beroende på dina behov så är det billigaste alternativet någon av: 
        Azure, Amazon Work Station eller Google Cloud. 
        Detta betyder inte nödvändigtvis att de är rätt för dig dock, 
        lokala/mindre molntjänster har fördelen av att kunna erbjuda mer personlig assistans, 
        och för ett företag utan en egen IT-avdelning kan det vara ett smartare alternativ.
        Men för att ge en siffra på hur stor skillnaden faktiskt är mellan de större 
        företagen och de mindre så fick vi betala nästan precis dubbelt så mycket för en 
        Linux 2 CPU 8GB RAM 10GB Förvaring hos en nordisk molntjänst än vad det hade kostat hos "jättarna".

        Min slutsats är att i nästan samtliga fall så vinner Goliat över David i denna aspekten av livet, 
        och tyvärr så är det nog mer prisvärt för företagare att bidra till Goliats fortsatta tillväxt.


![googlevscloud](https://user-images.githubusercontent.com/70150296/132320588-5e264356-af4a-4066-b9d1-d1fad3e1ef7d.png){: .mx-auto.d-block :}


## Referenser

       **https://www.cloudflare.com/learning/cloud/what-is-the-cloud/** **https://www.salesforce.com/products/platform/best-practices/benefits-of-cloud-computing/** **https://www.nibusinessinfo.co.uk/content/disadvantages-cloud-computing**