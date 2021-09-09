---
layout: post
title: Inlägg 09-09-2021
subtitle: CI Pipelines
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [CI-Pipelines, GitHub-Actions]
comments: true
readtime: true
---

### Vad är en CI pipeline?

<p1>CI eller *Continuous Integration* som det heter är ett sätt att automatisera testning av din kod. 
Pipelinen bygger kod, kör test och kör programmet på ett säkert sätt.

Man kan säga att det är en körbar specifikation av de steg en utvecklare behöver
följa för att leverera en ny version av ett program.</p1>

## Hur fungerar då en CI pipeline?

För att beskriva detta på det mest utförliga sättet möjligt så tänker jag bryta ner processen i steg.

- Source: 
    För det mesta så är en *pipeline run* aktiverad av en ändring i källkoden, ändringen skickar en 
    notis till CI-verktyget och detta aktiverar pipelinen. Det finns många fler olika sätt att åstadkomma detta
    och utvecklaren kan själv avgöra vilket event som skall aktivera pipelinen. Man kan t.ex lägga in att den kör
    vid varje push på en specifik branch, eller vid både push och pul. Det finns många användbara möjligheter,
    men ett bra ställe att börja på för att komma in i det är att enbart lägga vid push. Det är enklare att
    säkerställa innehållet i din config.yml fil på detta viset.

- Build:
    Här kombinerar vi källkoden och dess dependencies, du kanske har skrivit ett program i C#, då måste
    pipelinen ladda ner och installera språket för att kunna köra programmet. Undantag för detta är om
    du skriver med Ruby, Python eller JavaScript då dessa kommer inkluderade. 

    Målet med detta steget är att skapa en körbar instans av vårt program. 

- Test:
    I detta stadiet så kör vi våra tester för att validera både vår kod och beteender av vårt program. 
    Man kan kalla detta skedet av pipelinen för ett slags säkerhetsnät, det finns där för att stoppa
    buggar innan de kan komma ut till användarna.

    Testerna i sig är något man behöver skriva själv, detta görs bäst genom det vi kallar TDD eller 
    *Test Driven Development*. Detta är en princip där testerna ligger i centrum för utvecklaren 
    när denne skriver sin kod. 

    Beroende på hur stort och komplext programmet är kan detta stadiet ta antingen ett par sekunder eller flera timmar
    och ifall pipelinen misslyckas under testerna kan detta uppmärksamma utvecklaren till problem i koden som de 
    inte förutsätt när de skrev koden. Här är det otroligt viktigt att våra felmeddelanden effektivt visar på vad
    som har gått fel så utvecklaren kan ta hand om problemet medans de fortfarande har sin tidigare lösning
    färsk i minnet.

- Deploy:
    När alla föregående steg har lyckats är det nu dags att starta upp programmet. Det finns vanligvis flera olika
    miljöer som programmet kan startas i, till exempel en *beta* eller *staging* miljö som används av utvecklarna eller
    en miljö som brukar kallas *production* för användarna.
    


## Finns det några fördelar med en pipeline?

Det enkla svaret är: Ja.

För att gå in lite mer i detalj finns det en hel radda med fördelar här, den första fördelen går i hand med det vi redan
har pratat om. Testningen av våra program sker mer effektivt, skulle vi manuellt testa att starta och köra testen med samma
interval så hade vi behövt lägga en väldans massa tid på det. Sen har vi en mycket mer transparent utvecklingsprocess, 
om man har en QA inkluderad i projektet blir dennes jobb mycket enklare då man bara kan gå in och titta på historiken över 
alla gånger pipelinen har körts, och kan därav inditifiera eventuella problem enklare.

En annan stor fördel är att man får veta nästan i realtid om det du har jobbat med faktiskt fungerar, och detta leder till både
en snabbare inlärningsprocess och etrt mycket mer utpräglat ansvarsystem, där det blir tydligt vem som har gjort eller inte gjort vad.


## Vad definerar en *bra* pipeline?

Det finns ett par kriterier som man tittar efter i en *bra* pipeline. Det är en kombination flera saker:

- Hastighet:
    Om din pipeline inte lyckas skapa feedback på mindre tid än det hade tagit för en utvecklare att identifiera problemet så är
    den inte speciellt användbar. Om din pipeline har en *runtime* på ca 1 timme, så betyder det att ni som lag är begränsade
    till ungefär 7 st genomgångar per dag. Detta kan leda till att man inte pushar upp sina ändringar lika ofta, och detta i sin tur
    leder till att buggar uppstår och mer tid behöver läggas på att fixa dem. 

    Behöver man som utvecklare lägga mer tid på att skapa och observera sin pipeline så är det i slutändan negativt för
    projektet.

- Pålitlighet:
    En pålitlig pipeline ger alltid samma sorts output oavsätt input, med detta menar jag inte att alla felmeddelanden är
    indentiska, det hade vart ett ganska bristfälligt verktyg. Tanken är att om jag stöter på ett fel inom samma område
    som en annan utvecklare på projektet så skall felmeddelandet reflektera ungefär samma sak. Detta är för att det skall finnas
    tydliga och konsekventa sätt att identifiera precis vad som har gått fel.

    Det är även viktigt att ha en stabil runtime på sin pipeline, detta är till stor del för att undvika frustration hos utvecklaren.

- Pricksäkerhet:
    All autmation är positivt, så länge det producerar minst lika effektiva resultat som det manuella arbetet. Den stora baksidan till
    automation är att om något går fel, har vi inte nödvändigtvis samma kontroll över processen. Och lösningen kan därav bli lite mer
    komplex än om en utvecklare hade suttit manuellt med uppgiften.

Därför använder man sig ofta av CI/CD verktyg som kan modellera både simpla och komplexa arbetsflöden. Den stora fördelen av just autmation
är samma som i alla andra områden av livet, manuella misstag blir nästan omöjliga. 

---


### Vår pipeline

Jag har tillsammans med [Sandra](https://github.com/SaAlgervik) lagt till en pipeline på ett projekt som vi båda var delaktiga i.

Vi började med att försöka implementera en template på en action via GitHub, vi hade inte så mycket framgång med detta då vi är 
relativt nya till processen och det fanns mycket orelevant text inkluderad i dessa templates. Vi beslutade oss istället för att
bygga upp pipelinen i samma anda som det demoexempel vi fick av [Stephan](https://github.com/skjohansen) 2021-09-08. Börja med det
enklaste och bygga upp därifrån.

Vi körde fast en sväng när det kom till att köra våra kommandon på korrekt ställe, men efter lite hjälp från Stephan så löste vi det
genom att referera direkt till den path vi behövde komma åt i våra run statements. 

Inkluderat undertil är ett screenshot av vår yml fil, där pipelinen har byggts upp.

![pipelines](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/CI-pipeline.png?raw=true)

Kortfattat så enablar vi pipelinen varje gång man pushar till GitHub, vi kör den på senaste versionen av Ubuntu, detta hade likväl kunnat 
vara windows, men vi körde på detta. Vi använder [Checkout](https://github.com/actions/checkout) och [setup-dotnet](https://github.com/actions/setup-dotnet)
för att komma åt vårt repo samt ha tillgång till en dotnet core cli miljö.

Vi definerar vilken version av dotnet som skall användas och kör sedan två run commands. Först kör vi en dotnet restore, så vi har tillgång till NuGet utan
att behöva köra det under nästa steg, som är dotnet build. Vi kör build för att starta upp projektet i vår tillfälliga miljö och inväntar sedan resultatet.

Det tog ett tag att få till rätt syntax, men vi fick det tillslut att gå igenom och visa en hjärvärmande grön checkmark. 

![Success](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/Success.png?raw=true)


### Referenser

[CI Pipeline](https://semaphoreci.com/blog/cicd-pipeline) [Github actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
    





    
            



            
