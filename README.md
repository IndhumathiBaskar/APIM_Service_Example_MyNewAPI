# HelloAzure .NET API & Docker Container/ Azure AD / APIM Learning Repository

This repository is a step-by-step learning project for creating, containerizing, deploying, and securing a .NET API using Azure services.

---

## **Project Overview**

* Simple .NET API (`HelloAzure`) with `/weatherforecast` endpoint.
* Returns random weather data.
* Built using **.NET 10**.
* Explored **Docker**, **Azure Container Registry (ACR)**, **Azure Container Apps**, and **Azure API Management (APIM)** with **Azure AD (Microsoft Entra ID)** authentication.

---

## **Prerequisites**

* GitHub account
* Azure free account
* GitHub Codespaces or local machine with Docker installed
* Azure CLI (`az`)

---

## **Step 1: Create GitHub Repository**

1. Create a new repository in GitHub: e.g., `HelloAzure-DotNet-Learning`.
2. Open the repository in **Codespaces**.
3. Initialize a new .NET Web API project:

```bash
dotnet new webapi -n MyNewAPI
cd MyNewAPI
```

4. Initialize git (if not already) and push to GitHub:

```bash
git add .
git commit -m "Initial commit - MyNewAPI"
git push origin main
```

---

## **Step 2: Run and Test API Locally in Codespaces**

1. Run the API:

```bash
dotnet run
```

2. Open the forwarded URL in Codespaces browser, e.g.:

```
https://special-spoon-xxxx-8080.app.github.dev/weatherforecast
```

* You should see JSON output with random weather forecast data.

---

## **Step 3: Create Dockerfile (Multi-stage Build)**

```dockerfile
# Stage 1: Build
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore HelloAzure.csproj
RUN dotnet publish HelloAzure.csproj -c Release -o /app

# Stage 2: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY --from=build /app .
EXPOSE 8080
ENTRYPOINT ["dotnet","HelloAzure.dll"]
```

**Explanation:**

* **Stage 1 (Build)**: Restores NuGet packages and builds the project.
* **Stage 2 (Runtime)**: Copies compiled output into a smaller runtime image.
* **EXPOSE 8080**: Container listens on port 8080.
* **ENTRYPOINT**: Runs `HelloAzure.dll`.

---

## **Step 4: Build & Test Docker Image**

1. Build image:

```bash
docker build -t helloazure:latest .
```

2. Run image locally (Codespaces):

```bash
docker run -p 8080:8080 helloazure:latest
```

3. Open the forwarded URL to test:

```
https://special-spoon-xxxx-8080.app.github.dev/weatherforecast
```

---

## **Step 5: Push Docker Image to Azure Container Registry (ACR)**

1. Login to Azure CLI:

```bash
az login
```

2. Login to ACR:

```bash
az acr login --name azuredotnetapi
```

3. Tag and push the Docker image:

```bash
docker tag helloazure:latest azuredotnetapi.azurecr.io/helloazure:latest
docker push azuredotnetapi.azurecr.io/helloazure:latest
```

4. Pull the image (for testing or redeployment):

```bash
docker pull azuredotnetapi.azurecr.io/helloazure:latest
```

**Key Points:**

* Docker image is self-contained (app + runtime + dependencies).
* ACR stores Docker images permanently.
* Codespaces is temporary; images there disappear if closed.

---

## **Step 6: Deploy Docker Image as Azure Container App**

1. In Azure portal, create **Container App**:

* Name: `helloazure-app`
* Image source: Azure Container Registry
* Registry: `azuredotnetapi.azurecr.io`
* Image: `helloazure`
* Tag: `latest`
* Managed Identity: System-assigned
* CPU/Memory: 0.5 CPU, 1 GiB memory (consumption plan)

2. **Enable Ingress** for public access:

* Ingress type: HTTP
* Target port: 8080

3. Deployment successful. Public URL will look like:

```
https://helloazure-app.<region>.azurecontainerapps.io/weatherforecast
```

---

## **Step 7: Secure API Using Azure AD & APIM**

1. Register an application in **Azure AD** (Microsoft Entra ID):

* Record: **Tenant ID**, **Client ID**, **Client Secret**, **Domain**.

2. Update `appsettings.json`:

```json
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "Domain": "yourtenant.onmicrosoft.com",
  "TenantId": "YOUR_TENANT_ID",
  "ClientId": "YOUR_APPLICATION_ID",
  "ClientSecret": "YOUR_CLIENT_SECRET",
  "Audience": "api://YOUR_APPLICATION_ID"
}
```

3. Update `Program.cs`:

```csharp
builder.Services.AddAuthentication("Bearer")
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization();
builder.Services.AddControllers();
```

4. Deploy API to APIM:

* Create an API in **APIM** using your deployed Container App URL.
* Select **OAuth 2.0** validation with Azure AD.
* Make API **public** or attach **OAuth server** for token validation.

5. Test API through APIM:

* Generate an Azure AD token using client credentials:

```bash
curl -X POST https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/token \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&scope=api://YOUR_CLIENT_ID/.default"
```

* Use `Authorization: Bearer <token>` to call your API via APIM.

---

## **Step 8: Key Learnings**

* **GitHub**: Stores source code.
* **Docker image**: Self-contained package (app + runtime).
* **ACR**: Stores Docker images; independent of Codespaces.
* **Container App**: Hosts your Dockerized API in Azure.
* **APIM + Azure AD**: Secure API using OAuth 2.0 and validate tokens.
* **Codespaces**: Temporary development environment.

---

## **Step 9: Clean-up**

* Delete **Container App** or **ACR** if you want to save cost.
* Dockerfile + source code in GitHub is enough to rebuild.

---


# MyNewAPI - .NET Web API & Azure AD / APIM Learning

This repository contains a **simple .NET Web API** project (`MyNewAPI`) for learning how to expose APIs in Azure and secure them using **Azure API Management (APIM)** with **Azure AD (Microsoft Entra ID)** authentication.

---

## **Project Overview**

* Simple .NET API with `/weatherforecast` endpoint.
* Returns random weather data.
* Built using **.NET 10 (or your version)**.
* Explored **Azure API Management** with OAuth2.0 / Azure AD authentication.

---

## **Prerequisites**

* GitHub account
* Azure free account
* Codespaces or local machine with .NET SDK installed
* Azure CLI (`az`)

---

## **Step 1: Create GitHub Repository & .NET API**

1. Create a new repository in GitHub: e.g., `APIM_Service_Example_MyNewAPI`.
2. Open the repository in **Codespaces**.
3. Create a new .NET Web API project:

```bash
dotnet new webapi -n MyNewAPI
cd MyNewAPI
```

4. Initialize git (if not already) and push to GitHub:

```bash
git add .
git commit -m "Initial commit - MyNewAPI"
git push origin main
```

---

## **Step 2: Run and Test API Locally**

1. Run the API:

```bash
dotnet run
```

2. Open the forwarded Codespaces URL:

```
https://special-spoon-xxxx-8080.app.github.dev/weatherforecast
```

* You should see a JSON array with weather forecast data.

---

## **Step 3: Configure Azure AD (Microsoft Entra ID)**

1. Register a new application in **Azure AD**:

* Note **Tenant ID**, **Client ID**, **Client Secret**, and **Domain**.

2. Update `appsettings.json`:

```json
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "Domain": "yourtenant.onmicrosoft.com",
  "TenantId": "YOUR_TENANT_ID",
  "ClientId": "YOUR_APPLICATION_ID",
  "ClientSecret": "YOUR_CLIENT_SECRET",
  "Audience": "api://YOUR_APPLICATION_ID"
}
```

3. Update `Program.cs` to enable Azure AD authentication:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication("Bearer")
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization();
builder.Services.AddControllers();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.MapGet("/weatherforecast", () =>
{
    var summaries = new[] { "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching" };
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast(
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        )).ToArray();
    return forecast;
});

app.Run();

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```

---

## **Step 4: Deploy API to Azure**

1. Deploy API to **Azure App Service** (Web App):

* Select runtime: `.NET 10` or your version.
* Publish code from GitHub repository.
* Enable **HTTPS**.

2. Test the deployed API using the public URL:

```
https://<your-app-name>.azurewebsites.net/weatherforecast
```

---

## **Step 5: Configure API Management (APIM)**

1. Create an **API Management instance** in Azure.
2. Add a new API using the **App Service endpoint** as backend.
3. Configure **OAuth 2.0** validation:

* Use **Azure AD** as the OAuth server.
* Register your API in APIM and set the **scope** to the Azure AD Application.
* Enable public access if needed for testing.

4. Test API via APIM:

* Generate a **Bearer token** from Azure AD using client credentials:

```bash
curl -X POST https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/token \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&scope=api://YOUR_CLIENT_ID/.default"
```

* Call the API with the token:

```bash
curl -H "Authorization: Bearer <ACCESS_TOKEN>" https://<apim-instance-name>.azure-api.net/weatherforecast
```

---

## **Step 6: Key Learnings**

* **GitHub**: Stores source code.
* **Azure App Service**: Hosts the .NET API without Docker.
* **APIM**: Centralized API gateway for securing and monitoring APIs.
* **Azure AD**: Provides OAuth 2.0 authentication and token validation.
* **Public API Testing**: By enabling ingress and APIM, we can call the API from anywhere.

---

## **Step 7: Clean-up / Learning Tip**

* Delete **APIM instance** or **App Service** if you want to save cost.
* The **GitHub repository** contains all source code and configurations for redeployment later.

---

This README captures the **entire process of building a .NET API, deploying it on Azure, and securing it via APIM + Azure AD**, **without using Docker**.


