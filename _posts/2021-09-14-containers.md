---
layout: post
title: Inlägg 14-09-2021
subtitle: Containers
gh-repo: joakimwidell
gh-badge: [star, fork, follow]
tags: [CI-Pipelines, GitHub-Actions]
comments: true
readtime: true
---

### Containrar och orkestrering

<p1>
Jag har arbetat med containrar tidigare, men i detta avsnitt  lärde jag mig lite nya sätt
att använda dem. Då vi har fått en lista av punkter på innehåll i denna uppgiften
tänker jag stegvis gå igenom dem.
</p1>



## Lokal mjukvara

Nedan finner ni de verktyg som jag har installerat lokalt på min dator för att 
möjliggöra/förenkla processerna:

    - VSCode:
        Jag valde att använda VSCode för detta då vanliga Visual studio har en tendens
        att inkludera mycket som inte riktigt behövs för uppgiften. Detta hade självklart
        inte påverkat min container, men jag föredrar lättvikten.
        
        - Extensions till VSCode:
            Jag har använt mig av ett par olika extensions till denna uppgiften, vissa är
            nödvändiga som [Omnisharp C# for Visual Studio Code](https://github.com/OmniSharp/omnisharp-vscode)
            medans andra är enbart för bekvämlighetens skull. Där räknas [Docker for Visual Studio Code](https://github.com/microsoft/vscode-docker)
            definitivt in. En kort beskrivning av vad Docker-extensionen gör är att den bidrar med mallar för
            olika containrar i form av Dockerfiles och docker-compose.yml. I detta fallet använde jag en ASP .NET Core 
            mall, och fick med de nödvändiga raderna för en grundläggande container.

            Här kan man se hur filerna ser ut direkt vid import:
               

    - Docker Desktop/WSL 2:
        Precis som VSCode är detta något som jag har arbetat med tidigare, Docker desktop
        ansvarar för att köra och bygga våra containers. Det körs på WSL som är en komplett
        Linux kernel som kan användas på Windows, [här](https://docs.docker.com/desktop/windows/wsl/) 
        kan man hitta instruktioner för att installera det.

    -