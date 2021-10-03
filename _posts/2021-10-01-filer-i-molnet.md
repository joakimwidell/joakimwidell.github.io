---
layout: post
title: Inlägg 01-10-2021
subtitle: Filer i molnet
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [AzureBlobs, RazorPages]
comments: true
readtime: true
---

## Beskrivning av applikationen

Applikationen är uppbyggd i Visual Studio med hjälp av Razor pages, och den funkar på ett ganska enkelt sätt.

Händelseförloppet ser ut såhär:

![Image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/flowchart-blobs.png)

Användaren kan välja att ladda upp en fil, och sedan submitta den. Det finns även stöd för drag and drop.

![Image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/blob-site.png)

När användaren sedan har valt sin fil och tryckt på submit så skickas filen upp i Azure Blob storage.

## DataFlow

Själva webbapplikationen har endast stöd för att ladda upp bilder i nuläget. Det ligger stöd för utökad funktionalitet inuti projektet dock. 

Stommen för hur genererad input kan röra sig genom programmet syns i följande diagram:

![Image](https://github.com/joakimwidell/joakimwidell.github.io/blob/main/_posts/Images/dataflow-blobl.png)

Beroende på vad användaren vill åstadkomma så tas ett input, antingen i form utav en fil eller ett request. Detta input hanteras sedan av applikationen och triggar en av följande metoder:

**UploadAsync();**

**CopyToAsync();**

**GetBlobsAsync();**

**DeleteAsync();**

**CreateBlobContainerAsync();**

Dessa är inbyggda metoder inom **Azure.Storage.Blobs** och **Azure.Storage.Models**, och kan användas fritt så länge det finns en korrekt koppling till ett blob storage account på Azure.

I detta programmet så har jag använt mig av ett repository pattern för att kunna använda mig av metoderna vid behov och för att ha relevanta kopplingar och hantering av användarinput lagrat direkt i metoden.

För att börja i rätt ände så skall jag visa hur metoden för att hantera input ser ut, och sedan kan vi gå in mer på *logik* metoderna:

```csharp
     public class UploadImage : PageModel
    {
        private static string connectionString = "CONNECT_STR"; //Variabel lokalt definerad i powershell via $env, annan lösning är t.ex local.settings.json
        private readonly Logic _logic = new(); //ny instans av repository pattern

        private readonly IWebHostEnvironment _environment; //Ger information om vilken webbhost miljö som används

        public UploadImage(IWebHostEnvironment environment)
        {
            _environment = environment;
        }

        [BindProperty]
        public IFormFile Upload { get; set; } //Formatet på user-input
        public async Task<IActionResult> OnPostAsync() //Hanterar ett POST request asynkront
        {
            if (Upload == null)
                return BadRequest("No input data found"); //Hanterar tom inmatning

            BlobServiceClient blobServiceClient = new(connectionString); //

            var localPath = $"{Guid.NewGuid()}{Upload.FileName}"; //Unik filepath

            var file = Path.Combine(_environment.ContentRootPath, "uploads", localPath); //den tillfälliga sökvägen till filen under uppladdningens gång
            using (var fileStream = new FileStream(file, FileMode.Create)) //Säger till OS att det skall skapa en ny fil
            {
                await Upload.CopyToAsync(fileStream); //Kopierar filens innhåll asynkront till vårt target stream
            }

            await _logic.IUploadBlobContentAsync("testb1547404-b86b-4ec1-bfb9-675d17e882c6", blobServiceClient, "uploads", localPath); //Kallar på en custom-metod för uppladdningen, det är möjligt att ta de hårdkodade variablerna som inparametrar via razor pages, men funktionellt var målet

            System.IO.File.Delete(localPath); //tar bort den tillfälliga filen
            return Page(); //Page refresh
        }
    }
```

Vi lagrar input i en ny instans av IFormFile, sparar sedan ner filens innehåll lokalt och skickar sedan kopian till storage innan den tas bort från den lokala förvaringen. Slutligen uppdaterar vi vår html och visar den uppdaterade versionen av sidan. 

### Logik

Merparten av logiken används inte i webbapplikationen, men det är vettigt att veta hur man kan använda dessa delar. Så nu skall vi gå igenom hur man hade kunnat implementera dessa olika metoder på en webbApp.


```csharp
    public async Task ICreateContainerAsync(BlobServiceClient blobServiceClient)
        {
            string containerName = $"test{Guid.NewGuid().ToString()}"; //Guid för att förhindra dubletter
            BlobContainerClient containerClient = await blobServiceClient.CreateBlobContainerAsync(containerName); //Skapar upp en ny container med containerName
        }
```

Det är ingen komplicerad affär att skapa upp en ny container, det viktiga är att man har en koppling till databasen och ett unikt namn på sin nya container. Här tar vi endast in en *BlobServiceClient* och får då tillgång till vårt storage account, med detta kan vi sedan definera ett unikt namn.  

Här kan man koppla på en *form* som tar in en sträng från användaren och därav skapar ett personligt nytt namn, då är inte heller Guid helt nödvändigt då det likväl hade gått att köra **GetBlobs** och sedan undersökt om det valda namnet redan finns innan en ny blob skapas.  

Här ser ni metoden jag byggde för att ladda upp en fil till en blob container:

```csharp
public async Task IUploadBlobContentAsync(string containerName, BlobServiceClient blobServiceClient, string localPath, string fileName)
        {
            string localFilePath = Path.Combine(localPath, fileName); //Skapar sökvägen till den uppladdade filen

            BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient(containerName);

            BlobClient blobClient = containerClient.GetBlobClient(fileName); //Skapar ett nytt BlobClient objekt där vår fil kommer lagras

            using FileStream uploadFileStream = File.OpenRead(localFilePath); //Förser filen med en Stream som stöder både async och sync ops.

            await blobClient.UploadAsync(uploadFileStream); //Laddar upp vår filestream till blobClient och lagrar datan.

            Console.WriteLine("Uploaded!");

            uploadFileStream.Close(); //Stänger ner Streamen och friar upp resurser
        }
```  
Här tar vi in parametrar för att först kunna skapa en koppling till vår blob. Namnet på den önskade containern och en instans av *BlobServiceClient* vilket ger oss funktionalitet för att manipulera vår blob storage. Vi tar även in ett lokalt directory och det önskade filnamnet som i denna applikationen är en kombination av filens man samt en Guid. Sedan lagras filen tillfälligt för att hanteras av metoden.  

Jag vet att det finns en bättre lösning här där man kan skicka filen direkt upp till storage men siktade in mig på funktionellt/fult och sedan tog tiden slut.  

Härnäst har vi en metod för att lista samtliga blobs i vår container:

```csharp
public async Task IListBlobsInContainerAsync(BlobServiceClient blobServiceClient, string containerName)
        {
            BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient(containerName);

            if(containerClient == null)
                Console.WriteLine("No blobs available, check your container name")
            else
            {
                await foreach (var item in containerClient.GetBlobsAsync()) //itererar genom alla tillgängliga blobs och skriver ut dem till konsollen
                    Console.WriteLine(item.Name);
            }
        }
```

Det hade räckt för en användare att ha tillgång till ett containernamn här och mata in det manuellt i programmet för att kunna lista alla blobs som finns tillgängliga.

Sen har vi metoden för att ladda ner innehåll från en specifik blob:

```csharp
public async Task IDownloadBlobAsync(
    string localPath, string fileName, string containerName, BlobServiceClient blobServiceClient)
    {

        string localFilePath = Path.Combine(localPath, fileName);

        BlobContainerClient blobContainerClient = blobServiceClient.GetBlobContainerClient(containerName);

        BlobClient blobClient = blobContainerClient.GetBlobClient(fileName);

        BlobDownloadInfo download = await blobClient.DownloadAsync(); //Laddar ner innehållet i en blob

        int index = localFilePath.LastIndexOf(".");

            string fileType = "";

            if(index > 0)
            {
                localFilePath = localFilePath.Substring(0, index);
                fileType = fileType.Substring(index + 1);
            }

        string downloadFilePath = $"{localFilePath}{Guid.NewGuid()}.{fileType}"; //Sökväg för det nedladdade innehållet

        using FileStream downloadFileStream = File.OpenWrite(downloadFilePath); //Ersätter innehåll eller skapar en ny fil med det önskade innehållet
        await download.Content.CopyToAsync(downloadFileStream); //Läser innehållet i FileStream och kopierar ner dem

        downloadFileStream.Close();
    }
```

Förutsatt att användaren har det unika blob-namnet kan allt detta skötas med input via en webbapp.

Sist har vi metoden för att ta bort blobs ur containern. 

```csharp
public async Task IDeleteBlobAsync(string containerName, BlobServiceClient blobServiceClient)
    {
            
        Console.WriteLine("Type 'y' to delete or 'n' to exit:\n");
        bool choice = Console.ReadLine() == "y" ? true : false; //Sätter bool beroende på användarens val

        if (choice)
        {
            BlobContainerClient blobContainerClient = blobServiceClient.GetBlobContainerClient(containerName);
            if (blobContainerClient == null)
                Console.WriteLine("No such container");
            else
            {
                await blobContainerClient.DeleteAsync(); //Markerar specifik container och tar bort den

                Console.WriteLine("Container deleted! " + containerName);
            }
        }
    }
```

----

## Kostnad

Priset på de begärda specifikationerna, dvs en Premium performance blob storage med ungefär 1000 filer med 3 nedladdningar per dag hade hamnat på ungefär 100$ enligt Azure Price Calculator.

## Säkerhet

Azure ger användare möjligheten att använda *secret keys* som antingen är genererade av Microsoft eller på egen hand. 

Även om detta ger en nivå av säkerhet för uppkopplingen finns det en hel rad användarspecifika saker man måste tänka på när man handskas med molnet, *vart är dina filer lagrade?*, *Skickas din data mellan olika punkter?*, *Är din data offentlig?* osv.

Därför är det väldigt viktigt att man som utvecklare inte bara antar att säkerheten kommer inbyggd, utan även kontrollerar alla möjliga säkerhetsrisker.

## Referenser

Azure blob storage - https://medium.com/@rammonzito/azure-blob-storage-using-a-net-core-console-application-106a0c2e6de5
Azure storage CRUD operations - https://www.c-sharpcorner.com/article/azure-storage-crud-operations-in-mvc-using-c-sharp-part-two/
Azure storage blobs - https://docs.microsoft.com/en-us/dotnet/api/azure.storage.blobs?view=azure-dotnet









