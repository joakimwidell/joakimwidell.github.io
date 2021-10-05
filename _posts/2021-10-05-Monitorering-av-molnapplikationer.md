---
layout: post
title: Inlägg 01-10-2021
subtitle: Logging, not the kind with trees
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [logging, serilog]
comments: true
readtime: true
---

## Vad gör applikationen?

Själva syftet med applikationen är detsamma som den vi byggde för [Webbapplikationer i molnet](https://joakimwidell.github.io/2021-09-23-Webbapplikationer-i-molnet/).  

Jag tänker därför inte gå igenom de delar som vi redan har täckt, utan istället enbart förklara den nya funktionaliteten.

Det nya är logging. Till synes så gör applikationen precis samma saker som innan, den tar in ett input och lagrar det i en databas, alternativt så hämtar den ut en lista av all input och visar för användaren. Det sker en hel del fler saker under huven dock, dessa saker tänker jag bryta ner nu. Vi kan börja med POST metoden:

```csharp
public async Task<IActionResult> OnPost()
        {
            var watch = new Stopwatch(); // La in denna för att mäta förloppet av själva applikationen
            watch.Start();
            TimeSpan elapsedTime; // Skapar upp en instans av TimeSpan, denna används senare för att lagra en TimeSpan på ett request
            bool validInput = true; // Variabel för att kontrollera felaktig input

            CosmosClient cosmosClient = new (ConnectionString);
            Database database = cosmosClient.GetDatabase("UserDatabase");
            Container container = database.GetContainer("UserContainer");

            var user = new User
            {
                Id = Guid.NewGuid().ToString(),
                LastName = NewUserName
            };

            foreach (char c in user.LastName) //Felhantering vid input av allting förutom Unicode characters
            {
                if (!char.IsLetter(c))
                {
                    validInput = false;
                    Log.Error("Input contains invalid character, terminating operation"); //Loggar ett error om input är i felaktigt format
                }
            }
            try
            {
                if (validInput)
                {
                    ItemResponse<User> response = await container.CreateItemAsync<User>(user, new PartitionKey(user.LastName));
                    elapsedTime = response.Diagnostics.GetClientElapsedTime(); //Lagrar tidsramen i vår TimeSpan variabel
                
                    if (elapsedTime.TotalSeconds > 5)
                        Log.Warning($"Slow request detected, {elapsedTime.TotalMilliseconds}ms passed."); // Varnar om responsetiden har blivit väldigt lång av någon anledning
                    else
                        Log.Information($"Request time: {elapsedTime}ms"); // Loggar tidsförloppet

                    

                }
            }
            catch (Exception exception)
            {
                Log.Error(exception.ToString()); // Loggar möjligt exception som förhindrade koden i tryblocket
                throw;
            }

            watch.Stop(); //Stannar tidsförloppet på applikationen
            Log.Information($"Application runtime: {watch.ElapsedMilliseconds}ms"); // Loggar tidsförloppet

            return Page();
        }
```

När vi ändå höll på med logging och lite utökad felhantering kändes det vettigt att kolla livscykeln på applikationen i samma svep, jag la därför in en stopwatch som ligger och kör under tiden all kod håller på att köras. Detta är väldigt användbart för en utvecklare då det kan bli lite svårt att avgöra om man har några bottlenecks i programmet när man sitter i själva miljön.

De sakerna som jag ville logga här var främst response-tiden och felhanteringen. Och genom att skapa upp stöd för logging av input osv så fick jag en bättre bild av vliken sorts felhantering som kan krävas av en såhär webbapplikation, eftersom tankesättet blir lite annorlunda så kan det dyka upp lite lösningar på problem man inte ens visste existerade. 

Först la jag in felhantering för user input, det förklarar sig självt. Om någon del av användarens input inte är Unicode så kommer vår *validInput* variable sättas till false, och med det så kommer ingenting att skrivas till databasen. Eftersom responsetiden är kopplad till **CreateItemAsync** så kommer den inte heller loggas i detta fallet, applikationens runtime kommer däremot loggas och mest sannolikt visa en mycket lägre siffra än om allt hade körts. Och eftersom vilkoret ligger innanför ett try-vilkor så kommer ingen exception kastas, då det inte är programmet som gör fel, utan användaren som skickar en felaktig input.

Här loggar vi ett **Error**, detta skall användas när något stör programmets förlopp och förhindrar den önskade koden från att köras. Om felhanteringen hade gällt något som potentiellt skulle kunna krascha programmet eller göra det okörbart så hade vi använt oss av **Fatal** (*Log.Fatal*) istället för som det står nu.

Inuti tryblocket så skapar vi en instans av ett *ItemResponse* med typen *User* genom att asykront köra metoden **CreateItemAsync**, detta är samma metod som tidigare fast kopplad på ett *ItemResponse*, vi gör detta för att komma åt de detaljer som sparas i samband med att requestet körs.

Vi loggar sedan en responsetid beroende på om den har tagit mer- eller mindre tid än vi anser vara godkänt. I detta fallet så har jag satt gränsen på 5 sekunder, tar vårt response längre tid så loggas en **Warning**, vilket påvisar att något oönskat har hänt i programmet men det kunde fortfarande köras. Om responsetiden hamnar under 5 sekunder så loggas **Information** vilket används när man enbart vill logga information som ger nyttig data angående requestet. Nästa steg ner är det som kallas **Debug** (*Log.Debug*) och det är menat för information som enbart är relevant för t.ex en utvecklare.

Härnäst har vi GET metoden:

```csharp
public async Task<IActionResult> OnGetUsers()
        {

            var watch = new Stopwatch(); //Samma stopwatch som tidigare
            watch.Start();

            CosmosClient cosmosClient = new CosmosClient(ConnectionString);
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
                    if(Users.Count < 16) // Kollar om databasen är "full"
                    {
                        var user = new User
                        {
                            Id = item.Id,
                            LastName = item.LastName
                        };
                        Users.Add(user);
                    }
                }
            }
            Log.Information(Users.Count.ToString() + " users found"); // Loggar samtliga användare hämtade

            if (Users.Count > 10)
                Log.Warning($"More then 10 users added to database, capacity nearing limit"); // Varning för att databasen håller på att fyllas
            if (Users.Count > 15)
                Log.Fatal($"Database is full, contact support"); // Log error när databasen är full

            if (iterator == null)
                Log.Error("No content found"); // Loggar ett error om GET körs när databasen är tom

            watch.Stop();
            Log.Information($"Executed in {watch.ElapsedMilliseconds}ms"); 
            return Page();
        }
```

I denna metoden ligger det inte riktigt något nytt jämtemot den tidigare metoden, men det finns med ett bra exempel på hur **severity level** defineras. Det finns 6 olika nivåer och de ser ut såhär:

- Verbose/Debug:
    Här presenteras information som relaterar till källkoden, något som enbart är relevant för de som har tillgång till utvecklingen.
- Information:
    Event som delar icke-kritisk information till administratören, liknande vid en lapp det står *För er kännendom* på.
- Warning:
    Händelser som påvisar ett framtida problem
- Error: 
    Uppmärksammar problem som inte behöver direkt uppmärksamhet
- Fatal: 
    Här faller allt som kräver direkt uppmärksamhet t.ex en korrupt databas eller ett syntax fel.

Nedan kan ni se ett diagram på hur applikationen fungerar samt vilket syfte loggingen kan ha:

