# Github Actions

## Token + docker login

### Generieren des access tokens

1. Generiere in deinen Settings (https://github.com/settings/apps) einen personal Access Token (classic)
2. Bei *New personal access token (classic)* wähle *write:packages*
3. *Make sure to copy your personal access token now. You won’t be able to see it again!*

### Konfigurieren von docker login

Starte docker desktop. Führe in der git bash folgenden Befehl aus
```
echo <your_token> | docker login ghcr.io -u <your_github_user> --password-stdin
```

### Erstellen des dockerfiles

*asp-rest-demo* muss an deinen Projektnamen angepasst werden.

```dockerfile
# docker build --tag ghcr.io/<your_github_user>/asp-rest-demo:latest . 
# docker push ghcr.io/<your_github_user>/asp-rest-demo:latest
# docker run -d -p 5001:8080 --name asp-rest-demo ghcr.io/<your_github_user>/asp-rest-demo:latest

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
# copy everything else and build app
COPY . ./source/asp-rest-demo/
WORKDIR /source/asp-rest-demo
RUN dotnet restore
RUN dotnet publish -c release -o /app --no-restore

# final stage/image
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app ./
EXPOSE 8080
ENTRYPOINT ["dotnet", "asp-rest-demo.dll"]
```

Führe dann *docker build --tag ghcr.io/<your_github_user>/asp-rest-demo:latest . * aus.
*<your_github_user>* muss natürlich angepasst werden.

### Push zu ghcr

Führe den folgenden Befehl in der Shell aus. Tausche *<your_github_user>*!

```
docker push ghcr.io/<your_github_user>/asp-rest-demo:latest
```

#### Package auf public setzen

1. Gehe in Github auf dein Profil und wähle *Packages*.
2. Klicke auf das Package und wähle *Package settings*.
3. Wähle bei *Change package visibility* ganz unten *public*.

#### Container lokal starten

1. Lösche in docker desktop das lokale Image
2. Führe den folgenden Befehl aus, um den Server auf Port 5001 zu starten:
   
```
docker run -d -p 5001:8080 --name asp-rest-demo ghcr.io/<your_github_user>/asp-rest-demo:latest
```

Die Applikation muss nun auf *http://localhost:5001* laufen.
Kontrolliere in docker desktop im Log die Ausgaben (Hosting environment, etc.).

## Erstellen des github workflows

### Anlegen der secrets

Gehe in github auf das Repository und wähle *Settings* - *Secrets and variables* - *Actions*.
Lege unter *Repository secrets* 2 Secrets an:
- *GHCR_USERNAME:* Username auf github.
- *GHCR_TOKEN:* Token


Erstelle im Root Ordner des Repos einen Ordner *.github* und in diesem Ordner den Ordner *workflows*.
Erstelle eine Datei *deploy.yaml* und kopiere folgenden Code:

```yaml
name: Deploy ASP.NET Core App

on:
    push:
      branches: [ "main" ]

jobs:
    deploy:
        runs-on: ubuntu-latest
        steps:        
        - name: Login to GitHub Container Registry
          uses: docker/login-action@v3
          with:
            registry: ghcr.io
            username: ${{ secrets.GHCR_USERNAME }}
            password: ${{ secrets.GHCR_TOKEN }}

        - name: Checkout Repository
          uses: actions/checkout@v4
    
        - name: Build the Docker image
          working-directory: src
          run: docker build . -t ghcr.io/${{ secrets.GHCR_USERNAME }}/asp-rest-demo:latest

        - name: Push the images to github packages
          run: docker push ghcr.io/${{ secrets.GHCR_USERNAME }}/asp-rest-demo:latest
```

Führe nun einen commit (z. B. *add workflow*) durch.
