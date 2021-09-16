---
layout: post
title: Inlägg 16-09-2021
subtitle: Serverless
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [Azure, serverless, cloud]
comments: true
readtime: true
---

# Serverless

I detta inlägget skall jag beskriva vad Serverless och FaaS(Function As a Service) 
innebär, jag kommer även gå igenom hur jag har arbetat med Azure i form av våran
C# kalkyklator.

## Vad är Serverless computing?

När man säger *serverless* så syftar man på en metod där backend-tjänsterna enbart 
blir utnyttjade vid behov. Man gör det möjligt för användaren att skriva och 
publicera kod utan oro över den underliggande infrastrukturen. 

Ett företag som använder sig av en *serverless*-tjänst betalar enbart för den del 
av processering som de faktiskt använder, istället för att betala för en statisk
mängd, detta innebär även att man har inbyggt automatisk uppskalning i sin tjänst.

Det är värt att nämna, trots att det kallas för *serverless* så använder man sig
av fysiska servrar. Men utvecklarna behöver inte känna till dem.

## Varför använda sig av Serverless computing?

Varför skall man då som kunnigt IT företag använda sig av serverless computing?
Det finns säkert en handfull människor på företaget som har kunskapen och kompetensen
för att kunna driva egna servrar, eller så kan man investera i en IaaS-tjänst där
man har full tillgång till sin infrastruktur. 

Det finns ett par goda anledningar till varför det skulle kunna bara att föredra
serverless över alternativen, även för de tekniskt begåvade:

  - Kostnad:
    
    Då man enbart betalar för precis den mängd processering som används så blir det 
    monetärt fördelsaktigt att investera i serverless. Då slipper man som företag 
    ändra sina avtal och eventuellt sälja av/köpa ny hårdvara för en lokal service.
    
  - Upp- och Nedskalning:
    
    Inom affärvärlden finns det väldigt få konstanter, men det enda som är garanterat
    är att ingenting ligger kvar på samma ställe förevigt. Medans det kanske för tillfället
    kan vara mer pålitligt och kostnadseffektivt att använda sig av andra alternativ så
    är man allting en stor upp eller nedsving iväg från att behöva göra stora ändringar
    i sin vardag. Med serverless så sköts den biten automatiskt, dina kostnader stiger 
    om ditt behov stiger, samtidigt som det sjunker om ditt behov sjunker. 
    
  - Utförlighet:
    
    Med mindre fokus på back-end så kommer de utvecklare som är tillgängliga kunna lägga
    mer tid på andra delar av systemet. Samtidigt blir det mycket enklare att skapa 
    mindre funktioner som utför färre saker, därav utöva [SOLID](https://en.wikipedia.org/wiki/SOLID) 
    principerna i sitt arbete.
    
   - Tid till marknad
   
     Detta går hand i hand med det som jag tidigare nämnde om att utvecklarna får mer
     utrymme för att arbeta med andra delar av projektet. Inte bara hinner man göra ett
     antagligen bättre arbete, men även skär man ner på den totala tiden för projektet.
     
 ## Vad är då skillnaden mellan FaaS och Serverless?
   
 Det snabbaste svaret på denna frågan är ett ord: **Abstraktion**.

 Inom utvecklingsvärlden använder vi oss av många olika verktyg för att kompilera 
 de olika termer och språk som vi använder, man tar ett koncept som verkar främmande.
 Och sätter sedan in det i ett bekant scenario, t.ex psuedokod där man i klartext beskriver
 vad programmet skall görra och i vilken ordning, innan man sedan börjar skriva koden.
 
 **Faas** ligger en grad högre än servreless på abstraktionsskalan, då fokus enbart
 placeras på små funktioner som skall kunna anropas från en *outside prompt*. 
 
 Det är väldigt svårt att säga vilket skulle vara bättre än det andra, som i det mesta
 här i livet så beror det helt på situationen och dina specifika behov. 
 
 Ett exempel på när serverless skulle vara mer användbart än FaaS är om du skulle
 deploya en slags eShop. Där är den enklaste vägen att utveckla en applikation som
 tar hand om inventarie, kundvagn och betalningssystem. 
 
 Däremot om du har ett existerande system där du behöver lägga till någon sorts individuell
 funktion som daglig statistik hade FaaS vart ett bättre alternativ.
 
 ----
 
 # Azure calculator
 
 Jag har tillsammans med Sandra deployat en väldigt primitiv kalkylator till Azure, både genom
 den inbydga webbportalen samt via Visual Studio. 
 
 ![image](https://user-images.githubusercontent.com/70150296/133639189-0b5bcda6-9272-474f-abd2-5ee2ca73dbce.png)
 
 Funktionen tar in två parametrar via ett http-request som den sedan adderar ihop. Kodmässigt händer det inte speciellt mycket här
 men målet var inte att göra en komplex applikation, utan att publicera en fungerande applikation till Azure.
 
 Vi använde oss av två olika metoder för att publicera, den första var att använda oss av [Azures webbportal](https://portal.azure.com/#home).
 Här är det möjligt att importera in en så kallad *Http-trigger* template. Den fungerar i princip som ett API-call då den tar in 
 en string variabel via request och skickar tillbaka samma string inkluderad i en mening. Från detta är det ganska enkelt att manipulera
 koden så den hanterar t.ex två int som input och skickar tillbaka summan av dem.
 
 ![image](https://user-images.githubusercontent.com/70150296/133640383-d49c4c17-25cb-4053-bef5-07cffb026edd.png)
 
 Vi startade även upp samma projekt i Visual Studio där förloppet var ungefär detsamma, jag föredrar definitivt att använda
 en miljö som Visual Studio eller VSCode för att bygga applikationen dock. Det finns inte i närheten samma stöd i Azure-prtalen,
 fördelen där är främst att det är enkelt att testa dina request. Det är däremot inget som är omöjligt via VSCode eller studio.
 
 Testningen av applikationen sköttes i detta fallet manuellt, har tyvärr inte riktigt haft tid att bygga tester/pipeline till
 projektet, detta är något jag kommer göra i efterhand. Så för tillfället finns det inga konkreta tester att peka på, det jag 
 gjorde var egentligen bara att testa olika inparametrar via Azure. För att få en känsla av hur det fungerar och vad man kan titta
 efter om något går fel. Jag ser det inte riktigt som en större utmaning att implementera både en CI-pipeline samt enhetstestning
 på en Azure-publicerad applikation, då detta är något vi har gjort relativt mycket under kursens gång vid detta stadiet.
 
 ## Referenser:
 
[FaaS/Serverless](https://www.ibm.com/se-en/cloud/learn/faas)
 
[Getting started with Azure functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-get-started?pivots=programming-language-csharp)
 
 

     
     
     
     
     
    
