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
# docker run -d -p 5001:8080 --name asp-rest-demo asp-rest-demo
# docker push ghcr.io/<your_github_user>/asp-rest-demo:latest

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
