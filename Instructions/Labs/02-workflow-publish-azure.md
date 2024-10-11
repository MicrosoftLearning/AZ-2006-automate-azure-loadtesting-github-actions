---
lab:
    title: 'Lab 02: Use GitHub Actions for Azure to publish a web app to Azure App Service'
    module: 'Module 2: Implement GitHub Actions for Azure'
---

# Overview

In this exercise, you earn how to implement a GitHub Action workflow that deploys a web app to Azure App Service.

After you complete this lab, you will be able to:

* Implement a GitHub Action workflow for CI/CD.
* Explain the basic characteristics of GitHub Action workflows.

**Estimated completion time: 40 minutes**

## Prerequisites

* An **Azure account** with an active subscription. If you don't already have one, you can sign up for a free trial at [https://azure.com/free](https://azure.com/free).
    * An Azure web portal supported [browser](https://learn.microsoft.com/azure/azure-portal/azure-portal-supported-browsers-devices).
    * A Microsoft account or a Microsoft Entra account with the Contributor or the Owner role in the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).
* A GitHub account. If you don't have a GitHub account that you can use for this lab, follow instructions available at [Signing up for a new GitHub account](https://github.com/join) to create one.

## Instructions

## Exercise 1: Import eShopOnWeb to your GitHub Repository

In this exercise, you will import the [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) repository to your GitHub account. The repository is organized the following way:

| Folder | Contents |
| -- | -- |
| **.ado** | Azure DevOps YAML pipelines |
| **.devcontainer** | Configuration to develop using containers (either locally in VS Code or GitHub Codespaces) |
| **infra** | Bicep and ARM infrastructure as code templates used in some lab scenarios |
| **.github** | YAML GitHub workflow definitions |
| **src** | The .NET 8 website used on the lab scenarios |

### Task 1: Import the eShopOnWeb repository

1. In your web browser navigate to [GitHub](http://github.com) and sign in using your account.
1. Start the import process [https://github.com/new/import](https://github.com/new/import).
1. Enter the following information in the **Import your project to GitHub** page.

    | Setting | Action |
    |--|--|
    | **The URL for your source repository** | Enter `https://github.com/MicrosoftLearning/eShopOnWeb` |
    | **Owner** | Select your GitHub alias |
    | **Repository name** | Enter **eShopOnWeb** |
    | **Privacy** | After selecting the **Owner** the privacy options will appear. Select **Public**. |

1. Select **Begin import** and wait for the import process to complete.
1. On the repository page select **Settings**, then select  **Actions > General** in the left navigation pane.
1. In the **Actions permissions** section of the page select the **Allow all actions and reusable workflows** option, and then select **Save**.

> **NOTE:** The eShopOnWeb is a large repository and might take 5-10 minutes to finish importing.

## Exercise 2: Create Azure resources and configure GitHub 

In this exercise, you create an Azure Service Principal to authorize GitHub accessing your Azure subscription from GitHub Actions. You also review and modify the GitHub workflow that builds, tests, and deploys your website to Azure.

### Task 1: Create an Azure Service Principal and save it as GitHub secret

In this task, you will create a resource group and Azure Service Principal. The service principal is used by GitHub to deploy the desired eShopOnWeb app.

1. In your browser navigate to the [Azure portal](https://portal.azure.com).
1. Open the **Cloud shell** and select the **Bash** mode. **Note:** You need to configure the persistent storage if this is the first time you launched the Cloud Shell.
1. Create a resource group with the following `az group create` CLI command. Replace `<location>` with a region near you.

    ```
    az group create -n az2006-rg -l <location>
    ```

1. Run the following command to register the resource provider for the **Azure App Service** you will deploy later in the lab.

    ```bash
    az provider register --namespace Microsoft.Web
    ```

1. Retrieve your Azure subscription ID with the `az account show` command. This command produces JSON output, please copy and save the GUID in the `"id": <GUID>` field. This is needed to create the service principal.

1. Create a service principal with the following `az ad sp` command. Replace `<SUBSCRIPTION-ID>` with the id you copied in the previous step.

    ```
    az ad sp create-for-rbac --name GH-Action-eshoponweb --role contributor --scopes /subscriptions/<SUBSCRIPTION-ID>/resourceGroups/az2006-rg
    ```

    This command outputs a JSON object that contains the identifiers used to authenticate against Azure in the name of a Microsoft Entra identity (service principal). Copy the JSON object for use in the following steps. 

1. In a browser window navigate to your **eShopOnWeb** GitHub repository.
1. On the repository page select **Settings**, then select **Secrets and variables > Actions** in the left navigation pane.
1. Select **New repository secret** and enter the following information:
    * **NAME**: `AZURE_CREDENTIALS`
    * **Secret**: Enter the JSON object created when creating the service principal.
1. Select **Add secret**.

### Task 2: Modify and execute the GitHub workflow

In this task, you modify the provided GitHub workflow and execute it to deploy the solution to your own subscription.