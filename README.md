# LastPass Solution for Azure Sentinel
This repository contains all resources for the LastPass Azure Sentinel Solution.
The LastPass Solution is built in order to easily integrate LastPass with Azure Sentinel.

By deploying this solution, you'll be able to monitor activity within LastPass and be alerted when potential security events arise.
The solution consists out of the following resources:
- A data connector using an Azure App Service to go out to the LastPass API.
- One workbook to visualize some of the activity within LastPass
- Hunting Queries to look into potential security events
- Analytic Rules to generate alerts and incidents when potential malicious events happen

## Data Connector Deployment
The data connector will retrieve the LastPass Activity data through the LastPass Enterprise API.

Authentication is done through a LastPass Provisioning Hash API key which can be generated by a LastPass administrator by following the steps in the following [How To Article](https://support.logmeininc.com/lastpass/help/use-the-lastpass-provisioning-api-lp010068).

An ARM template is provided to easily deploy all of the components for the data connector, this includes:
- The Key Vault used to store the credentials for both the Log Analytics workspace and the LastPass API key
- An App Configuration which contains the configuration variables.
- An App Services which contains one C# Azure Functions which pulls data from the LastPass API.

### Deploy to your Azure subscription

1. Click on the "Deploy to Azure" button below. When prompted, log in to your Azure subscription. 

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthecollectiveconsulting%2FLastPassAzureSentinel%2Fmain%2FData%2520Connectors%2FDeployLastPassConnector.json)

2. Select a subscription and resource group, we recommend creating a new resource group.
3. Fill in the required fields:

  1. **Serviceplan Name**: the name of the serviceplan.
  2. **Key Vault Name**: the name of the keyvault, where we will store the secrets.
  3. **Storage Name**: The name of the bot.
  4. **App Config Name**: The name of the app-configuration, where we will store the connector settings.
  5. **Connector Name**: The name of the dataconnector.
  6. **Shared Key**: The sharedKey from the LastPass API.
  7. **Prov Hash**: The Provisioning Hash from the LastPass API.
  8. **Cid**: The Provisioning Hash from the LastPass API.
  9. **Wk Space Id**: The Log analytics workspace ID.

4. when the deployment is done, we need to do some manual configurations.

    1. **Add Access Policies**:

        1. Go to the Created KeyVault.
        2. Select **Access policies**, which can be found under the Settings blade.
        3. Click **+ Add Access Policy**
        4. Set Secret permissions:

        1. Get
        2. List
        
        5. Select principal: select the dataconnector.
        6. report steps 3-5 and select the app configuration as principal.
        7. Click Save

    2. **Add App config access**

        1. Go to the created app configuration
        2. Select **Access keys**, which can be found under the Settings blade.
        3. Click the **copy** button from the **connection string** value under the Primary key section.
        4. Navigate to the dataconnector that was created.
        5. Select **Configuration**, which can be found under the Settings blade.
        6. Click on the Application Setting **AppConfigurationUri**
        7. Paste the Access key inside the value input field.
        8. Press OK followed by Save

5. Restart the dataconnector

> If All succeeded you should see a custom table **LastPass_Data_CL** inside of the Log Analytics Workspace.
> Note: It can take a couple of minutes before the data is visible.

## Workbook
The workbook contains visualizations about the activity within LastPass and provides an overview of the user activity.
This allows you to identify user with a high amount of activity.

Besides user activity, the sign-ins logs are correlated to point out sign-ins which were done from previously unknown IPs and admin activity is surfaced.

The workbook can be deployed by creating an empty workbook and adding the data from the Gallery template.

## Hunting
- Login into LastPass from a previously unknown IP.
- Failed sign-ins into LastPass due to MFA.
- Password moved to shared folders

## Analytic Rules
The solution currently includes five analytic rules:
- TI map IP entity to LastPass data
- Highly Sensitive Password Accessed
- Failed sign-ins into LastPass due to MFA
- Employee account deleted
- Unusual Volume of Password Updated or Removed