# Microsoft Sentinel All-in-One

<p align="center">
<img src="./Media/Sentinel All-in-One logo.jpg?raw=true">
</p>

Avertium's Fork of Azure's Microsoft All-in-one V2 template is aimed at helping Avertium Engineers quickly create a full Microsoft Sentinel environment, speeding up deployment and configuration tasks with a quick and intuitive wizard, saving time and simplifying setup.

A link to the original repo and template is <a href="https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Sentinel-All-In-One">here</a>.

## What does All-in-One do?

Microsoft Sentinel All-in-One automates the following tasks:

- Creates resource group
- Creates Log Analytics workspace
- Installs Microsoft Sentinel on top of the workspace
- Sets workspace retention, daily cap and commitment tiers if desired
- Enables UEBA with the relevant identity providers (Entra ID and/or Active Directory)
- Enables health diagnostics for Analytics Rules, Data Connectors and Automation Rules
- Installs Content Hub solutions from a predefined list in three categories: Microsoft, Essentials and Training
- ✨Installs Avertium's standard connector set (see [Standard connector set](#standard-connector-set))
- Enables Data Connectors from this list:
    + Entra ID (with the ability to select which log types will be ingested)
    + ✨Azure Activity (from current subscription)
    + Dynamics 365
    + Microsoft Insider Risk Management
    + Microsoft Project
    + Office 365
- Enables analytics rules (Scheduled and NRT) included in the selected Content Hub solutions, with the ability to filter by severity
- ✨Connects to an Azure Lighthouse offer for Avertium Managed Services

✨ Denotes customizations made in this fork of the All-in-One template

## Prerequisites

- Azure Subscription
- Azure user account with enough permissions to enable the desired connectors. See table at the end of this page for additional permissions. Write permissions to the workspace are **always** needed.
- Some data connectors require the relevant licence in order to be enabled. See table at the end of this page for details.

## Try it now!

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAvertium%2FMoM_SentinelBootstrap_v2%2Fmaster%2Fazuredeploy.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2FAvertium%2FMoM_SentinelBootstrap_v2%2Fmaster%2FcreateUiDefinition.json" target="_blank">
    <img src="https://aka.ms/deploytoazurebutton""/>
</a>

## Parameters

Each parameter is listed with the wizard field where it is set. Parameters marked *Parameter file only* are not shown in the wizard.

### Basics

| Parameter | Wizard field | Description | Default |
| --------- | ------------ | ----------- | ------- |
| `rgName` | Resource Group name | Name of the resource group to create. The workspace and all related resources are deployed into it. | |
| `location` | Region | Azure region for the resource group and the workspace. | Region selected in the wizard |
| `workspaceName` | Workspace Name | Name of the Log Analytics workspace where Microsoft Sentinel is enabled. | |
| `pricingTier` | Select pricing tier for Sentinel and Log Analytics | `PerGB2018` (Pay-as-you-go): billed per GB ingested. `CapacityReservation` (Commitment Tier): a fixed daily volume at a lower price per GB. The full commitment is billed every day, even if less data is ingested. The tier can be increased at any time and lowered once every 31 days. | `PerGB2018` |
| `capacityReservation` | Select Commitment Tier for Sentinel and Log Analytics | Daily volume in GB for the Commitment Tier: 100, 200, 300, 400, 500, 1000, 2000 or 5000. Only applies when `pricingTier` is `CapacityReservation`, and is ignored otherwise. | `100` |
| `dailyQuota` | Daily ingestion limit in GBs | Daily cap on ingested data, in GB. When the cap is reached, ingestion stops until the daily reset and new data is dropped, which leaves a gap in detections. `0` means no cap. | `0` |
| `dataRetention` | Number of days of retention | Number of days data is kept in the workspace (30 to 730). Microsoft Sentinel includes the first 90 days at no extra cost. | `90` |
| `immediatePurgeDataOn30Days` | *Parameter file only* | Only applies when `dataRetention` is 30. If `true`, data older than 30 days is deleted immediately. If `false`, it is deleted after a short grace period. Has no effect with any other retention value. | `true` |

### Settings

| Parameter | Wizard field | Description | Default |
| --------- | ------------ | ----------- | ------- |
| `enableUeba` | Enable User Entity Behavior Analytics (UEBA)? | Enables UEBA in Microsoft Sentinel. Requires Global Admin or Security Admin. | `true` |
| `identityProviders` | Select which Identity Providers will be synchronized with UEBA | `AzureActiveDirectory` (Entra ID) and/or `ActiveDirectory` (requires Microsoft Defender for Identity). | |
| `enableDiagnostics` | Enable Sentinel health diagnostics? | Sends health data for analytics rules, data connectors and automation rules to the workspace. | |

### Content Hub solutions

Solutions are identified by their Content Hub ID, for example `azuresentinel.azure-sentinel-solution-azureactivedirectory`. Display names are also accepted.

| Parameter | Wizard field | Description | Default |
| --------- | ------------ | ----------- | ------- |
| `enableSolutions1P` | Select Microsoft Content Hub solutions to install | Microsoft solutions to install. Each solution adds its data connectors, analytics rule templates, workbooks and hunting queries. | [Standard connector set](#standard-connector-set) |
| `enableSolutionsEssentials` | Select Essentials Content Hub solutions to install | Essentials solutions (ASIM-based detections, SOC Handbook, SOC Process Framework, UEBA Essentials). | All |
| `enableSolutionsTraining` | Select Training and Tutorials Content Hub solutions to install | KQL Training and Microsoft Sentinel Training Lab. | All |

### Data connectors

| Parameter | Wizard field | Description | Default |
| --------- | ------------ | ----------- | ------- |
| `enableDataConnectors` | Select data connectors to onboard | Connectors configured by the template: `AzureActiveDirectory` (Entra ID), `AzureActivity`, `Dynamics365`, `OfficeIRM`, `Office365Project`, `Office365`. Entra ID creates a tenant diagnostic setting named `<workspaceName>-aadDiagnosticSetttings`. Azure Activity creates a subscription diagnostic setting named `<workspaceName>-AzureActivity`. | Entra ID, Azure Activity, Office 365 |
| `aadStreams` | Select Entra ID log types to enable | Entra ID log types sent to the workspace. Sign-in and Microsoft Graph activity logs require Entra ID P1 or P2. Network access, remote network health and enriched Office 365 logs require Global Secure Access. If empty, the Entra ID diagnostic setting is not created. | All 15 log types |

### Analytics rules

| Parameter | Wizard field | Description | Default |
| --------- | ------------ | ----------- | ------- |
| `enableScheduledAlerts` | Enable Scheduled alert rules for selected Content Hub solutions and Data Connectors? | Creates analytics rules from the rule templates of the installed solutions. | `false` |
| `severityLevels` | Select the severity of the rules to enable | Only rule templates with these severities are enabled: `High`, `Medium`, `Low`, `informational`. | `High`, `Medium` |

### XDR Lighthouse connection

| Parameter | Wizard field | Description | Default |
| --------- | ------------ | ----------- | ------- |
| `enableXDRLighthouse` | Enable XDR Lighthouse connection for Avertium Managed Services? | Delegates access to the subscription to Avertium through Azure Lighthouse. | `true` |
| `mspTenantId` | Managing Tenant ID | Tenant ID of the managing (Avertium) tenant. Required when `enableXDRLighthouse` is `true`. | |

### Template location

| Parameter | Wizard field | Description | Default |
| --------- | ------------ | ----------- | ------- |
| `_artifactsLocation` | *Parameter file only* | Base URL from which the linked templates and scripts are downloaded. Must end with `/`. | `https://raw.githubusercontent.com/Avertium/MoM_SentinelBootstrap_v2/master/` |

## Standard connector set

The following Content Hub solutions are selected by default, together with their analytics rules:

| Connector | How it's connected |
| --------- | ------------------ |
| Azure Activity, Microsoft Entra ID, Microsoft 365 | Connected by the template |
| Azure Key Vault, Azure Storage Account, Azure Web Application Firewall (WAF), Network Security Groups | After deployment: add a diagnostic setting to each resource, sending logs to the workspace |
| Common Event Format (CEF) via AMA, Syslog via AMA, Custom logs via AMA | After deployment: create a Data Collection Rule and install the Azure Monitor Agent on the forwarders or servers |
| Threat Intelligence (Microsoft Defender Threat Intelligence) | After deployment: open the **Microsoft Defender Threat Intelligence** connector and select **Connect** |
| Microsoft Defender XDR, Defender for Endpoint, Defender for Identity, Defender for Office 365, Defender for Cloud Apps, Defender alerts, Defender Vulnerability Management, Purview DLP, Entra ID Protection | After deployment: in the **Microsoft Defender XDR** connector, turn on incidents and the event tables you need |

Analytics rules whose tables don't exist yet in the workspace can be enabled later from **Analytics > Rule templates**.

## Parameter file

`azuredeploy.parameters.json` contains the standard Avertium selection. Replace `<RESOURCE_GROUP_NAME>`, `<WORKSPACE_NAME>` and `<MANAGING_TENANT_ID>` before deploying.

To use it: open the custom deployment, select **Edit parameters** > **Load file**, choose `azuredeploy.parameters.json` and save.

## Deployment notes

- **Diagnostic settings limit:** Entra ID (tenant) and each subscription allow a maximum of 5 diagnostic settings. If the limit is reached, the deployment fails with *"The limit of 5 diagnostic settings was reached"*. Delete an unused setting (Entra ID > Monitoring & health > Diagnostic settings, or Monitor > Activity log > Export Activity Logs) and redeploy.
- **Redeploying:** to an existing workspace, set `enableUeba` to `false`, otherwise the `settings` step fails with *"Update request should provide ETag"*.

## Supported connectors

The following table summarizes licenses, permissions and cost for each data connector configured by the template:

| Data Connector                                 | License         |  Permissions                    | Cost      |
| ---------------------------------------------- | --------------- |---------------------------------|-----------|
| Entra ID (Tenant scope version only)           | Any Entra ID license; P1/P2 for sign-in and Graph activity logs; Global Secure Access for network and enriched Office 365 logs | Global Admin or Security Admin  | Billed    |
| Azure Activity                                 | None            | Subscription Owner or Monitoring Contributor | Free      |
| Dynamics 365                                   | D365 license    | Global Admin or Security Admin  | Billed    |
| Microsoft Insider Risk Management              | IRM license     | Global Admin or Security Admin  | Free      |
| Microsoft Project                              | MS Project license | Global Admin or Security Admin | Billed  |
| Office 365                                     | None            | Global Admin or Security Admin  | Free      |
