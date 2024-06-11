---
title: Azure Hybrid Benefit for Linux virtual machines
description: Learn how Azure Hybrid Benefit can save you money on Linux virtual machines.
services: virtual-machines
author: vvarshney06
manager: gachandw
ms.service: virtual-machines
ms.subservice: billing
ms.collection: linux
ms.topic: conceptual
ms.date: 05/02/2023
ms.author: vvarshney
ms.reviewer: mattmcinnes
ms.custom: kr2b-contr-experiment, linux-related-content, devx-track-azurecli
---

# Azure Hybrid Benefit for Red Hat Enterprise Linux (RHEL) and SUSE Linux Enterprise Server (SLES) virtual machines

Azure Hybrid Benefit (AHB) for Linux lets you easily switch the software subscription model for your VM. You can remove licensing cost by bringing your Red Hat and SUSE Linux subscriptions directly to Azure, or utilize a model where you pay for subscriptions as you use them. This article defines 'BYOS' and 'PAYG' licensing models, compares the benefits of each model, and shows how you can use the Azure Hybrid Benefit to switch between the two at any point. This process applies to Virtual Machine Scale Sets, Spot Virtual Machines, and custom images. It allows for seamless bi-directional conversions between the two models.

Customers may see savings estimated to up to 76% with Azure Hybrid Benefit for Linux. Savings estimates are based on one standard D2s v3 Azure VM with RHEL or SLES subscription in the East US region running at a pay-as-you-go rate vs a reduced rate for a 3-year reserved instance plan. Based on Azure pricing as of October 2022. Prices subject to change. Actual savings may vary based on location, instance type, or usage.

> [!TIP]
> Try the **[Azure Hybrid Benefit Savings Calculator](https://azure.microsoft.com/pricing/hybrid-benefit/#calculator)** to visualize the cost saving benefits of this feature.

## Defining Pay-as-you-go (PAYG) and Bring-your-own-subscription (BYOS)

In Azure, there are two main licensing pricing options: 'pay-as-you-go' (PAYG) and 'bring-your-own-subscription' (BYOS). 'PAYG' is a pricing option where you pay for the resources you use on an hourly or monthly basis. You only pay for what you use and can scale up or down as needed. On the other hand, 'BYOS' is a licensing option where you can use your existing licenses for certain software, in this case RHEL and SLES, on Azure virtual machines. You can use your existing licenses and don't have to purchase new ones for use in Azure.

:::image type="content" source="./media/ahb-linux/azure-hybrid-benefit-compare.png" alt-text="Diagram that shows the use of Azure Hybrid Benefit to switch Linux virtual machines between pay-as-you-go and bring-your-own-subscription.":::

> [!NOTE]
> Virtual machines deployed from PAYG images or VMs converted from BYOS models incur *both* an infrastructure fee and a software fee. If you have your own license, use Azure Hybrid Benefit to convert from a PAYG to BYOS model.

You can use Azure Hybrid Benefit to switch back to pay-as-you-go billing at any time.

## Which Linux virtual machines qualify for Azure Hybrid Benefit?

Azure dedicated host instances and SQL hybrid benefits aren't eligible for Azure Hybrid Benefit if you already use Azure Hybrid Benefit with Linux virtual machines.

## Enabling Azure Hybrid Benefit

### Enabling AHB on New VMs
You can invoke AHB at the time of virtual machine creation. Benefits of doing so are threefold:

- You can provision both PAYG and BYOS virtual machines by using the same image and process.
- It enables future licensing mode changes.
- The virtual machine is connected to Red Hat Update Infrastructure (RHUI) by default, to help keep it up to date and secure. You can change the updated mechanism after deployment at any time.

#### [Azure portal](#tab/ahbNewPortal)
To enable Azure Hybrid Benefit when you create a virtual machine, use the following procedure. (The SUSE workflow is the same as the RHEL example shown here.)

1. Go to the [Azure portal](https://portal.azure.com/).
1. Go to **Create a virtual machine**.

   ![Screenshot of the portal page for creating a virtual machine.](./media/)
1. In the **Licensing** section, select the checkbox that asks if you want to use an existing RHEL subscription and the checkbox to confirm that your subscription is eligible.

   ![Screenshot of the Azure portal that shows checkboxes selected for licensing.](./media/azure-hybrid-benefit/create-vm-ahb-checkbox.png)
1. Create a virtual machine by following the next set of instructions.
1. On the **Configuration** pane, confirm that the option is enabled.

   ![Screenshot of the Azure Hybrid Benefit configuration pane after you create a virtual machine.](./media/azure-hybrid-benefit/create-configuration-blade.png)

#### [Azure CLI](#tab/ahbNewCli)

You can use the `az vm extension` and `az vm update` commands to update new virtual machines after they've been created.

1. Install the extension
   ```azurecli
   az vm extension
   ```

1. Update with the correct license type
   ```azurecli
   az vm update
   ```

- RHEL License Types: RHEL_BASE, RHEL_EUS, RHEL_SAPAPPS, RHEL_SAPHA, RHEL_BASESAPAPPS, RHEL_BASESAPHA​
- SLES License Types: SLES_STANDARD, SLES_SAP, SLES_HPC​

---
### Enabling AHB on Existing VM
#### [Azure portal](#tab/ahbExistingPortal)
To enable Azure Hybrid Benefit on an existing virtual machine:

1. Go to the [Azure portal](https://portal.azure.com/).
1. Open the virtual machine page on which you want to apply the conversion.
1. Go to **Configuration** > **Licensing**. To enable the Azure Hybrid Benefit conversion, select **Yes**, and then select the confirmation checkbox.

![Screenshot of the Azure portal that shows the Licensing section of the configuration page for Azure Hybrid Benefit.](./media/azure-hybrid-benefit/create-configuration-blade.png)

#### [Azure CLI](#tab/ahbExistingCli)

You can use the `az vm extension` and `az vm update` commands to update existing virtual machines.

1. Install the extension
   ```azurecli
   az vm extension
   ```
   > [!Note]
   > The complete az vm extension depends on the particular distribution you are using, please refer to the next section for the complete details.

1. Update with the correct license type
   ```azurecli
   az vm update
   ``````

- RHEL License Types: RHEL_BASE, RHEL_EUS, RHEL_SAPAPPS, RHEL_SAPHA, RHEL_BASESAPAPPS, RHEL_BASESAPHA​
- SLES License Types: SLES_STANDARD, SLES_SAP, SLES_HPC​

---

## Check the current licensing model of an AHB enabled VM

It is required the Azure Hybrid Benefit extension be installed on the VM to switch the licensing model from BYOS to PAYG or vice versa. You can view whether the agent is installed using the Azure CLI or the Azure Instance Metadata Service.

### [Azure CLI](#tab/licenseazcli)

1. You can use the `az vm get-instance-view` command to check whether the extension is installed or not. Look for the `AHBForSLES` or `AHBForRHEL` extension, if the corresponding one is installed, the Azure Hybrid Benefit has been enabled,
review the license type to review which licensing model your VM is using.

   ```azurecli
   az vm get-instance-view -g MyResourceGroup -n myVm --query instanceView.extensions
   ```
1. Once the corresponding Red Hat or SUSE Hybrid beneift extension is installed, use the following command to review the license type the machine is using.

   ```azurecli
   az vm get-instance-view -g MyResourceGroup -n myVM --query licenseType
   ```

1. The following license types correspond to PAYG model.

   - For RHEL: `RHEL_BASE`, `RHEL_EUS`, `RHEL_SAPAPPS`, `RHEL_SAPHA`, `RHEL_BASESAPAPPS`, or `RHEL_BASESAPHA`.
   - For SLES: `SLES`, `SLES_SAP`, or `SLES_HPC`

1. These ones correspond to BYOS.

   - For RHEL: `RHEL_BYOS`
   - For SLES: `SLES_BYOS`

If the license type of the VM has not been modified, the previous command returns an empty string and the VM continues to use the billing model of the image used to deploy it.


### [Azure PowerShell](#tab/licensepowershell)

1. You can use the `az vm get-instance-view` command to check whether the extension is installed or not. Look for the `AHBForSLES` or `AHBForRHEL` extension, if the corresponding one is installed, the Azure Hybrid Benefit has been enabled,
review the license type to review which licensing model your VM is using.

   ```azurepowershell
   Get-AzVM -ResourceGroupName MyResourceGroup -Name myVM -Status | Select-Object -ExpandProperty Extensions
   ```

1. Once the corresponding Red Hat or SUSE Hybrid beneift extension is installed, use the following command to review the license type the machine is using.

   ```azurepowershell
   Get-AzVM -ResourceGroupName MyResourceGroup -Name myVM).LicenseType
   ```

1. The following license types correspond to PAYG model.

   - For RHEL: `RHEL_BASE`, `RHEL_EUS`, `RHEL_SAPAPPS`, `RHEL_SAPHA`, `RHEL_BASESAPAPPS`, or `RHEL_BASESAPHA`.
   - For SLES: `SLES`, `SLES_SAP`, or `SLES_HPC`

1. These ones correspond to BYOS.

   - For RHEL: `RHEL_BYOS`
   - For SLES: `SLES_BYOS`

If the license type of the VM has not been modified, the previous command returns an empty string and the VM continues to use the billing model of the image used to deploy it.

---

## PAYG to BYOS conversions
---
### Convert a Pay As You Go(PAYG) image to BYOS using the Azure CLI
If you deployed an Azure Marketplace image with PAYG licensing model and desire to convert it to BYOS, follow this process to convert it to the desired licensing model.

#### [Red Hat (RHEL)](#tab/rhelAzcliByosConv)

1. Apply the `RHEL_BYOS` license type to the machine:

    ```azurecli
    # This will enable BYOS on a RHEL(PAYG) virtual machine using Azure Hybrid Benefit
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS
    ```
1. Once the PAYG to BYOS conversion is complete, you must register the machine with Red Hat for system updates and usage compliance.

1. If you desire to return to PAYG model, you need to set up the license-type to "None", otherwise, it continues to be BYOS.
    ```azurecli
    # If the image started as PAYG and was converted to BYOS, the following command will revert it back to PAYG.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
    ```


#### [SUSE (SLES)](#tab/slesAzcliByosConv)

1. Apply the `SLES_BYOS` license type to the virtual machine.

    ```azurecli
    # This will enable BYOS on a SLES virtual machine
    az vm update -g myResourceGroup -n myVmName --license-type SLES_BYOS
    ```

1. Once the PAYG to BYOS conversion is complete, you must register the machine on your own with SUSE for software updates and usage compliance.

1. If you desire to return to PAYG model, you need to set up license-type to "None", otherwise, it continues to be BYOS.
    ```azurecli
    # If the image started as PAYG and was converted to BYOS, the following command will revert it back to PAYG.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
    ```

---

## BYOS to PAYG conversions
Converting to PAYG model is supported for Azure Marketplace images labeled BYOS, machines imported from on-premises or a third party cloud provider.

#### [Red Hat (RHEL)](#tab/rhelazclipaygconv)

1. Install the Azure Hybrid Benefit extension on a running virtual machine. You can use the following command via the Azure CLI:
    ```azurecli
    az vm extension set -n AHBForRHEL --publisher Microsoft.Azure.AzureHybridBenefit --vm-name myVMName --resource-group myResourceGroup
    ```

1. After the extension is installed successfully, change the license type based on what you need:

    ```azurecli
    # This will enable Azure Hybrid Benefit to fetch software updates for RHEL base/regular repositories
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BASE

    # This will enable Azure Hybrid Benefit to fetch software updates for RHEL EUS repositories
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_EUS

    # This will enable Azure Hybrid Benefit to fetch software updates for RHEL SAP APPS repositories
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_SAPAPPS

    # This will enable Azure Hybrid Benefit to fetch software updates for RHEL SAP HA repositories
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_SAPHA

    # This will enable Azure Hybrid Benefit to fetch software updates for RHEL BASE SAP APPS repositories
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BASESAPAPPS

    # This will enable Azure Hybrid Benefit to fetch software updates for RHEL BASE SAP HA repositories
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BASESAPHA
   ```

1. If you desire to return to BYOS model, you need to set up license-type to "None", otherwise, it continues to be PAYG.
    ```azurecli
    # If the image started as BYOS and was converted to PAYG, the following command will revert it back to BYOS.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
    ```

#### [SUSE (SLES)](#tab/slesazclipaygconv)

1. Install the Azure Hybrid Benefit extension on a running virtual machine. You can use the Azure portal or use the following command via the Azure CLI:
    ```azurecli
    az vm extension set -n AHBForSLES --publisher SUSE.AzureHybridBenefit --vm-name myVMName --resource-group myResourceGroup
    ```

1. After the extension is installed successfully, change the license type based on what you need:

    ```azurecli
    # This will enable Azure Hybrid Benefit to fetch software updates for SLES Standard repositories
    az vm update -g myResourceGroup -n myVmName --license-type SLES

    # This will enable Azure Hybrid Benefit to fetch software updates for SLES SAP repositories
    az vm update -g myResourceGroup -n myVmName --license-type SLES_SAP

    # This will enable Azure Hybrid Benefit to fetch software updates for SLES HPC repositories
    az vm update -g myResourceGroup -n myVmName --license-type SLES_HPC
    ```

1. If you desire to return to BYOS model, you need to set up the "None" license type, otherwise, it continues to be PAYG.
    ```azurecli
    # If the image started as BYOS and was converted to PAYG, the following command will revert it back to BYOS.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
    ```
---

#### Multiple VMs

The following command converts the machines specified in the argument to BYOS.

```azurecli
# This will enable BYOS on a RHEL virtual machine. In this example, ids.txt is an
# existing text file that contains a delimited list of resource IDs corresponding
# to the virtual machines using Azure Hybrid Benefit
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS --ids $(cat ids.txt)
```

The following examples show two methods of getting a list of resource IDs: one at the resource group level, and one at the subscription level.

```azurecli
# To get a list of all the resource IDs in a resource group:
az vm list -g MyResourceGroup --query "[].id" -o tsv

# To get a list of all the resource IDs of virtual machines in a subscription:
az vm list -o json | jq '.[] | {VirtualMachineName: .name, ResourceID: .id}'
```
---

### Operating system instructions
#### [Red Hat (RHEL)](#tab/rhelpaygconversion)

To start using Azure Hybrid Benefit for Red Hat:

1. Install the `AHBForRHEL` extension on the virtual machine on which you want to apply the Azure Hybrid Benefit BYOS benefit. You can do this installation via the Azure CLI or PowerShell.

1. Depending on the software updates that you want, change the license type to a relevant value. Here are the available license type values and the software updates associated with them:

    | License type  | Software updates  | Allowed virtual machines|
    |---|---|---|
    | RHEL_BASE  | Installs Red Hat regular/base repositories on your virtual machine. | RHEL BYOS virtual machines, RHEL custom image virtual machines|
    | RHEL_EUS | Installs Red Hat Extended Update Support (EUS) repositories on your virtual machine. | RHEL BYOS virtual machines, RHEL custom image virtual machines|
    | RHEL_SAPAPPS  | Installs RHEL for SAP Business Apps repositories on your virtual machine. | RHEL BYOS virtual machines, RHEL custom image virtual machines|
    | RHEL_SAPHA | Installs RHEL for SAP with High Availability (HA) repositories on your virtual machine. | RHEL BYOS virtual machines, RHEL custom image virtual machines|
    | RHEL_BASESAPAPPS | Installs RHEL regular/base SAP Business Apps repositories on your virtual machine. | RHEL BYOS virtual machines, RHEL custom image virtual machines|
    | RHEL_BASESAPHA | Installs regular/base RHEL for SAP with HA repositories on your virtual machine.| RHEL BYOS virtual machines, RHEL custom image virtual machines|

1. Wait one hour for the extension to read the license type value and install the repositories.

   > [!NOTE]
   > If the extension isn't running by itself, you can run it on demand.

1. You should now be connected to Azure Red Hat Update. The relevant repositories are installed on your machine.

1. If you want to switch back to the bring-your-own-subscription model,  just change the license type to `None` and run the extension. This action removes all Red Hat Update Infrastructure (RHUI) repositories from your virtual machine and stops the billing.

> [!Note]
> In the unlikely event that the extension can't install repositories or there are any other issues, switch the license type back to empty and reach out to Microsoft support. This ensures that you don't get billed for software updates.

#### [SUSE (SLES)](#tab/slespaygconversion)

After you successfully install the `AHBForSLES` extension, you can use the `az vm update` command to update the existing license type on your running virtual machines. For SLES virtual machines, run the command and set the `--license-type` parameter to one of the following license types: `SLES`, `SLES_SAP`, or `SLES_HPC`.

To start using Azure Hybrid Benefit for SLES virtual machines:

1. Install the `AHBForSLES` extension on the SLES virtual machine.
1. Change the license type to the value that reflects the software updates you want. Here are the available license type values and the software updates associated with them:

    | License type  | Software updates  | Allowed virtual machines|
    |---|---|---|
    | SLES | Installs SLES Standard repositories on your virtual machine. | SLES BYOS virtual machines, SLES custom image virtual machines|
    | SLES_SAP | Installs SLES SAP repositories on your virtual machine. | SLES SAP BYOS virtual machines, SLES custom image virtual machines|
    | SLES_HPC | Installs SLES High Performance Computing repositories on your virtual machine. | SLES HPC BYOS virtual machines, SLES custom image virtual machines|

1. Wait five minutes for the extension to read the license type value and install the repositories.

   > [!NOTE]
   > If the extension isn't running by itself, you can run it on demand.

1. You should now be connected to the SUSE public cloud update infrastructure on Azure. The relevant repositories are installed on your machine.

1. If you want to switch back to the bring-your-own-subscription model,  just change the license type to `None` and run the extension. This action removes all repositories from your virtual machine and stops the billing.

After you successfully install the `AHBForSLES` extension, you can use the `az vm update` command to update the existing license type on your running virtual machines. For SLES virtual machines, run the command and set the `--license-type` parameter to one of the following license types: `SLES`, `SLES_SAP`, or `SLES_HPC`.

---

## AHB for reserved instance VMs

[Azure reservations](../../cost-management-billing/reservations/save-compute-costs-reservations.md) (Azure Reserved Virtual Machine Instances) help you save money by committing to one-year or three-year plans for multiple products. Azure Hybrid Benefit for pay-as-you-go virtual machines is available for reserved instances.

If you've purchased compute costs at a discounted rate by using reserved instances, you can apply Azure Hybrid Benefit on the licensing costs for RHEL and SUSE on top of it. The steps to apply Azure Hybrid Benefit for a reserved instance remain exactly same as they are for a regular virtual machine.

![Screenshot of the interface for purchasing reservations for virtual machines.](./media/azure-hybrid-benefit/reserved-instances.png)

>[!NOTE]
>If you've already purchased reservations for RHEL or SUSE pay-as-you-go software on Azure Marketplace, please wait for the reservation tenure to finish before using Azure Hybrid Benefit for pay-as-you-go virtual machines.

## Compliance

### [Red Hat compliance](#tab/rhelcompliance)

Customers who use Azure Hybrid Benefit for pay-as-you-go RHEL virtual machines agree to the standard [legal terms](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Cloud_Software_Subscription_Agreement_for_Microsoft_Azure.pdf) and [privacy statement](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Privacy_Statement_for_Microsoft_Azure.pdf) associated with the Azure Marketplace RHEL offers.

Customers who use Azure Hybrid Benefit for pay-as-you-go RHEL virtual machines have three options for providing software updates and patches to those virtual machines:

- [Red Hat Update Infrastructure](../workloads/redhat/redhat-rhui.md) (default option)
- Red Hat Satellite Server
- Red Hat Subscription Manager

Customers can use RHUI as the main update source for Azure Hybrid Benefit for pay-as-you-go RHEL virtual machines without attaching subscriptions. Customers who choose the RHUI option are responsible for ensuring RHEL subscription compliance.

Customers who choose either Red Hat Satellite Server or Red Hat Subscription Manager should remove the RHUI configuration and then attach a cloud-access-enabled RHEL subscription to Azure Hybrid Benefit for PAYG RHEL virtual machines.

For more information about Red Hat subscription compliance, software updates, and sources for Azure Hybrid Benefit for pay-as-you-go RHEL virtual machines, see the [Red Hat article about using RHEL subscriptions with Azure Hybrid Benefit](https://access.redhat.com/articles/5419341).

Customers who use Azure Hybrid Benefit BYOS to PAYG capability for RHEL agree to the standard [legal terms](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Cloud_Software_Subscription_Agreement_for_Microsoft_Azure.pdf) and [privacy statement](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Privacy_Statement_for_Microsoft_Azure.pdf) associated with the Azure Marketplace RHEL offerings.

### [SUSE compliance](#tab/slescompliance)

To use Azure Hybrid Benefit for pay-as-you-go SLES virtual machines, and to get information about moving from SLES pay-as-you-go to BYOS or moving from SLES BYOS to pay-as-you-go, see [SUSE Linux Enterprise and Azure Hybrid Benefit](https://aka.ms/suse-ahb).

Customers who use Azure Hybrid Benefit for pay-as-you-go SLES virtual machines need to move the cloud update infrastructure to one of three options that provide software updates and patches to those virtual machines:

- [SUSE Customer Center](https://scc.suse.com)
- SUSE Manager
- SUSE Repository Mirroring Tool

If you use Azure Hybrid Benefit BYOS to PAYG capability for SLES and want more information about moving from SLES pay-as-you-go to BYOS, or moving from SLES BYOS to pay-as-you-go, see [Azure Hybrid Benefit Support](https://aka.ms/suse-ahb) on the SUSE website.

---

## Frequently asked questions

- **Q: Can I use a license type of RHEL_BYOS with a SLES image, or vice versa?**

    - A: No, you can't. Trying to enter a license type that incorrectly matches the distribution running on your virtual machine won't update any billing metadata. But if you accidentally enter the wrong license type, updating your virtual machine again to the correct license type still enables Azure Hybrid Benefit.

- **Q: I've registered with Red Hat Cloud Access but still can't enable Azure Hybrid Benefit on my RHEL virtual machines. What should I do?**

    - A: It might take some time for your Red Hat Cloud Access subscription registration to propagate from Red Hat to Azure. If you still see the error after one business day, contact Microsoft support.

- **Q: I've deployed a virtual machine by using a RHEL BYOS "golden image." Can I convert the billing on this image from BYOS to pay-as-you-go?**

    - A: Yes, you can use Azure Hybrid Benefit for BYOS virtual machines to do this. Learn more about this capability.

- **Q: I've uploaded my own RHEL or SLES image from on-premises (via Azure Migrate, Azure Site Recovery, or otherwise) to Azure. Can I convert the billing on these images from BYOS to pay-as-you-go?**

    - A: Yes, you can use Azure Hybrid Benefit for BYOS virtual machines to do this. Learn more about this capability.

- **Q: I've uploaded my own RHEL or SLES image from on-premises (via Azure Migrate, Azure Site Recovery, or otherwise) to Azure. Do I need to do anything to benefit from Azure Hybrid Benefit?**

    - A: No, you don't. RHEL or SLES images that you upload are already considered BYOS, and you're charged only for Azure infrastructure costs. You're responsible for RHEL subscription costs, just as you are for your on-premises environments.

- **Q: Can I use Azure Hybrid Benefit for pay-as-you-go virtual machines for Azure Marketplace RHEL and SLES SAP images?**

    - A: Yes. You can use the license type of RHEL_BYOS for RHEL virtual machines and SLES_BYOS for conversions of virtual machines deployed from Azure Marketplace RHEL and SLES SAP images.

- **Q: Can I use Azure Hybrid Benefit for pay-as-you-go virtual machines on Virtual Machine Scale Sets for RHEL and SLES?**

    - A: Yes. Azure Hybrid Benefit on Virtual Machine Scale Sets for RHEL and SLES is available to all users. Learn more about this benefit and how to use it.

- **Q: Can I use Azure Hybrid Benefit for pay-as-you-go virtual machines on reserved instances for RHEL and SLES?**

    - A: Yes. Azure Hybrid Benefit for pay-as-you-go virtual machines on reserved instances for RHEL and SLES is available to all users.

- **Q: Can I use Azure Hybrid Benefit for pay-as-you-go virtual machines on a virtual machine deployed for SQL Server on RHEL images?**

    - A: No, you can't. There's no plan for supporting these virtual machines.

- **Q: Can I use Azure Hybrid Benefit on my RHEL for Virtual Datacenters subscription?**

    - A: No. RHEL for Virtual Datacenters isn't supported on Azure at all, including Azure Hybrid Benefit.

## Next steps

* [Learn how to create and update virtual machines and add license types (RHEL_BYOS, SLES_BYOS) for Azure Hybrid Benefit by using the Azure CLI](/cli/azure/vm)
* [Learn about Azure Hybrid Benefit on Virtual Machine Scale Sets for RHEL and SLES and how to use it](../../virtual-machine-scale-sets/azure-hybrid-benefit-linux.md)
