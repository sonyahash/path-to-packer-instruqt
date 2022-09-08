---
slug: codify-your-runbook
id: setnfwolz8sz
type: challenge
title: "1- \U0001F4DD Codify your runbook"
teaser: Learn how to codify the runbook that has been given. Set up each block of
  a Packer template.
notes:
- type: text
  contents: |-
    <p align="center">You have Dunder Mifflin's runbook and need to implement it using HashiCorp Packer. You need to hurry, be like Dwight! No time for coffee breaks!</p>

    <p align="center">
      <img width="660" height="500" src="../assets/dwight-coffee-sprint.gif">
    </p>
tabs:
- title: Explainer
  type: service
  hostname: cloud-client
  path: /uc-01-challenge
  port: 5000
- title: Runbook
  type: service
  hostname: cloud-client
  path: /uc-01-section-01
  port: 5000
- title: Packer
  type: code
  hostname: cloud-client
  path: /root/assets/packer/development
- title: Terraform
  type: code
  hostname: cloud-client
  path: /root/assets/terraform/production
- title: Terminal
  type: terminal
  hostname: cloud-client
difficulty: basic
timelimit: 1980
---

# Goals:
<p>
<li>Turn Dunder Mifflin's runbook into a Packer template</li>
<li>Adding code to the <span style="color:#02A8EF"><b><i>ubuntu_base.pkr.hcl</b></i></span> template</li>
<li>Understand how a Packer file is formatted and the mapping between the runbook and the code.</li>
</p>

# 1 - Converting your runbook to a Packer template
======================================

Use the Dunder Mifflin runbook to build a Packer template and associate the new build in the HCP Packer Registry. This runbook can be found under the ```Runbook``` tab.

<p>You will be given blocks of code to insert into the <span style="color:#02A8EF"><b><i>ubuntu_base.pkr.hcl</b></i></span> file to create your Packer template.</p>

<p><span style="color:orange"><b>NOTE:</b></span> You will often see <code>var.{VARIABLE_NAME}</code>. This indicates that you are referencing a variable defined in the <span style="color:#02A8EF"><b><i>variables.pkr.hcl</b></i></span> file.</p>

# 2 - Defining your Data Source block
====================================
<p>Under the <code>Packer</code> tab, edit the template file <span style="color:#02A8EF"><b><i>ubuntu_base.pkr.hcl</b></i></span> and include the following code stanza (start on line 9).</p>

```bash
data "amazon-ami" "ubuntu-server-east" {
  region = var.region
  filters = {
    name                = var.image_name
    root-device-type    = "ebs"
    virtualization-type = "hvm"
  }
  most_recent = true
  owners      = ["099720109477"]
}
```
A data block will request Packer to read from a given data source (```"amazon-ami"```) and export the results under the given local name (```"ubuntu-server-east"```).

Adding this chunk of code chooses the region you wish to set up your instance in.

# 3 - Define your Source block
=============================
<p>In the same template file, <span style="color:#02A8EF"><b><i>ubuntu_base.pkr.hcl</b></i></span>, insert the following code snippet on line 19:</p>

```bash
source "amazon-ebs" "ubuntu-server-east" {
  region         = var.region
  source_ami     = data.amazon-ami.ubuntu-server-east.id
  instance_type  = "t2.small"
  ssh_username   = "ubuntu"
  ssh_agent_auth = false
  ami_name       = "path-to-packer{{timestamp}}_v${var.version}"
  tags           = var.aws_tags
}
```
This block of code defines your builder type (```"amazon-ebs"```) and the unique name or identifier you want to give the source (```"ubuntu-server-east"```).

<p>Looking at the <code>Runbook</code> tab, adding this block of code correlates to <b><i>Create a new instance by using the Amazon EC2</i></b> under <b><i>Creating an Ubuntu Server in AWS.</i></b> You can now tell that this block applies the settings you wish your instance to have.</p>

<p>Setting the <code>ssh_username</code> to <code style="color:#F9966B">"ubuntu"</code> creates a user in your Ubuntu server. Notice that this maps to <b><i>Add a New User and Director</i></b> in the runbook.

# 4 - Add to your build block
======================================

4.1 Associate your build with the HCP Packer Registry. You can do so by adding this block of code to the ```build``` block:

```bash
hcp_packer_registry {
    bucket_name   = "path-to-packer-aws"
    description   = "Path to Packer Demo"
    bucket_labels = var.aws_tags
    build_labels = {
      "build-time"   = timestamp(),
      "build-source" = basename(path.cwd)
    }
  }
```

The ```bucket_name``` will help you identify the image that shows up in your HCP Packer Registry.

By adding the ```hcp_packer_registry``` block, you are defining the image you would like to store in your HCP Packer account.

----------------------------------------

4.2 Continue adding to the ```build``` block by inserting this snippet of code:

```bash
sources = [
    "source.amazon-ebs.ubuntu-server-east"
  ]
```

This tells Packer to build the instance you have defined in your ```source``` block you created in #3

-------------------------------------

4.3 Add your provisioners (continuing in the ```build``` block):

A ```file``` provisioner uploads files to the machine built by Packer, then the ```shell``` provisioner will move those files to the proper place, set permissions, etc.

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

  provisioner "shell" {
    inline = [
      "sleep 30",
      "sudo apt-get update",
      "sudo apt-get upgrade -y",
      "sudo apt-get install -y nginx",
      "sudo systemctl start nginx",
      "sudo apt-get install tree"
    ]
  }
```

Looking in the ```Runbook``` tab, notice that the above ```shell``` provisioner is downloading and configuring Nginx.

You have now successfully created your Packer template file!
<body><font color="yellow">Make sure to click the save button to keep your work!</body>