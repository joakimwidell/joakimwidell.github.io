---
layout: post
title: Inlägg 14-09-2021
subtitle: Containers
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [Docker, CI-Pipelines, GitHub-Actions, GitHub-Publish]
comments: true
readtime: true
---

# Containrar och orkestrering

<p1>
Jag har arbetat med containrar tidigare, men i detta avsnitt  lärde jag mig lite nya sätt
att använda dem. Då vi har fått en lista av punkter på innehåll i denna uppgiften
tänker jag stegvis gå igenom dem.
</p1>



## Lokal mjukvara

Nedan finner ni de verktyg som jag har installerat lokalt på min dator för att 
möjliggöra/förenkla processerna:

VSCode:

Jag valde att använda VSCode för detta då vanliga Visual studio har en tendens
att inkludera mycket som inte riktigt behövs för uppgiften. Detta hade självklart
inte påverkat min container, men jag föredrar lättvikten.
        
Extensions till VSCode:

Jag har använt mig av ett par olika extensions till denna uppgiften, vissa är
nödvändiga som [Omnisharp C# for Visual Studio Code](https://github.com/OmniSharp/omnisharp-vscode)
medans andra är enbart för bekvämlighetens skull. Där räknas [Docker for Visual Studio Code](https://github.com/microsoft/vscode-docker)
definitivt in. En kort beskrivning av vad Docker-extensionen gör är att den bidrar med mallar för
olika containrar i form av Dockerfiles och docker-compose.yml. I detta fallet använde jag en ASP .NET Core 
mall, och fick med de nödvändiga raderna för en grundläggande container.

Här kan man se hur filerna ser ut direkt vid import:

![image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/dockerfile-template.png?raw=true)
----
![image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/docker-compose-template.png?raw=true)

**Jag kommer senare i inlägget gå igenom hur dessa Dockerfiles fungerar** 


Docker Desktop/WSL 2:

    Precis som VSCode är detta något som jag har arbetat med tidigare, Docker desktop
    ansvarar för att köra och bygga våra containers. Det körs på WSL vilket är en komplett
    Linux kernel som kan användas på Windows, [här](https://docs.docker.com/desktop/windows/wsl/) 
    kan man hitta instruktioner för att installera det.

Windows Terminal:

    Jag använder mig av Windows Terminal av den enkla anledningen att allt finns på samma ställe.
    Här kör vi i Powershell våra docker commands, som *compose-up*, *images*, *rmi* osv..

GitHub Desktop:

    Jag har märkt att GitHub Desktop är något som jag använder mindre och mindre ju längre in i studierna
    jag kommer. Det är fortfarande ett väldigt bra verktyg för att kunna klona ner repositories och för
    att säkerställa vad som pushas upp. 

Utöver dessa program, som egentligen redan fanns på min dator har jag inte behövt installera något nytt för
denna veckans uppgift. 


## Göra om applikationen till en container/Vad är en dockerfile?

När man skall bygga en container i docker så finns det ett par grundläggande steg som kan ha lite skarp
inlärningskurva, det första är att skapa sina dockerfiler. Först har vi den som enbart heter *Dockerfile* 

![image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/dockerfile-template.png?raw=true)

Som man kan se här här är det ingen större mängd text att oroa sig över, och då detta kommer från en template
så skulle vemsomhelst kunna använda sig av det. Men det är viktigt att veta vad man gör, därför ska vi nu gå igenom
vad som sker i filen.

Först hämtar vi ner .NET/aspnet via mcr (microsoft container registry) som ett *lager* med **FROM**, sedan lägger vi till filerna
till vårt aktuella directory med **COPY**. **EXPOSE** Avgör vilken port containern skall lyssna på för sin connection. Detta 
är allt då en del av **BUILD** i denna Dockerfile. 

Därefter kommer vi till **ENV**, vilket är ett sätt att koppla ihop en nyckel med ett värde, i detta fallet sätter vi
*ASPNETCORE_URLS* till *http://+:80*, vilket man kan se tidigare är den porten vi vill att containerns connection skall
ligga på.

Nu har vi landat på den *största* delen av vår Dockerfile, som vi kallar för *build* sektionen. Vi börjar med att hämtar ner
ytterligare ett lager i form av .NET/sdk. Det placeras i vårt **WORKDIR** (working directory), sedan kopierar vi ner filerna i projektet
genom **COPY** där vi först väljer vår fil, och sedan vart den skall hamna i vår container. Direkt efter detta måste vi säkerställa
att vi har tillgång till alla nädvändiga dependencies för projektet, därför kör vi en **RUN** dotnet restore, detta är då ett 
*shell-command* och emulerar att man skriver samma sak i Powershell. Det går även att köra **RUN** som en *executable* form, då ser syntaxen
ut såhär: **RUN [exe, param1, param2]**.

Nu har vi alltså våra dependencies tillgängliga, och då kör vi återigen **COPY** på innehållet i containern, vi sätter sedan vårt
**WORKDIR** till samma path som vi har våra filer och kör sedan ytterligare ett shell command i form av **RUN dotnet build**

Vi bygger där applikationen till via -Release till /app/build, efter detta kör vi en **RUN dotnet publish** som kompilerar applikationen
och publicerar den till /app/publish.

I sista steget tar vi vårt **WORKDIR** där summan av Dockerfilen finns, och sätter en **ENTRYPOINT** till *dotnet* med path efteråt.
Detta gör att vi kan köra vår container på samma sätt som en EXE fil, på simplaste nivå betyder det klicka och kör. Även om det krävs lite
annat för att få tillgång till containern.

Detta leder nu till den andra dockerfilen, *docker-compose.yml*

![image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/docker-compose-template.png?raw=true)

Det docker-compose bidrar med är möjligheten att köra samtliga containers i din applikation med ett command. 

Till att börja med så definerar vi vår version av Docker Compose, i detta fallet är det *3.4*,
sedan kommer vi in på *services* sektionen, den definerar alla olika containers vi vill skapa. I vårat fall
är det bara en container, men här går det i teorin att skapa flera olika om din applikation kräver det.

Vi definerar sedan vår container som heter *simplewebhalloworld* (klonade och importade template i samma vända
därav bristen på unikt namn 😅). Image delen av denna filen är antagligen inte nödvändig enligt vad jag har läst tidigare, då detta är något som bara behöver defineras om man har en pre-existerande image som man vill använda sig av, men som man säger *if it aint broke...*. Vi lägger sedan till **build** med **context: .** och **dockerfile: ./Dockerfile** som säger vart vår **Dockerfile** samt **docker-compose.yml** finns. Vi pekar slutligen på den port vi skall använda, och syntaxen för detta är **[HOST:CONTAINER]**, vi har valt att lägga port 80 på båda då det fanns med i uppgiften att använda sig av denna. Skulle man bara skriva in en port här kommer den enbart definera **CONTAINER** och **HOST** får en slumpmässad port tilldelad. 


Man kan säga att compose är en trestegs-process:

    1. Definera miljön för din applikation med en **Dockerfile** så att den kan reproduceras självständigt
    2. Definera tjänsterna som bygger upp din app i en **docker-compose.yml** fil så att de kan köras
       tillsammans i en isolerad miljö.
    3. Kör **docker compose up** för att starta och köra hela applikationen.

------

## Vår GitHub Pipeline

![image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/docker-pipeline.png?raw=true)

Vi använde oss till stor del av Stephans exempel i slutändan, innan dess använde vi oss av [GitHub Docs](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions). Jag skall försöka bryta ner processen utan att vara för långrandig.

Vi namnger processen, säger till den att köras vi varje *push*. Sedan definerar vi våra **env** variabler, vi använder oss av *ghcr.io*(GitHub container registry) och en variabel för det aktuella repot.

Sedan defineras vårt **jobs**, den övergripande titeln är **build-and-push-image**. Allting körs på senaste versionen av ubuntu och eftersom vi både vill se innehållet och kunna publicera paketet så ger vi permissions till 
**contents: read** och **packages: write**. 

Sedan kommer vi till de stegen vi skall ta eller **steps**, först kör vi som tidigare [Checkout](https://github.com/actions/checkout) för att få tillgång till koden. Sedan loggar vi in på Docker via [Docker login](https://github.com/docker/login-action) med informationen som finns undertill, **with** pekar på ghcr.io, detta är så att vi inte kopplar upp till dockerhub av misstag. Sedan definerar vi ett användarnamn via variabel, och sist men inte minst definerar vi **password** med hjälp av en **PAT** (personal access token), denna skapade vi via github med permissions för enbart read och write på packages och lagrade sedan som en secret i repot, därav **SECRET_KEY**.

Efter allt detta kör vi [Docker build/push action](https://github.com/docker/build-push-action), vi sätter context till hela **workdir**, enablar det vid **push** och inkluderar senaste versionen av applikationen i form av en tag.



## Hemligheter

![image](https://miodatos.com/wp-content/uploads/2016/10/top-secret-file.jpg)

När det kommer till secrets har vi enbart använt det på ett ställe i applikationen, och det är vid lösenordet till docker login. Man skulle antaligen kunnat använda det på fler ställen och fått en bättre grad av säkerhet på applikationen, men jag ansåg inte det vara nödvändigt i detta fallet.





    