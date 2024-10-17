---
lab:
    title: 'Lab 03: Automate Azure Load Testing using GitHub Actions '
    module: 'Module 3: Implement Azure Load Testing'
---

# Overview

In this lab you learn how to configure GitHub Actions to deploy a sample web app and start a load test using Azure Load Testing.

In this lab you will:

* Create App Service and Load Testing resources in Azure.
* Create and configure a service principal to enable GitHub Actions workflows to perform actions in your Azure account.
* Deploy a .NET 8 application to Azure App Service using a GitHub Actions workflow.
* Update a GitHub Actions workflow to invoke a URL-based load test.

**Estimated completion time: 40 minutes**

## Prerequisites

* An **Azure account** with an active subscription. If you don't already have one, you can sign up for a free trial at [https://azure.com/free](https://azure.com/free).
    * An Azure web portal supported [browser](https://learn.microsoft.com/azure/azure-portal/azure-portal-supported-browsers-devices).
    * A Microsoft account or a Microsoft Entra account with the Contributor or the Owner role in the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).
* A GitHub account. If you don't have a GitHub account that you can use for this lab, follow instructions available at [Signing up for a new GitHub account](https://github.com/join) to create one.


## Instructions

## Exercise 1: Import the sample app to your GitHub repositry

In this exercise, you will import the [Azure load test sample app](https://github.com/MicrosoftLearning/azure-load-test-sample-app) repository into your own GitHub account.

### Task 1: Import the eShopOnWeb repository

1. In your web browser navigate to GitHub [http://github.com](http://github.com) and sign in using your account.
1. Start the import process [https://github.com/new/import](https://github.com/new/import).
1. Enter the following information in the **Import your project to GitHub** page.

    | Setting | Action |
    |--|--|
    | **The URL for your source repository** | Enter `https://github.com/MicrosoftLearning/azure-load-test-sample-app` |
    | **Owner** | Select your GitHub alias |
    | **Repository name** | Name your repository |
    | **Privacy** | After selecting the **Owner** the privacy options will appear. Select **Public**. |

1. Select **Begin import** and wait for the import process to complete.
1. On the new repository page select **Settings**, then select  **Actions > General** in the left navigation pane.
1. In the **Actions permissions** section of the page select the **Allow all actions and reusable workflows** option, and then select **Save**.

## Exercise 2: Create resources in Azure

In this exercise you create the resources in Azure needed to deploy the app and run the test. 

### Task 1: Create resources using the Azure CLI

In this task you create the following Azure resources:

* Resource group
* App Service Plan
* App Service instance
* Load testing instance

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com).
1. Open the **Cloud shell** and select the **Bash** mode. **Note:** You might need to configure the persistent storage if this is the first time launching the Cloud Shell.

1. Run the following commands one at a time to create variables used in the commands in the rest of the steps. Replace `<mylocation>` with your preferred location.

    ```
    myLocation=<mylocation>
    myAppName=az2006app$RANDOM
    ```
1. Run the following command to create the resource group to contain the other resources.

    ```
    az group create -n az2006-rg -l $myLocation
    ```

1. Run the following command to register the resource provider for the **Azure App Service**.

    ```bash
    az provider register --namespace Microsoft.Web
    ```

1. Run the following command to create the App Service plan.

    ```
    az appservice plan create -g az2006-rg -n az2006webapp-plan --sku F1
    ```

1. Run the following command to create the App Service instance for the app.

    ```
    az webapp create -g az2006-rg -p az2006webapp-plan -n $myAppName --runtime "dotnet:8"
    ```

1. Run the following command to create a load test resource. If you get a prompt to install the **load** extension choose yes.

    ```
    az load create -n az2006loadtest -g az2006-rg --location $myLocation
    ```

1. Run the following commands to retrieve your subscription ID. Be sure to copy and save the output from the commands, the subscription ID value is used later in this lab.

    ```
    subId=$(az account list --query "[?isDefault].id" --output tsv)
    
    echo $subId
    ```

### Task 2: Create the service principal and configure authorization

In this task you create a service principal for the app and configure it for OpenID Connect federated authentication.

1. In the Azure portal search for **Microsoft Entra ID** and navigate to the service.

1. In the left navigation pane select **App registrations** in the **Manage** group. 

1. Select **+ New registration** in the main panel and enter `GH-Action-webapp` as the name, and then select **Register**.

    >**IMPORTANT:** Copy and save both the **Application (client) ID** and **Directory (tenant) ID** values for later in this lab.


1. In the left navigation pane select **Certificates & secrets** in the **Manage** group, and then in the main window select **Federated credentials**. 

1. Select **Add a credential** and then select **GitHub Actions deploying Azure resources** in the selection drop down.

1. Enter the following information in the **Connect your GitHub account** section.

    | Field | Action |
    |--|--|
    | Organization | Enter your user or organization name. Example: `https://github.com/<user>/<repository>`. |
    | Repository | Enter the name of the repository from earlier in the lab. |
    | Entity type | Select **Branch**. |
    | GitHub branch name | Enter **main**. |

1. In the **Credential details** section give your credential a name and then select **Add**.

### Task 3: Assign roles the service principal

In this task you assign the necessary roles to the service principal to access your resources.

1. Run the following commands to assign the "Load Test Contributor" role so the GitHub workflow can send the resource tests to run. 

    ```
    spAppId=$(az ad sp list --display-name GH-Action-webapp --query "[].{spID:appId}" --output tsv)

    loadTestId=$(az resource show -g az2006-rg -n az2006loadtest --resource-type "Microsoft.LoadTestService/loadtests" --query "id" -o tsv)

    az role assignment create --assignee $spAppId --role "Load Test Contributor"  --scope $loadTestId
    ```

1. Run the following command to assign the "contributor" role so the GitHub workflow can deploy the app to App Service. 

    ```
    rgId=$(az group show -n az2006-rg --query "id" -o tsv)
    
    az role assignment create --assignee $spAppId --role contributor --scope $rgId
    ```

### Task 6: View load test results

When you run a load test from your CI/CD pipeline, you can view the summary results directly in the CI/CD output log. If you published the test results as a pipeline artifact, you can also download a CSV file for further reporting.

![Screenshot that shows the workflow logging information.](./media/github-actions-workflow-completed.png)

