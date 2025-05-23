---
title: Trusted Launch for Virtual Machines in Azure DevTest Labs
description: Learn how to use Trusted Launch for Generation 2 virtual machines (VMs) in Azure DevTest Labs.
ms.topic: how-to
ms.author: anishtrakru
author: RoseHJM
ms.date: 02/13/2025
ms.custom: UpdateFrequency2
---

# Trusted Launch for Generation 2 VMs in Azure DevTest Labs

Trusted Launch provides a seamless solution to enhance the security of Generation 2 (Gen 2) virtual machines (VMs) by protecting against advanced and persistent attack techniques. This feature is composed of several coordinated infrastructure technologies that can be enabled independently, each adding an additional layer of defense against sophisticated threats. With Trusted Launch, you can securely deploy VMs with verified boot loaders, operating system (OS) kernels, and drivers, as well as protect keys, certificates, and secrets within the VMs. Additionally, it offers insights and confidence in the integrity of the entire boot chain, ensuring that workloads are trusted and verifiable.

For more information about Trusted Launch, see [Trusted Launch for Azure VMs](/azure/virtual-machines/trusted-launch).

This article explains how to use Trusted Launch for Gen 2 VMs in Azure DevTest Labs.

> [!IMPORTANT]
> **Trusted Launch** for Generation 2 VMs is currently in preview in Azure DevTest Labs. For more information about the preview status, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). The document defines legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability.

## Create lab virtual machines with Trusted Launch for Generation 2 VMs

### Prerequisite

You need at least [user](devtest-lab-add-devtest-user.md#devtest-labs-user) access to a lab in DevTest Labs. For more information about creating labs, see [Create a lab in the Azure portal](devtest-lab-create-lab.md).

### Create a Gen 2 VM with Trusted Launch

1. In the [Azure portal](https://portal.azure.com), go to the **Overview** page for the lab.

1. On the lab **Overview** page, select **Add**.

   :::image type="content" source="./media/devtest-lab-add-vm/portal-lab-add-vm.png" alt-text="Screenshot of lab overview page showing add button." lightbox="./media/devtest-lab-add-vm/portal-lab-add-vm.png":::

1. On the **Choose a base** page, select a Generation 2 image for the VM. The **Generation** column in the list of images displays whether it is a Generation 1 or Generation 2 image.

    :::image type="content" source="./media/devtest-lab-gen2-vm/dev-test-lab-gen-2-images.png" alt-text="Screenshot of list of available base images."  lightbox="./media/devtest-lab-gen2-vm/dev-test-lab-gen-2-images.png":::

1. On the **Basics Settings** tab of the **Create lab resource** screen, provide the following information:

   - **Virtual machine name**: Keep the autogenerated name, or enter another unique VM name.
   - **User name**: Keep the user name, or enter another user name to grant administrator privileges on the VM.
   - **Use a saved secret**: Select this checkbox to use a secret from Azure Key Vault instead of a password to access the VM. If you select this option, under **Secret**, select the secret to use from the dropdown list. For more information, see [Store secrets in a key vault](devtest-lab-store-secrets-in-key-vault.md). 
   - **Password**: If you don't choose to use a secret, enter a VM password between 8 and 123 characters long.
   - **Save as default password**: Select this checkbox to save the password in the Key Vault associated with the lab.
   - **Virtual machine size**: Keep the default value for the base, or select **Change Size** to select different sizes.
   - **Allow hibernation**: Select this option to enable hibernation for this virtual machine. If you enable Hibernation, you also must select **Public IP** in the Advanced settings as Private and Shared IP are currently not supported if Hibernation is enabled.
   - **OS disk type**: Keep the default value for the base, or select a different option from the dropdown list.
   - **Security type**: Select **Trusted Launch** to enable it for Generation 2 VMs. On selecting Trusted Launch, the options Secure boot, vTPM, and Integrity Monitoring will appear. Based on your needs, select the appropriate options among them for your deployment. For more information, see [Trusted Launch-enabled security features](/azure/virtual-machines/trusted-launch#secure-boot).
   - **Artifacts**: This field shows the number of artifacts already configured for this VM base. Optionally, select **Add or Remove Artifacts** to select and configure artifacts to add to the VM.

   :::image type="content" source="./media/devtest-lab-add-vm/portal-lab-vm-basic-settings.png" alt-text="Screenshot of virtual machine basic settings page." lightbox="./media/devtest-lab-add-vm/portal-lab-vm-basic-settings.png":::

1. After you configure all settings, on the **Basic Settings** tab of the **Create lab resource** screen, select **Create** to deploy the VM.

During VM deployment, you can select the **Notifications** icon at the top of the screen to see progress. Creating a VM takes a while.

When the deployment is complete, if you kept yourself as VM owner, the VM appears under **My virtual machines** on the lab **Overview** page. To connect to the VM, select it from the list, and then select **Connect** on the VM's **Overview** page. If the VM is stopped, select **Start** first to start the VM.