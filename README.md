# Minimal-Api-with-MongoDB-Dokumentation
### Aufgabe 7.4-1

.NET 7 installieren.



Dockerfile:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env
WORKDIR /build
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:6.0
LABEL description="Minimal Api with MongoDB"
LABEL organisation="KSB St. Gallen"
LABEL author="Manuel Bruelisauer"
WORKDIR /app
COPY --from=build-env /build/out .
ENTRYPOINT ["dotnet", "app.dll"]
```

```bash
docker run -p 5001:80 myminimalapi
```

Dieser Command war wichtig, da der Port auf dem Localhost sonst nicht richtig funktioniert hat.



Docker compose file:

```bash
version: "3.9"
services:
  myminimalapi:
    build: WebApi
    ports:
      - 5001:80
```



### Aufgabe 7.4-2

Um Applikation auszuführen

```bash
dotnet run
```

```bash
code .
```

Mongodb Container:

```bash
docker run -d --name my-mongodb -p 27017:27017 -v mydata:/data/db mongo
```

```bash
dotnet add package MongoDB.Driver
```

Connection-String hardcode:

```c#
using MongoDB.Driver;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Minimal API Version 1.0");

// app.MapGet("/check", () => { /* Code zur Prüfung der DB ...*/ return "Zugriff auf MongDB ok.";});

app.MapGet("/check", () =>
{
    try
    {
        var mongoDbConnectionString = "mongodb://localhost:27017";
        var mongoClient = new MongoClient(mongoDbConnectionString);
        var databaseNames = mongoClient.ListDatabaseNames().ToList();
        
        return "Zugriff auf MongoDB ok. Vorhandene DBs: " + string.Join(",", databaseNames);
    }
    catch (System.Exception e)
    {
        return "Zugriff auf MongoDB funktioniert nicht: " + e.Message;
    }
});


app.Run();
```

Neues cs (kein hardgecodeter Connection-String mehr):

```c#
using MongoDB.Driver;

var builder = WebApplication.CreateBuilder(args);

var movieDatabaseConfigSection = builder.Configuration.GetSection("DatabaseSettings");
builder.Services.Configure<DatabaseSettings>(movieDatabaseConfigSection);

var app = builder.Build();

app.MapGet("/", () => "Minimal API Version 1.0");

// app.MapGet("/check", () => { /* Code zur Prüfung der DB ...*/ return "Zugriff auf MongDB ok.";});

app.MapGet("/check", (Microsoft.Extensions.Options.IOptions<DatabaseSettings> options) => 
{
    try
    {
        var mongoDbConnectionString = options.Value.ConnectionString;
        var mongoClient = new MongoClient(mongoDbConnectionString);
        var databaseNames = mongoClient.ListDatabaseNames().ToList();
        
        return "Zugriff auf MongoDB ok. Vorhandene DBs: " + string.Join(",", databaseNames);
    }
    catch (System.Exception e)
    {
        return "Zugriff auf MongoDB funktioniert nicht: " + e.Message;
    }
});


app.Run();
```

appsettings.json:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "DatabaseSettings": { "ConnectionString": "mongodb://localhost:27017"}
}
```

neues Docker-compose file:

```bash
version: "3.9"
services:
  mongodb:
    image: mongo

  myminimalapi:
    build: WebApi
    ports:
      - 5001:80
    depends_on:
      - mongodb
    environment:
      - MoviesDatabaseSettings__ConnectionString=mongodb://gbs:geheim@mongodb:27017
```
