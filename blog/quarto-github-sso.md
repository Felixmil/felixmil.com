---
title: "Deploying a Quarto Website Behind Corporate Authentication"
date: 2025-11-21
description: "A complete guide to hosting a Quarto/GitHub site on Azure with Entra ID authentication for secure internal use"
categories:
  - tutorial
  - quarto
  - azure
image: https://learn.microsoft.com/fr-fr/training/achievements/azure-static-web-apps.svg
---

## Why Lock Down Your Quarto Site?

Sometimes you need to share content with your team without broadcasting it to the entire internet. By combining Quarto, GitHub, and Azure Static Web Apps with Entra ID authentication, you get easy-to-maintain sites with enterprise security—colleagues access with work credentials, everyone else sees a login screen.

The real advantage is collaboration: team members can contribute via pull requests, track changes, and maintain a complete history. And it **works with private GitHub repositories**, your source code stays private while the rendered site is accessible to authenticated users.

This is particularly useful in the R community where Quarto has become the go-to tool for documentation and websites. Now you can leverage GitHub's collaborative features while keeping everything secure and internal.

## The Complete Setup Guide

### Step 1: GitHub Repository Setup

1. Create a new GitHub repository for your project
2. Initialize a Quarto website in your repository
3. Configure Quarto to render to the `_site/` folder
4. Add `_site/` to your `.gitignore` file

### Step 2: GitHub Actions Workflow

Create a workflow file at `.github/workflows/publish.yml` in your `main` branch:

```yaml
on:
  workflow_dispatch:
  push:
    branches: main

name: Quarto Publish to website branch

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      deployments: write
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2
        with:
          tinytex: true

      - name: Install R
        uses: r-lib/actions/setup-r@v2

# Uncomment these lines if R packages are required to build the site
#     - name: Install R Dependencies
#       uses: r-lib/actions/setup-renv@v2

      - name: Create clear globs file
        if: github.event_name != 'pull_request'
        run: |
          echo "*" > clear_globs.txt
          echo "!.github/**" >> clear_globs.txt

      - name: Commit to website branch
        if: github.event_name != 'pull_request'
        uses: s0/git-publish-subdir-action@develop
        env:
          REPO: self
          BRANCH: website
          FOLDER: _site
          GITHUB_TOKEN: ${{ secrets.ACTIONS_PAT }}
          CLEAR_GLOBS_FILE: clear_globs.txt
```

### Step 3: Azure Static Web Apps Configuration

Create a `staticwebapp.config.json` file in your root directory:

```json
{
  "navigationFallback": {
    "rewrite": "/index.html"
  },
  "routes": [
    {
      "route": "/*",
      "allowedRoles": [ "authenticated" ]
    }
  ],
  "responseOverrides": {
    "401": {
      "statusCode": 302,
      "redirect": "/.auth/login/aad"
    }
  },
  "auth": {
    "identityProviders": {
      "azureActiveDirectory": {
        "registration": {
          "openIdIssuer": "https://login.microsoftonline.com/YOUR-TENANT-ID/v2.0",
          "clientIdSettingName": "AZURE_CLIENT_ID",
          "clientSecretSettingName": "AZURE_CLIENT_SECRET_APP_SETTING_NAME"
        }
      }
    }
  }
}
```

**Important:** Replace `YOUR-TENANT-ID` with your organization's Azure tenant ID.

### Step 4: Configure Quarto to Include Config File

Update your `_quarto.yml` to ensure the configuration file is included in the render:

```yaml
project:
  type: website
  resources:
    - staticwebapp.config.json
```

### Step 5: Create Website Branch

Create a branch called `website` and sync it to GitHub. If you're using R, you can run:

```r
usethis::use_github_pages("website")
```

Push all branches to GitHub.

### Step 6: Create Azure Static Web App

1. Go to [Azure Static Web Apps service](https://portal.azure.com/#view/HubsExtension/BrowseResource.ReactView/resourceType/Microsoft.Web%2FStaticSites)
2. Click "Create" and fill in the following:
   - Select your Subscription and Resource Group
   - Enter a name for your website
   - Select the **"Standard"** plan (required for authentication)
   - Choose **GitHub** as the deployment source
   - Select your organization, repository, and the `website` branch
   - Don't change the build settings
3. Click through until you can select the region (choose **West Europe** or your preferred region)
4. Review and click **Create**

### Step 7: Configure Custom Domain (Optional)

Set up a custom domain name in the Azure portal if desired.

### Step 8: Modify Azure Workflow

Switch to the `website` branch in your IDE. Azure will have created a workflow file. Make these modifications:

1. Add `skip_app_build: true` to the deployment step
2. Remove the `close_pull_request_job` block

Your modified workflow should look like this:

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - website
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - website

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true
          lfs: false
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "/"
          api_location: ""
          skip_app_build: true
          output_location: ""
```

Commit and push this change. Your website should now be available online (but not yet protected by authentication).

### Step 9: Register App in Microsoft Entra

1. Go to [App Registration Menu on Microsoft Entra](https://entra.microsoft.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
2. Click "New registration" and enter:
   - **Name:** Same as your Azure Static Web App name
   - **Supported account types:** Select your organization
   - **Redirect URI:**
     - Type: **Web**
     - URL: `https://<YOUR-WEBSITE-URL>/.auth/login/aad/callback`
3. Click **Register**

### Step 10: Configure Authentication

1. In the left sidebar, click **Authentication**
2. Under "Implicit grant and hybrid flows", check both token boxes:
   - Access tokens
   - ID tokens
3. Click **Save**

### Step 11: Get Client Credentials

1. Click **Overview** in the left sidebar
2. Copy the **Application (client) ID** and save it in a notepad

### Step 12: Create Client Secret

1. Click **Certificates & secrets** in the left sidebar
2. Click **New client secret**
3. Enter description: `AZURE_CLIENT_SECRET_APP_SETTING_NAME`
4. Set expiration (2 years maximum)
5. Click **Add**
6. **Important:** Copy the secret value immediately (you won't be able to see it again) and save it in a notepad

### Step 13: Add Environment Variables to Azure

1. Go back to [Azure Static Web Apps](https://portal.azure.com/#view/HubsExtension/BrowseResource.ReactView/resourceType/Microsoft.Web%2FStaticSites)
2. Open your Static Web App
3. Click **Environment variables** in the left sidebar (under Settings)
4. Add two environment variables:
   - Name: `AZURE_CLIENT_ID`, Value: The Application (client) ID you copied
   - Name: `AZURE_CLIENT_SECRET_APP_SETTING_NAME`, Value: The client secret you created
5. Click **Save**

### Step 14: Test Your Authenticated Site

Visit your website URL. You should now be redirected to the Microsoft login page. After authenticating with your corporate credentials, you'll be able to access your Quarto site!

## The Sweet Taste of Secure Success

Congratulations! You've successfully deployed a Quarto website that's accessible only to your organization. Your content is now living in that sweet spot between "completely public" and "locked in a SharePoint dungeon that nobody can find."

Now go forth and document those internal processes, share those R analyses, and build that knowledge base. Just remember: with great authentication power comes the occasional need to explain to your CEO why they need to log in to see the company blog. 

Happy deploying! 🚀🔐
