---
slug: put-image-in-hcp-packer-deploy-tfc
id: ukkm5flyi1tv
type: challenge
title: "2- \U0001F4F8 Put image in HCP Packer and deploy with TFC"
teaser: Make a commit to a repository with HashiCorp Terraform code that uses an Amazon
  Machine Image (AMI) reference to build a new Amazon EC2 resource.
notes:
- type: text
  contents: |-
    <p align="center">
    Now it is time to deploy images using HashiCorp Terraform from our HCP Packer registry!
    Let's create snapshots with our golden image. </p>

    <p align="center">
      <img width="660" height="500" src="https://play.instruqt.com/assets/tracks/vd9omqsavijj/ce917743a87172b2f10a10dc5d960537/assets/click.gif"
      </p>
tabs:
- title: Explainer
  type: service
  hostname: cloud-client
  path: /uc-02-challenge
  port: 5000
- title: Packer
  type: code
  hostname: cloud-client
  path: /root/assets/packer/production
- title: Terraform
  type: code
  hostname: cloud-client
  path: /root/assets/terraform/production
- title: Terminal
  type: terminal
  hostname: cloud-client
- title: Cloud Credentials
  type: service
  hostname: cloud-client
  path: /
  port: 80
difficulty: basic
timelimit: 1980
---

# Goals:
<p>
<li>Build an Amazon Machine Image (AMI) using Packer</li>
<li>Recognize that without Packer, cloud practitioners build, track, and maintain their machine images manually</li>
<li>With Packer and HCP Packer, cloud practitioners use a central registry to build, track and maintain the lifecycle of their machine images with automation</li>
</p>

# 1- Build a machine image with Packer
================================
1.1 Create a unique fingerprint to associate your new machine image in the HCP Packer Registry in the `Terminal` tab:
```bash
# HCP Packer requires a unique ID associated with a new image
export HCP_PACKER_BUILD_FINGERPRINT="path-to-packer-aws-$(date +%s)"
```

The purpose of the ```HCP_PACKER_BUILD_FINGERPRINT``` is to identify the build iteration for your image in the HCP Packer registry.

1.2 Now, build your new image!

```bash
# Build the new image
cd $ASSETS_HOME/packer/production
packer init .
packer fmt .
packer validate .
packer build .
```

<p><span style="color:orange"><b>NOTE:</b></span> After building your image, you will notice that each build iteration of your image will have a specific fingerprint. If you use the same fingerprint, Packer will exit without running the build.</p>

# 2 - Create an HCP Packer Channel
=====================================
This channel will support the distribution control for the machine image.

```bash
source $ASSETS_HOME/api/shortcuts.bash

curl -s -H "Authorization: Bearer ${HCP_CLIENT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"slug" : "'"${HCP_PACKER_CHANNEL_SLUG}"'", "incremental_version": 1}' \
  -X POST $HCP_PACKER_API_CREATE_CHANNEL \
  | jq
```

<p><span style="color:orange"><b>NOTE:</b></span> By creating a channel, you can reference a specific build iteration from your HCP Packer registry in Packer or Terraform. This will be helpful when building your images then deploying them with Terraform!</p>

# 3 - Edit your TFC Organization and Workspace names
=======================================================
<p>In the <code>Terraform</code> tab, open <span style="color:#D891EF"><b><i>main.tf</i></b></span>. In the <code>cloud</code> block notice there are variables for an organization and a workspace name.

<p>Change the <code style="color:#F9966B">ORG_NAME</code> to match your organization name that you created earlier in the track. Keep the workspace name set to <code>"path-to-packer"</code>.</p>
<p><span style="color:yellow">Make sure to save your work by clicking the save button!</span></p>

# 4 - Deploy with the AWS EC2 CLI
================================

4.1 In the `Terminal` tab, use the AWS EC2 CLI to confirm that we have a new machine image:

```bash
aws ec2 describe-images \
  --owners $AWS_ACCOUNT_ID
```

4.2 Set the AMI ID variable to create a new EC2 instance

```bash
# Obtain the AWS AMI ID to create a new EC2 instance
export IMAGE_ID=$(aws ec2 describe-images \
  --owners $AWS_ACCOUNT_ID \
  | jq -r '.[] | .[] | .ImageId')
```

# 5 - Automate with HCP Packer
=================================================

In the `Terminal` tab, confirm the HCP Packer Registry entry for the latest iteration of this machine image.

```bash
curl -s -H "Authorization: Bearer ${HCP_CLIENT_TOKEN}" \
  -X GET $HCP_PACKER_API_GET_BUCKET \
| jq '.bucket.latest_iteration'
```


# 6 - Deploy a new machine image using Terraform
==================================================
6.1 Login to your Terraform Cloud account. Do so with the following commands:

```bash
cd $ASSETS_HOME/terraform/production
terraform login
```

Follow along with the prompts on the screen. Make sure to use the same API token you used when inputting your Terraform credentials!

--------------------------------------
<p><span style="color:orange"><b>If you forgot your API token value, change directories using</b></span></p>

```bash
cd ~/www
cat hcp_credentials
```

<p><span style="color:orange"><b>to find your </b><code>TFE_TOKEN</code><b>, it is the second to last value that is output!</span></b></p>

---------------------------------------------

6.2 Initialize Terraform and run an apply!

```bash
terraform init
terraform apply -auto-approve
```
To see your website, click on the `your_app_url` address given in the output.