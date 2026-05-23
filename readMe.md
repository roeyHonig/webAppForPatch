# ASP.NET Core MVC Development and Deployment Guide

## Overview

This project includes a **Dev Container** configuration for **GitHub Codespaces** and a **deployment workflow** for Firebase Hosting.  
The guide explains how the development environment is set up, how launch settings are configured, and how automated deployment works.


# Dev Container (`.devcontainer/devcontainer.json`)

The Dev Container ensures a **consistent development environment**:

- Base Docker image with the correct .NET SDK.  
- Tools and extensions required for development, e.g., Git and C# support.  
- Port forwarding for the app (HTTP).  
- Post-create commands like `dotnet restore` for ready-to-code setup.

**Purpose:** Developers can clone the repository and start coding immediately, without worrying about environment differences.

# Launch Settings (`Properties/launchSettings.json`)

For development in Codespaces:

- The **HTTPS endpoint is removed** to avoid certificate issues.  
- Only HTTP (`http://localhost:5000`) is used.  

**Reason:** HTTPS certificates do not persist in containerized environments like Codespaces, which can prevent the app from running smoothly.

**Important Notes:**

- Runs in HTTP only inside Codespaces.  
- Before deploying to production, the HTTPS configuration must be restored.

**Example dev launchSettings.json (HTTP-only):**

```json
{
  "profiles": {
    "MyMvcApp": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

# Production Launch Settings (`Properties/launchSettings.Production.json`)

For production deployment:

- HTTPS is enabled (`https://localhost:5001`)  
- HTTP remains available (`http://localhost:5000`)  
- Environment set to `Production`

**Example production launchSettings.json:**

```json
{
  "profiles": {
    "MyMvcApp": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:5001;http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Production"
      }
    }
  }
}
```

Note: This file is only used during deployment. Dev environments continue using the HTTP-only version.


# CI/CD Deployment Workflow (GitHub Actions)

The deployment process is automated via **GitHub Actions**, triggered on push or merge to `master`.  

**Workflow steps:**

1. Checkout the repository  
2. Set up .NET SDK  
3. Restore dependencies  
4. Replace dev launchSettings with production launchSettings  
5. Build and publish the project  
6. Install Firebase CLI  
7. Deploy the published app to Firebase Hosting

**Workflow YAML (`.github/workflows/deploy.yml`):**

```yaml
name: Deploy ASP.NET MVC to Firebase

on:
  push:
    branches:
      - master

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '7.0.x'

      - name: Restore packages
        run: dotnet restore

      - name: Use production launch settings
        run: cp Properties/launchSettings.Production.json Properties/launchSettings.json

      - name: Build and publish
        run: dotnet publish -c Release -o ./publish

      - name: Install Firebase CLI
        run: npm install -g firebase-tools

      - name: Deploy to Firebase Hosting
        env:
          FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
        run: firebase deploy --only hosting --project your-firebase-project-id


```

Notes:

Replace your-firebase-project-id with your actual Firebase project ID.
Add FIREBASE_TOKEN as a GitHub secret (Settings → Secrets → Actions) using firebase login:ci.




# Summary

- **DevContainer**: Consistent environment for Codespaces, HTTP-only for simplicity  
- **HTTPS removed** for development to avoid container certificate issues  
- **Production launchSettings** includes HTTPS and environment set to `Production`  
- **GitHub Actions** automates deployment to Firebase Hosting on push/merge to master  
- Clean separation between dev and production environments ensures reliable development and safe deployment
