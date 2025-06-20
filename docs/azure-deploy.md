# Deploying Karmabot to Azure (Docker + Cosmos DB)

This guide walks you through deploying your Karmabot (Dockerized) and MongoDB (via Azure Cosmos DB) to Azure using the Azure CLI.

---

## Prerequisites
- Azure account ([sign up](https://azure.com/free) if needed)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) installed
- Docker installed
- Slack app credentials ready

---

## 1. Login and Create Resource Group
```sh
az login
az group create --name karmabot-rg --location eastus
```

---

## 2. Create Azure Container Registry (ACR)
```sh
az acr create --resource-group karmabot-rg --name <your-acr-name> --sku Basic
```
- `<your-acr-name>` must be globally unique, all lowercase, 5-50 chars (e.g., `karmabotacr12345`).

---

## 3. Build and Push Docker Image
```sh
az acr login --name <your-acr-name>
docker tag glitter <your-acr-name>.azurecr.io/karmabot:latest
docker push <your-acr-name>.azurecr.io/karmabot:latest
```

---

## 4. Create Azure Cosmos DB for MongoDB
```sh
az cosmosdb create --name <your-cosmosdb-name> --resource-group karmabot-rg --kind MongoDB
```
- `<your-cosmosdb-name>` must be globally unique (e.g., `karmabotmongo12345`).

Get the connection string:
```sh
az cosmosdb keys list --name <your-cosmosdb-name> --resource-group karmabot-rg --type connection-strings
```

---

## 5. Create Azure Web App for Containers
```sh
az appservice plan create --name karmabot-plan --resource-group karmabot-rg --is-linux --sku B1
az webapp create --resource-group karmabot-rg --plan karmabot-plan --name <your-webapp-name> --deployment-container-image-name <your-acr-name>.azurecr.io/karmabot:latest
```
- `<your-webapp-name>` must be globally unique (e.g., `karmabotapp12345`).

---

## 6. Configure Web App Environment Variables
```sh
az webapp config appsettings set \
  --resource-group karmabot-rg \
  --name <your-webapp-name> \
  --settings MONGODB="<cosmosdb-connection-string>" VERIFICATION_TOKEN="<your-slack-verification-token>" [other env vars]
```
- Add any other required environment variables (see your README for options).

---

## 7. (If Needed) Configure ACR Authentication for Web App
If your ACR is private, set credentials:
```sh
az acr credential show --name <your-acr-name>
az webapp config container set \
  --name <your-webapp-name> \
  --resource-group karmabot-rg \
  --docker-registry-server-url https://<your-acr-name>.azurecr.io \
  --docker-registry-server-user <acr-username> \
  --docker-registry-server-password <acr-password>
```

---

## 8. Final Steps
- Update your Slack app to point to your Azure Web App's public URL.
- Test your deployment!

---

## Notes
- Replace all placeholders (e.g., `<your-acr-name>`, `<your-webapp-name>`, etc.) with your actual values.
- For more details, see the [Azure Web App for Containers documentation](https://docs.microsoft.com/en-us/azure/app-service/containers/).
- For Cosmos DB MongoDB API, see [Azure Cosmos DB MongoDB API docs](https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb/introduction). 