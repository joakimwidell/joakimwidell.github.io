---
layout: post
title: Inlägg 23-09-2021
subtitle: Webbapplikationer i molnet
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [The Cloud, Web Application]
comments: true
readtime: true
---

# Webbapplikatoner i molnet

----

# Vår applikation

Vi har skapat en applikation som tar in en användares namn och lagrar denne
med ett unikt ID i en CosmosDB databas. Applikationen ligger uppe på
Azure och kan nås från vartsomhelst (sitter i nederländerna nu och det funkar kanonbra).

# Kodstruktur

Då vi tidigare har gått igenom hur man börjar med CosmosDB kommer jag bara plocka upp
där vi slutade senast, vi använde oss tillochmed av samma databas för denna uppgift.

Först har vi vår Index.cshtml.cs fil, den innehåller de relevanta metoderna för att
hämta och lagra information i databasen. Vi kan börja med metoden för att hämta ut
information:

```csharp
public async Task<IActionResult> OnGetUsers()
{
    CosmosClient cosmosClient = new CosmosClient(ConnectionString); // Publik variabel
    Database database = cosmosClient.GetDatabase("UserDatabase"); 
    Container container = database.GetContainer("UserContainer");

    // Hämta och presetera data ifrån CosmosDb

    string sqlQueryText = "SELECT * FROM c";

    QueryDefinition definition = new QueryDefinition(sqlQueryText);

    var iterator = container.GetItemQueryIterator<User>(definition);

    while (iterator.HasMoreResults)
    {
        foreach (var item in await iterator.ReadNextAsync())
        {
            var user = new User
            {
                Id = item.Id,
                Name = item.Name
            };
            Users.Add(user); //Publik lista
        }
    }
    return Page();
}
```

Här börjar vi med att starta upp en instans av CosmosClient, detta är en client-side representation
av vårt CosmosDb konto. Den är nödvändig för att kunna interagera med databasen.

Sedan använder vi oss av klienten för att instansiera **Database**, vilket innehåller
operationer som relaterar till själva databasen.

Sen är sista steget i ledet för att nå dit vi vill att skapa en instans av klassen **Container**,
vi har nu tillgång till *insidan* av vår databas, där all enskild data ligger lagrad. 

Sedan definerar vi vår Query som skall skickas till databasen, i detta fallet använder vi oss av en 
select som siktar in sig på hela databasen, då tanken var att en användare skall kunna trycka på
en knapp och se alla användare som finns i databasen. Detta går att ändra genom att antingen
skapa flera knappar med olika querys kopplade till dem eller att helt enkelt ta emot ett input i form
av t.ex namn och använda samma input for att skapa vår **sqlQuerytext**.

Härnäst skapar vi upp en instans av QueryDefinition klassen, denna definerar vårt query till ett
Cosmos Query som vi sedan använder när vi skapar upp instansen av **GetItemQueryIterator** på
våran container. 

Vi ger instansen typen **User** då det är objekten vi itererar igenom, och loopar igenom
resultaten så länge det finns några resultat kvar att titta på.

För varje item som finns i databasen skapar vi en en användare i en lista som sedan skall
presenteras i RazorPages, mer om det senare.

Slutligen gör vi en *return Page();* för att refresha sidan och presentera rätt delar i frontenden.

Härnäst har vi vår metod för att posta användare till databasen:

```csharp
public async Task<IActionResult> OnPost()
{
    CosmosClient cosmosClient = new CosmosClient(ConnectionString);
    Database database = cosmosClient.GetDatabase("UserDatabase");
    Container container = database.GetContainer("UserContainer");
    var user = new User
    {
        Id = Guid.NewGuid().ToString(),
        Name = NewUserName //Publik variabel
    };
    try
    {
       await container.CreateItemAsync<User>(user, new PartitionKey(user.Name));
    }
    catch (Exception)
    {

        throw;
    }
    return Page();
}
```

Starten på denna metod fungerar som den tidigare, vi skapar upp alla nödvändiga instanser.

Sedan skapar vi upp en användare som får ett unikt ID i form av Guid tilldelat och ett användarnamn
som hämtas med hjälp av razor pages via input och **CreateItemAsync** metoden. 

Vi kör sedan en try på skapandet av vårt item, går det igenom så skickas användaren till databasen 
och sidan refreshas. Misslyckas det så kastas ett exception.

Här har vi vårt objekt:

```csharp
public class User
{
    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }
    [JsonProperty(PropertyName = "Name")]
    public string Name { get; set; }

}
```

Notera att vi har satt JsonProperties på dem, detta är för att objektet
skall kunna serial- och deserializas från och till Json-format. 

Nedanför är koden till den frontend vi använt, inget grafiskt slående. Men det fungerar!

```html
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<div class="text-center">
    <h1 class="display-4">Welcome</h1>

    <a asp-page-handler="Users" class="btn btn-info" role="button">Get All Users</a>

    <ul class="list-group">
        <br />
        @if (Model.Users != null)
        {
        @foreach (var item in Model.Users)
            {
        <li class="list-group-item">@item.Name</li>

            }
        }

    </ul>
</div>
<br />
<div class="text-center">
    <form method="post">
        <input type="text" asp-for="NewUserName" placeholder="Enter Name" />
        <input type="submit" class="btn btn-info" />
    </form>

</div>
```

Vi använder en asp-page-handler för att komma åt metoden som hämtar informationen från databasen. 
Sedan iterar vi igenom dem och lägger i en lista.

För att skriva till databasen har vi skapat en form som tar text-input, detta
skickas sedan till metoden som ansvarar för att lägga till i databasen.


---

# Vår Pipeline:

```csharp
name: Build and deploy ASP.Net Core app to Azure Web App - WebAppAzureClass20210922104053

on:
  push:
    branches:
      - master
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '5.0.x'
        include-prerelease: true

    - name: Build with dotnet
      run: dotnet build --configuration Release

    - name: dotnet publish
      run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp

    - name: Upload artifact for deployment job
      uses: actions/upload-artifact@v2
      with:
        name: .net-app
        path: ${{env.DOTNET_ROOT}}/myapp

  deploy:
    runs-on: windows-latest
    needs: build
    environment:
      name: 'production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
    - name: Download artifact from build job
      uses: actions/download-artifact@v2
      with:
        name: .net-app

    - name: Deploy to Azure Web App
      id: deploy-to-webapp
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'WebAppAzureClass20210922104053'
        slot-name: 'production'
        publish-profile: ${{ secrets.AzureAppService_PublishProfile_b81ada95cad646b58d99dbcba6e57b68 }}
        package: .
```

Flödet i denna pipeline skapades via Azure's webbportal, de första bitarna är
relativt lika vår tidigare pipeline. Tillägget är **Deploy** stycket, jag tänker fokusera mer på det
och de delarna av det som vi inte tidigare har pratat om här på bloggen.

Vi börjar med att skriva in **deploy**, detta element används för att köra *steps* som krävs för att 
deploya applikationen.

Nästa nya element är **needs**, vi använder detta när vi behöver definera ett så kallat *dependency-relationship*
mellan våra jobs.

Vi definerar sedan vår environment till production och får tillgång till nödvändiga resurser.

Nästa steg var ett helt nytt koncept för mig, och det är något som kallas [Artifacts](https://github.com/actions/download-artifact).
Artifacts är ett sätt att publicera och använda sig av olika sorters *packages* som innehåller
diverse NuGet, maven eller npm-packages. Detta betyder att man har tillgång till de dependencies som behövs
även om de inte finns kvar i det orginella flödet.

I vårt fall kommer vi nu åt det vi behöver från **build** delen av pipelinen i **deploy**, även om dessa två 
tekniskt sätt är helt separata och fristående.

Sista steget är att deploya till Azure, detta gör vi genom [Webapps-Deploy](https://github.com/Azure/webapps-deploy),
för att den skall kunna deploya krävs två saker. Först måste man ha tillgång till [Checkout](https://github.com/actions/checkout) vilket vi får
genom artifacts, sedan krävs det en av två möjliga metoder för verifiering, antingen [Azure/Login](https://github.com/Azure/login), vilket kräver
att man enablar ett CLI-command i yml-filen, och loggar in sig den vägen.

Den metoden vi valde var via **Azure Web App Publish Profile**, denna metoden är enligt mig mycket mer stabil och inuitiv. 
Detta betyder att man använder sig av Webb-portalen för att bygga sin pipeline samt för att skaffa de nödvändiga detaljerna 
för verifiering.



## Kostnader

I min tidigare uppgift hade jag lite fel uppfattning om hur kostnaderna faktiskt såg ut, så nu har jag räknat lite mer noggrannt och
detta är vad jag har kommit fram till.

Förutsatt att servern körs på ett väldigt basic system med låg traffik kommer den måntliga kostnaden hamna på ungerfär $80USD,
detta är med den grad av utrustning som krävs för att köra applikationen. Och vid extremt hög användning så skulle prislappen kunna
hamna närmre $2,600USD. Detta är en bra anledning att aldrig lämna en sida rullandes utan att ha någon sorts inkomst från den.

----

##Referenser:

[Deployment credentials](https://github.com/projectkudu/kudu/wiki/Deployment-credentials#site-credentials-aka-publish-profile-credentials)

[Azure Artifacts](https://newsignature.com/articles/all-about-azure-artifacts/)

[Yaml pipelines in Azure](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema)





