---
slug: convert-your-work-to-azure
id: f1ohv4xefhem
type: challenge
title: 3- ☁️ Convert your work to Azure
teaser: Deploy Terraform and Packer with your image using a new cloud service, Azure.
notes:
- type: text
  contents: |-
    <p align="center">Breaking news! Dunder Mifflin just got acquired by the printer company Sabre.</p>

    <p align="center">
      <img width="660" height="500" src="../assets/Sabre.gif">
    </p>

    <p align="center"> After being one of the few surviving branches of Dunder Mifflin, the Scranton Branch is now having to accept all the new policies from Sabre. That includes everything from the company culture to the IT infrastructure. </p>
- type: text
  contents: |
    <p align="center">Oh no, Sabre uses Azure instead of AWS as their cloud platform. You’re now being forced to convert all of your previous work to Azure. Luckily this can be done easily with Packer!</p>

    <p align="center">
      <img width="560" height="400" src="../assets/remain-calm.gif">
    </p>
tabs:
- title: Explainer
  type: service
  hostname: cloud-client
  path: /uc-03-challenge
  port: 5000
- title: Packer
  type: code
  hostname: cloud-client
  path: /root/assets/packer/production
- title: Terraform
  type: code
  hostname: cloud-client
  path: /root/assets/terraform/azure
- title: Terminal
  type: terminal
  hostname: cloud-client
- title: Credentials
  type: service
  hostname: cloud-client
  path: /
  port: 80
difficulty: basic
timelimit: 1980
---

# Goals:

- Convert your Packer template used to deploy in AWS to a template you can deploy in Azure. **You must do so because variable names and syntax differ between the two cloud providers.**
- Build your image from the Azure Packer template
- Deploy your image using TFC in Azure

#1 - Convert from AWS to Azure
======================================
<p>Here you will need to add an Azure builder to your Packer file. Make the following changes in <span style="color:#02A8EF"><b>ubuntu_base.pkr.hcl</b></span> under the <code>Packer</code> tab.

1.1 In your `packer` block, you need to add the Azure plugin. Do so by adding the following code snippet under your Amazon plugin:

```bash
azure = {
      version = ">= 1.0.3"
      source  = "github.com/hashicorp/azure"
    }
```

--------------------------

<p>1.2 Notice that you have a <code style="color:#66CDAA">data</code> and a <code>source</code> block for AWS, we need to add a <code style="color:#66CDAA">source</code> block for Azure now. You can add the following code block under the Amazon <code>source</code> block:

```bash
source "azure-arm" "ubuntu" {
  azure_tags = {
    dept = "Engineering"
    task = "Image deployment"
  }
  subscription_id                   = var.subscription_id
  client_id                         = var.client_id
  client_secret                     = var.client_secret
  tenant_id                         = var.tenant_id
  image_offer                       = "UbuntuServer"
  image_publisher                   = "Canonical"
  image_sku                         = "16.04-LTS"
  location                          = var.location
  managed_image_name                = "myPackerImage"
  managed_image_resource_group_name = "path-to-packer"
  os_type                           = "Linux"
  vm_size                           = var.vm_size
}
```

-----------------------------

<p>1.3 In your <code>build</code> block notice that you are sourcing your Amazon instance in the <code style="color:#5CB3FF">sources</code> block. Change that to source your Azure instance instead.
Replace <code style="color:#F9966B">"source.amazon-ebs.ubuntu-server-east"</code> with: </p>

```bash
"source.azure-arm.ubuntu"
```

-------------------------

1.4 Remove the following block of code, it is not necessary when using Azure:

```bash
# Add startup script that will run path to packer on instance boot
  provisioner "file" {
    source      = "../production/setup-deps-path-to-packer.sh"
    destination = "/tmp/setup-deps-path-to-packer.sh"
  }

  # Move temp files to actual destination
  # Must use this method because their destinations are protected
  provisioner "shell" {
    inline = [
      "sudo cp /tmp/setup-deps-path-to-packer.sh /var/lib/cloud/scripts/per-boot/setup-deps-path-to-packer.sh",
    ]
  }
```

<p><span style="color:yellow">Remember to save your work before moving on!</span></p>

#2 - Create and connect to your Azure account
===========================================

2.1 In the `Terminal` tab, login to Azure with the following command. The Azure subscription and enviroment have already been created through Instruqt.
```bash
az login \
--username=$INSTRUQT_AZURE_SUBSCRIPTION_AZURESUBSCRIPTION_USERNAME \
--password=$INSTRUQT_AZURE_SUBSCRIPTION_AZURESUBSCRIPTION_PASSWORD
```
The output will show your credntials.

2.2 Create a Resource Group:
```bash
az group create -n path-to-packer -l centralus
```
This will create a Resource Group named ***path-to-packer*** and will be in the `Central US` region.

#3 - Edit your Terraform and Packer files
============================================
<p>3.1 In the <code>Terraform</code> tab, open the <span style="color:#D891EF"><b><i>main.tf</i></b></span> file. On line 10, change the <code style="color:#F9966B">ORG_NAME</code> to your TFC organization name.</p>
<p><span style="color:yellow">Remember to save your work before moving on!</span></p>

---------------------------------

3.2 In the ```Terminal``` tab, type the following command to find the Azure credentials needed for your Packer and Terraform variable files:
```bash
export PKR_VAR_client_id=$ARM_CLIENT_ID
export PKR_VAR_client_secret=$ARM_CLIENT_SECRET
export PKR_VAR_tenant_id=$ARM_TENANT_ID
export PKR_VAR_subscription_id=$ARM_SUBSCRIPTION_ID

export TF_VAR_client_id=$ARM_CLIENT_ID
export TF_VAR_client_secret=$ARM_CLIENT_SECRET
export TF_VAR_tenant_id=$ARM_TENANT_ID
export TF_VAR_subscription_id=$ARM_SUBSCRIPTION_ID
```
For the prefix variable, replace `<REPLACE ME>` with your name:
```bash
export TF_VAR_prefix=<REPLACE_ME>
```

#4 - Build your Packer image on Azure
=====================
4.1 In the `Terminal` tab, create a new unique fingerprint for your new Azure image.
<p><span style="color:orange"><b>RECALL:</b></span> every image must have a unique fingerprint for Packer to execute properly!</p>

```bash
# HCP Packer requires a unique ID associated with a new image
export HCP_PACKER_BUILD_FINGERPRINT="path-to-packer-azure-$(date +%s)"
```

------------------------------

4.2 Build your Azure image

```bash
# Build the new image
cd $ASSETS_HOME/packer/production
packer init .
packer fmt .
packer validate .
packer build .
```

------------------------------

4.3 Get HCP Packer API endpoints
```bash
source $ASSETS_HOME/api/shortcuts.bash
```

------------------------------

4.4 Call the HCP Packer API and get details of the HCP Packer Registry:
```bash
curl -s -H   "Authorization: Bearer ${HCP_CLIENT_TOKEN}"   -X GET $HCP_PACKER_API_GET_REGISTRY | jq
```

------------------------------

4.5 Create an HCP Channel in your Azure image
```bash
curl -s -H "Authorization: Bearer ${HCP_CLIENT_TOKEN}" \
    -H "Content-Type: application/json" \
    -d '{"slug" : "'"${HCP_PACKER_CHANNEL_SLUG2}"'", "incremental_version": 1}' \
    -X POST $HCP_PACKER_API_CREATE_CHANNEL2 \
    | jq
```

<p><span style="color:yellow"><b>DON'T FORGET:</b></span><i> After creating your channel, go into your HCP account. Click on the</i> <b>Image ID</b>. <i>On the left side panel, go into the Channels tab. Make sure that your</i> <code>path-to-packer-azure-channel</code> <i>uses the latest iteration. Do so by clicking on the three dots, then</i> <b>"Change assigned iteration"</b>. <i>This is where you can update the iteration you are using on that specific channel!</i></span></p>

------------------------------

4.6 Confirm the HCP Packer Registry entry for the latest iteration of this machine image
```bash
curl -s -H "Authorization: Bearer ${HCP_CLIENT_TOKEN}" \
    -X GET $HCP_PACKER_API_GET_BUCKET2 \
    | jq '.bucket.latest_iteration'
```

#5 - Use Terraform to deploy your image in Azure
==========================

5.1 Deploy your image with Terraform
```bash
cd $ASSETS_HOME/terraform/azure
terraform init
terraform apply -auto-approve
```