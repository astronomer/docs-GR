---
sidebar_label: 'Azure'
title: 'Install Astro on Azure'
id: install-azure
description: Get started on Astro by creating your first Astro cluster on Azure.
sidebar_custom_props: { icon: 'img/azure.png' }
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

To complete the installation process, you'll:

- Add the Astronomer Service Principal to your Azure Active Directory (Azure AD) instance.
- Assign the Astronomer Service Principal an Owner role to your subscription.
- Register Microsoft Azure features.

When you've completed the installation process, Astronomer support creates a cluster within your Azure subscription to host the resources and Apache Airflow components necessary to deploy DAGs and execute tasks.

For more information about managing Azure subscriptions with the Azure CLI, see [How to manage Azure subscriptions with the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli).

## Prerequisites

- A clean Azure subscription. For security reasons, Azure subscriptions with existing tooling running aren't supported. Also, the subscription must be included in an Azure management group that doesn't apply Azure policies. See [What are Azure management groups](https://docs.microsoft.com/en-us/azure/governance/management-groups/overview).
- An Azure AD user with the following role assignments:
    - `Application Administrator`. See [Understand roles in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/concept-understand-roles).
    - `Owner` with permission to create and manage subscription resources of all types. See [Azure built-in roles](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles).

    These role assignments are required for cluster creation, and can be removed after the cluster is created.
- Microsoft Azure CLI or Azure Az PowerShell module.  See [How to install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) and [Install the Azure Az PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps).
- A minimum quota of 48 Standard Ddv5-series vCPUs in the deployment region. You can use Dv5-series vCPUs, but you'll need 96 total vCPUs composed of 48 Ddv5-series vCPUs and 48 Dv5-series vCPUs. To adjust your quota limits up or down, see [Increase VM-family vCPU quotas](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/per-vm-quota-requests).
- A subscription to the [Astro Status Page](https://status.astronomer.io). This ensures that you're alerted when an incident occurs or when scheduled maintenance is planned.
- The following domains added to your organization's allowlist:
    - `https://cloud.astronomer.io/`
    - `https://astro-<your-org>.datakin.com/`
    - `https://<your-org>.astronomer.run/`
    - `https://api.astronomer.io/`
    - `https://images.astronomer.cloud/`
    - `https://auth.astronomer.io/`
    - `https://updates.astronomer.io/`
    - `https://install.astronomer.io/`

For more information about the resources required to run Astro on Azure, see [Azure Resource Reference](resource-reference-azure.md).

### VPC peering prerequisites (Optional)

If any Azure resources are on a private network, you can choose one of the following options:

- Use the public internet and allow-lists for communication.
- Create a VPC Peering connection between the Astronomer VPC and your network VPCs.

To implement the second option, you'll additionally need:

- A CIDR block (RFC 1918 IP Space) no smaller than a `/19` range. You must ensure it does not overlap with the AWS VPC(s) that you will be peering with later. The default CIDR range is `172.20.0.0/19`.
- VPC Name / ID for peering with Astronomer.
- The IP addresses of your DNS servers.

## Step 1: Access Astro

1. Go to https://cloud.astronomer.io/ and create an Astronomer account.

2. Go to `https://cloud.astronomer.io/login`, enter your email address, and then click **Continue**.

3. Select one of the following options to access the Cloud UI:

    - Enter your password and click **Continue**.
    - To authenticate with an identity provider (IdP), click **Continue with SSO**, enter your username and password, and then click **Sign In**.
    - To authenticate with your GitHub account, click **Continue with GitHub**, enter your username or email address, enter your password, and then click **Sign in**.
    - To authenticate with your Google account, click **Continue with Google**, choose an account, enter your username and password, and then click **Sign In**.

    If you're the first person in an Organization to authenticate, you're added as a Workspace Admin to a new Workspace named after your Organization. You can add other team members to the Workspace without the assistance of Astronomer support. See [Add a user](add-user.md). To integrate an identity provider (IdP) with Astro, see [Set up an identity provider](configure-idp.md).

## Step 2: Prepare for data plane activation

The data plane is a collection of Astro infrastructure components that run in your cloud and are managed by Astronomer. This includes a central database, storage for Airflow tasks logs, and the resources required for task execution.

<Tabs
    defaultValue="azure"
    groupId= "step-2-prepare-for-data-plane-activation"
    values={[
        {label: 'Azure CLI on Bash', value: 'azure'},
        {label: 'PowerShell', value: 'powershell'},
    ]}>
<TabItem value="azure">

1. Run the following command to log in to your Azure account:

    ```sh
    az login
    ```
2. Run the following command to select your Azure subscription:

    ```sh
    az account set -s <subscription-id>
    ```
3. Run the following command to add the Astronomer Service Principal to Azure AD:

    ```sh
    az ad sp create --id a67e6057-7138-4f78-bbaf-fd9db7b8aab0
    ```
4. Run the following commands to get details about the Azure subscription and create a new role assignment for the Astronomer service principal:

    ```sh
    subid=$(az account show --query id --output tsv)
    az role assignment create --assignee a67e6057-7138-4f78-bbaf-fd9db7b8aab0 --role Owner --scope /subscriptions/$subid
    ```
5. Run the following commands to register the `EncryptionAtHost` feature:

    ```sh
    az feature register --namespace "Microsoft.Compute" --name "EncryptionAtHost"
    while [ $(az feature list --query "[?contains(name, 'Microsoft.Compute/EncryptionAtHost')].{State:properties.state}" -o tsv) != "Registered" ]
    do
    echo "Still waiting for Feature Registration (EncryptionAtHost) to complete, this can take up to 15 minutes"
    sleep 60
    done
    echo "Registration Complete"
    az provider register --namespace Microsoft.Compute
    ```

</TabItem>

<TabItem value="powershell">

1. Run the following command to log in to your Azure account:

    ```sh
    Connect-AzAccount
    ```

2. Run the following command to select your Azure subscription:

    ```sh
    Set-AzContext -SubscriptionId <subscription-id>
    ```
3. Run the following command to create the Astronomer service principal:

    ```sh
    $sp = New-AzADServicePrincipal -AppId a67e6057-7138-4f78-bbaf-fd9db7b8aab0
    ```
4. Run the following commands to get details about the Azure subscription and create a new role assignment for the Astronomer service principal:

    ```sh
    $sp = Get-AzADServicePrincipal -ApplicationId a67e6057-7138-4f78-bbaf-fd9db7b8aab0
    ```
    ```sh
    $subid = (Get-AzContext).Subscription.id
    ```
    ```sh
    $ra = New-AzRoleAssignment -ObjectId $sp.id -RoleDefinitionName Owner -Scope "/subscriptions/$subid"
    ```
5. Run the following commands to register the `EncryptionAtHost` feature:

    ```sh
    Register-AzProviderFeature -FeatureName EncryptionAtHost -ProviderNamespace Microsoft.Compute
    while ( (Get-AzProviderFeature -FeatureName EncryptionAtHost -ProviderNamespace Microsoft.Compute).RegistrationState -ne "Registered") {echo "Still waiting for Feature Registration (EncryptionAtHost) to complete, this can take up to 15 minutes"; sleep 60} echo "Registration Complete"
    ```
    ```sh
    Register-AzResourceProvider -ProviderNamespace Microsoft.compute
    ```

</TabItem>
</Tabs>

## Step 3: Provide setup information to Astronomer support

After you've prepared your environment for data plane activation, provide Astronomer support with the following information:

- Your preferred Astro cluster name.
- Your Azure TenantID and SubscriptionID.
- Optional. Your preferred node instance type. The default is Standard_D4d_v5.
- Optional. Your preferred Postgres Flexible Server instance type. The default is Standard_D4ds_v4.
- Optional. Your preferred maximum node count.
- Optional. Your custom CIDR ranges for Astronomer service connections. The default is `172.20.0.0/19`.

If you don't specify a preferred configuration for your organization, Astronomer support creates a cluster in `CentralUS` with the default configurations for Astro on Azure. See [Azure resource reference](resource-reference-azure.md).

## Step 4: Astronomer support creates the cluster

After you provide Astronomer support with the setup information for your organization, Astronomer support creates your first cluster on Azure.

Wait for confirmation from Astronomer support that the cluster has been created before creating a Deployment.

## Step 5: Create a Deployment

When Astronomer confirms that your Astro cluster has been created, you can create a Deployment and start deploying DAGs. Log in to the [Cloud UI](log-in-to-astro.md#log-in-to-the-cloud-ui) and [create a new Deployment](create-deployment.md). If the installation is successful, your new Astro cluster is listed as an option in the Cloud UI **Cluster** list:

<div class="text--center">
  <img src="/img/docs/create-new-deployment-select-cluster.png" alt="Cloud UI New Deployment screen" />
</div>

## Next steps

- [Set up an identity provider](configure-idp.md)
- [Install CLI](cli/overview.md)
- [Configure Deployments](configure-deployment-resources.md)
- [Deploy code](deploy-code.md)
- [Add users](add-user.md)
