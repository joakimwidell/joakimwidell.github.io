---
layout: post
title: Inlägg 21-09-2021
subtitle: The Cloud (same but different)
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [The-Cloud]
comments: true
readtime: true
---

# Databaser i molnet
---


## Vår applikation:

Vi har skapat en applikation som kan både lagra och raportera data med hjälp av input. Konceptet
är inte överdrivet komplicerat, men fick en bra påminnelse om hur saker inte alltid fungerar som man
tror de gör. En stor del av dagen spenderades genom att sitta och försöka få kopplat på de metoder 
vi skapade för att bygga och kontrollera databasen till vår function, detta är numera känt som
"entity-framework tunnel vision". Då vi båda har jobbat mycket med just entity framework tidigare
så var det naturligt att de metoder vi hade skapat upp för att bygga servern och manipulera/kontrollera
dess innehåll också skulle va inblandade i de request en potentiell användare skall göra. 

Detta är som vi nu vet, extremt oeffektivt och kanske inte ens möjligt via Azure functions.

## Hur fungerar applikationen?

Detta har jag tänkt bryta upp i två delar. Först har vi uppbyggnaden, i form av en konsollapp. Då det i 
slutändan blev väldigt mycket kod i plats så går jag endast igenom den generella strukturen.

Vi valde att bygga databasen med Azure web portal och Visual Studio, viktigt med denna metoden är
att man ser till att all support för Azure finns tillgängligt i Visual Studio under **Tools/Get tools and features**.

Första steget är att skapa sitt Azure Cosmo DB konto, detta kan göras antingen via en *IDE* som VSCode eller via 
Azures webbportal, det viktiga är att man ser noggrant över sina inställningar. Annars kan man råka åka på
onödiga kostnader eller inte ha det stöd man behöver för sin applikation. Nedan kan ni se lite mer ingående
bekrivning av exakt vad inställningarna betyder.

![image](https://user-images.githubusercontent.com/70150296/134193540-57b49707-4758-4874-81fd-bae09008e850.png) ![image](https://user-images.githubusercontent.com/70150296/134194010-24801895-622b-4045-81d7-ff9ebc1b8509.png)

Nu har vi vårat konto igång, då är nästa steg att skapa sitt Projekt. Detta är något man absolut kan göra
i webbportalen, och om inga större kodstycken behövs kan det definitivt vara smidigare. Men för våra syften
skapade vi upp en basic konsollapplikation för .NET Framework. Därefter installerade vi 
*Microsoft.Azure.Cosmos* via NuGet till projektet.

Efter kontot var skapat och NuGet paket installerat började vi med koden, vi hittade en väldigt bra guide 
för denna biten av uppgiften [här](https://docs.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-get-started).

Vi fick ersätta vissa saker i guiden för att anpassa funktionerna till våra syften, det första är att 
det behövdes en unik **EndpointUri** och **PrimaryKey**. Dessa är båda saker som kommer med på köpet
när man skapar sitt konto för sin Azure Cosmos DB, de kan man hitta antingen via sin IDE eller inne på webbportalen
under **Keys**.

När det gäller våra metoder så används det en hel del anrop till metoder som är inbakade i Cosmos Klienten,
ett exempel är:

![image](https://user-images.githubusercontent.com/70150296/134199657-78ebb666-b3e1-44e9-a884-bb0a51a72e48.png)

Det går att skapa databaser i Cosmos Db med både *CreateDatabaseIfNotExistsAsync* och *CreateDatabaseAsync*, dessa
är några av de *inbakade* metoderna jag pratade om innan.

Vi lade även i stöd för att skapa container, lägga till item och för att ta bort databasen:

![image](https://user-images.githubusercontent.com/70150296/134200992-50bf11f4-5986-4c11-b602-e73915fd09bd.png)

![image](https://user-images.githubusercontent.com/70150296/134201026-1da275d1-239d-458f-8628-7f3904ba100b.png)

![image](https://user-images.githubusercontent.com/70150296/134201061-bd8039eb-eb71-40fa-b947-06c1b6910fea.png)

Med detta ur vägen kan vi hoppa på våra functions.

![image](https://user-images.githubusercontent.com/70150296/134201457-91f68e6c-99f3-4627-9e59-4ab483e99c9d.png)

Här har vi använt oss av en template för en *HttpTrigger*, och har därefter lagt in det som faktiskt krävs för att
kunna prata med databasen (inte massa kopplingar till den förra biten..).
Vi placerar all relevant information i variabler, den mest kluriga biten här var att använda sig av 
**ConnectionStringSetting**. Vi lagrade först en lokal variabel i **local.settings.json** för strängen. Sedan när 
den skulle publiceras till Azure behövde vi även skapa en **application setting** i Azure som höll samma sträng 
som den lokala versionen. 

Det man ser är funktionen för att göra ett POST request, användaren skickar in ett namn. Användarens ID och 
status sätts automatiskt, ett meddelande skickas tillbaka som informerar att requestet har gått igenom.

Härrnäst ser man vårat GET request:

![image](https://user-images.githubusercontent.com/70150296/134203511-65e9aace-9e9c-413b-bb94-b2c14973cf07.png)

Det är egentligen samma information som skickas med här som i POST, skillnaden är att vi gör ett Query direkt
innan funktionens *body* körs.

## Möjliga uppdateringar


Förutsatt att jag använder serverless korrekt kommer mina konstnader alltid reflektera behovet av min produkt.

Men när det kommer till att ändra scheman och liknande så kan jag se svårigheter i det, inom en större applikation
är det därför väldigt viktigt att man tänker på detta vid utvecklingens tillfälle. Många funktioner som tjänar ett
mindre syfte över stora applikationer som täcker väldigt mycket mark. Har man använt sig av SOLID under utvecklingen
bör det dock inte vara några större problem att byta eller lägga till de klasser och metoder som krävs för att 
integrera ändringar i systemet.

När det kommer till kostnader så finns det väldigt enklar svar, om jag knappt har några besökare alls. Betalar jag nästan ingenting.
Summan för extremt lite trafik och en Azure Cosmos DB ligger på ungefär 3$, har man väldigt få besökare behövs inte alls
samma hastigheter eller processering.

Om jag däremot skulle få en väldigt stor mängd besökare hade kostnaden kunnat bli mycket högre. För en prislapp på 400$USD
per månad kan man få tillgång till 600x1MIL RU's (Request units) och 1 TB transactional storage. Detta bör räcka ganska långt
för en sida med väldigt lite funktioner och processering.

---

[Get started with SQL API Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-get-started)
[Serverless REST API](https://www.codeproject.com/Articles/1248331/Creating-a-Serverless-REST-API-with-Cosmos-DB-on-2)
[pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/)







