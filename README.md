# Tailscale in Codespaces | Private Azure Storage Account access

This repository provides a straightforward approach to connect GitHub Codespaces to a private Azure virtual network and access your Azure storage container via private endpoint using Tailscale. The end goal is to sync your Codespaces file system with that of your private storage account container.

## Prerequisites

- Set up a Tailscale account.
- Have access to create resources in the azure portal.
- Ability to create a codespace using this repositories [devcontainer.json](./.devcontainer/devcontainer.json), which will include the `tailscale` and `azcopy` cli tools.

## Connect Codespaces to an Azure Private Network and Access a Storage Account using a Private Endpoint

### Step 1: Create Resources in Azure

1. **Create a Virtual Network**

   - Navigate to the Azure portal and create a new virtual network.
2. **Create a Storage Account**

   - In the Azure portal, create a new storage account.
   - Assign yourself the `Storage Blob Data Contributor` role on the storage account.
   - Create a storage container and add some data to it.
   - Create a private endpoint for the storage account.
   - Set the storage account's public network access to "Disabled".
3. **Create a Virtual Machine**

   - Create an Ubuntu 22.04 virtual machine (VM) in Azure and set it up as a Tailscale machine in your tailnet.
     - Save your VM key file locally. We will later copy it into the `keys` directory in the Codespace file system.
   - If you do not want your VM exposed with a public IP, set up a bastion to connect to it before setting up the Tailscale client.
   - Connect to your VM.
   - Follow the guide to set up the Tailscale client on an Azure VM: [Setup Tailscale Client on Azure VM](https://tailscale.com/kb/1142/cloud-azure-linux)
     - If not exposing your VM with a public IP, you can ignore adding the inbound security rules.
     - Advertise the routes of your private endpoint subnet. For example, if your subnet has a block of `10.0.0.0/24`, you would run: `sudo tailscale up --advertise-routes=10.0.0.0/24,168.63.129.16/32 --accept-dns=false`
     - Approve your subnet routes from your Azure VM tailnet machine in the Tailscale admin console.
     - **Important: In Step 3: Add Azure DNS for your tailnet, specify the blob endpoint of your storage account (`<storage_account_name>.blob.core.windows.net`) as the search domain for the `168.63.129.16` nameserver.**
   - Configure the VM to allow IP forwarding by following this guide: [Advertise Subnet Routes](https://tailscale.com/kb/1019/subnets?tab=linux#advertise-subnet-routes)

### Step 2: Connect Codespaces to Your Tailnet and SSH into VM

- Create a codespace using the [devcontainer.json](./.devcontainer/devcontainer.json) in this repository.
- In your codespace, set up your client with:
  `sudo tailscale up --accept-routes`
- Copy your VM private key into the `keys` directory in your codespace's file system.
- Connect to your new tailnet Azure VM using the Tailscale machine name or the Tailscale address:
  `ssh -i ./keys/<vm_key_file>.pem azureuser@<tailscale_machine_name_or_machine_address>`
  - If having permission issues, ensure correct permissions on your key file:
    `chmod 600 ./keys/<vm_key_file>.pem`
  - **Note: If your VM is not exposing a public IP, it is possible the connection between your VM and codespaces was not correctly established. If you are having trouble connecting from codespaces to your VM, login to your VM and send a ping (`tailscale ping <codespace-address>`) to correctly establish the connection.**
- While connected, verify the storage endpoint resolves to its private IP:
  `curl -v <storage_account_name>.blob.core.windows.net`
- Disconnect from the VM and return to your codespace.

### Step 3: Sync Your Codespaces File System with Your Private Storage Container

- In your codespace, verify the URL resolves to the private endpoint:
  `curl -v <storage_account_name>.blob.core.windows.net`
- Test the `azcopy sync` command to sync your container files to your codespace:
  `azcopy login`
  `azcopy sync "https://<storage_account_name>.blob.core.windows.net/<storage_container_name>/" "./" --recursive`
    - *This may fail if your codespace runs out of memory. If so, use a bigger codespace or less data.*
- Congrats, you're done