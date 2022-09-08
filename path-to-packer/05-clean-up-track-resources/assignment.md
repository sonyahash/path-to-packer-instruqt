---
slug: clean-up-track-resources
id: njnka9c13hu5
type: challenge
title: "4- \U0001F6AE Clean up track resources"
teaser: Steps to clean up our environment.
notes:
- type: text
  contents: |
    <p align="center"> Once you are done using your image, you need to clean up your environment and destroy your infrastructure. </p>

    <p align="center">
      <img width="660" height="500" src="../assets/Andy-as-janitor.gif">
    </p>
tabs:
- title: Terminal
  type: terminal
  hostname: cloud-client
- title: Credentials
  type: service
  hostname: cloud-client
  path: /
  port: 80
difficulty: basic
timelimit: 630
---
Here are the steps to clean up our environment. These steps can be replicated outside of the Instruqt environment. Please clean up afterwards to ensure no sensitive item is left behind. This section shows how to:

- Remove your HCP Bucket and associated Iterations and Channels
- Remove your TFC Workspace
- Remove any EC2 instances and AMIs under a specific account (be careful here)
- Remove any instances in Azure

Step by step
============

**STEP 1:** Destroy they infrastructure you have built in AWS

```bash
cd $ASSETS_HOME/terraform/production
terraform destroy -auto-approve
```

----------------------

**STEP 2:** Type the following command to find the Azure credentials needed for your Packer and Terraform variable files:
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

----------------------

**STEP 3:** Destroy the infrastructure you have built in Azure

```bash
cd $ASSETS_HOME/terraform/azure
terraform destroy -auto-approve
```

----------------------

**STEP 4:** Use the credentials we obtained from the user
```bash
source $APP_HOME/hcp_credentials
```

----------------------

**STEP 5:** Use the API shortcuts for convenience
```bash
source $ASSETS_HOME/api/shortcuts.bash
```

----------------------

**STEP 6:** Clean up HCP Registry and remove bucket **"path-to-packer"**
```bash
curl -s -H "Authorization: Bearer ${HCP_CLIENT_TOKEN}" \
  -X DELETE $HCP_PACKER_DELETE_BUCKET
```

----------------------

**STEP 7:** Remove **path-to-packer** TFC Workspace
```bash
curl -s -k \
  --header "Authorization: Bearer $TFE_TOKEN" \
  --header "Content-Type: application/vnd.api+json" \
  --request DELETE \
  $TFC_API_DELETE_WORKSPACE
```

----------------------

**STEP 8:** Remove **path-to-packer-azure** TFC Workspace
```bash
curl -s -k \
  --header "Authorization: Bearer $TFE_TOKEN" \
  --header "Content-Type: application/vnd.api+json" \
  --request DELETE \
  $TFC_API_DELETE_WORKSPACE2
```