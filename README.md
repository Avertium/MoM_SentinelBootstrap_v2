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
- Enables UEBA with the relevant identity providers (AAD and/or AD)
- Enables health diagnostics for Analytics Rules, Data Connectors and Automation Rules
- Installs Content Hub solutions from a predefined list in three categories: 1st party, Essentials and Training
- Enables Data Connectors from this list:
    + Entra ID (with the ability to select which data types will be ingested)
    + ✨Azure Activity (from current subscription, via a subscription diagnostic setting)
    + Dynamics 365
    + Microsoft 365 Defender
    + Microsoft Defender for Cloud
    + Microsoft Insider Risk Management
    + Microsoft Power BI
    + Microsoft Project
    + Office 365
    + Threat Intelligence Platforms
    + ✨Enable and configure TAXII Threat Intelligence
- Enables analytics rules (Scheduled and NRT) included in the selected Content Hub solutions, with the ability to filter by severity
- Enables analytics rules (Scheduled and NRT) that use any of the selected Data connectors, with the ability to filter by severity
- ✨Connects to an Azure lighthouse

✨ Denotes customizations made in this fork of the All-in-One template

## Changes in v2

This repo (`MoM_SentinelBootstrap_v2`) replaces the original `MoM_SentinelBootstrap` repo. Changes:

- **Removed FusionEngine connectors.** The FusionEngine - LogRhythm, FusionEngine - SentinelOne and FusionEngine - Splunk options were removed from the wizard, along with their custom tables, data collection rules and data collection endpoint (`customLogTables.json`, `dataCollectionRules.json`, `dataCollectionEndpoint.json`).
- **Removed Entra ID Identity Protection connector.** Workspaces onboarded to the Microsoft Defender portal (new Sentinel workspaces are often onboarded automatically) receive Identity Protection alerts through Defender XDR, and the legacy connector can no longer be changed from Sentinel. Deploying it failed with *"The workspace is enabled through the Microsoft Threat Protection Portal"*.
- **Azure Activity uses a subscription diagnostic setting.** The legacy `AzureActivityLog` data source is deprecated by Microsoft (*DataSourceDeprecated*). The template now creates a subscription diagnostic setting named `<workspaceName>-AzureActivity` that sends all Activity log categories to the workspace.
- **Entra ID diagnostic setting can be skipped.** If `aadStreams` is empty, the Entra ID diagnostic setting is not created.
- **Repo location updated.** The Deploy to Azure button and the `_artifactsLocation` default now point to `https://raw.githubusercontent.com/Avertium/MoM_SentinelBootstrap_v2/main/`.
- **Parameter file added.** `azuredeploy.parameters.json` prefills the wizard with Avertium's standard selections (see below).

## Deployment notes

- **This repo must be public.** Azure downloads `azuredeploy.json`, `createUiDefinition.json` and everything in `LinkedTemplates/` anonymously from `_artifactsLocation`. If the repo is private, or the files are not at the repo root, the deployment fails or uses outdated templates.
- **Testing a branch:** set `_artifactsLocation` to `https://raw.githubusercontent.com/Avertium/MoM_SentinelBootstrap_v2/<branch>/` (keep the trailing slash). Changing only the Deploy to Azure link is not enough, because the linked templates are always downloaded from `_artifactsLocation`.
- **Diagnostic settings limit:** Entra ID (tenant) and each subscription allow a maximum of 5 diagnostic settings. If the limit is reached, the deployment fails with *"The limit of 5 diagnostic settings was reached"*. Delete an unused setting (Entra ID > Monitoring & health > Diagnostic settings, or Monitor > Activity log > Export Activity Logs) and redeploy.
- **Redeploying is safe.** All deployments are incremental. Steps that already succeeded are updated in place, not duplicated.

## Parameter file

`azuredeploy.parameters.json` prefills the deployment with:

| Setting | Value |
| ------- | ----- |
| Resource group / Workspace name | `<RESOURCE_GROUP_NAME>` / `<WORKSPACE_NAME>` (replace before deploying) |
| UEBA | Enabled (identity provider: Entra ID) |
| Sentinel health diagnostics | Enabled |
| Microsoft Content Hub solutions | Entra ID, Azure Activity, Microsoft 365, Microsoft Insider Risk Management, Microsoft Teams, Threat Intelligence |
| Essentials solutions | All |
| Training and Tutorials solutions | All |
| Data connectors | Entra ID (Sign-in and Audit logs), Azure Activity, Microsoft Insider Risk Management, Office 365, Threat Intelligence Platforms |
| Scheduled alert rules | Enabled (High and Medium severity) |
| XDR Lighthouse connection | Enabled, Managing Tenant ID `<MANAGING_TENANT_ID>` (replace before deploying) |
| Location / Pricing / Retention / Daily cap | `westus2` / Pay-as-you-go / 90 days / none |

To use it: open the custom deployment, select **Edit parameters** > **Load file**, choose `azuredeploy.parameters.json`, replace the placeholders and save.

## Prerequisites

- Azure Subscription
- Azure user account with enough permissions to enable the desired connectors. See table at the end of this page for additional permissions. Write permissions to the workspace are **always** needed.
- Some data connectors require the relevant licence in order to be enabled. See table at the end of this page for details.

## Try it now!

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAvertium%2FMoM_SentinelBootstrap_v2%2Fmain%2Fazuredeploy.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2FAvertium%2FMoM_SentinelBootstrap_v2%2Fmain%2FcreateUiDefinition.json" target="_blank">
    <img src="https://aka.ms/deploytoazurebutton""/>
</a>

## Supported connectors

The following table summarizes permissions, licenses and permissions needed and related cost to enable each Data Connector:

| Data Connector                                 | License         |  Permissions                    | Cost      |
| ---------------------------------------------- | --------------- |---------------------------------|-----------|
| Entra ID (Tenant scope version only)           | Any AAD license | Global Admin or Security Admin  | Billed    |
| Azure Activity                                 | None            | Subscription Owner or Monitoring Contributor (to create the diagnostic setting) | Free      |
| Dynamics 365                                   | D365 license    | Global Admin or Security Admin  | Billed    |
| Microsoft Insider Risk Management              | IRM license     | Global Admin or Security Admin  | Free      |
| Microsoft PowerBi                              | PowerBi license | Global Admin or Security Admin  | Billed    |
| Microsoft Project                              | MS Project license | Global Admin or Security Admin | Billed  |
| Office 365                                     | None            | Global Admin or Security Admin  | Free      |
| Threat Intelligence Platforms                  | None            | Global Admin or Security Admin  | Billed    |
