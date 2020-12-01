---
title: "Terraform Service for VMC Automation"
date: 2020/11/23 09:00
anchor: "terraformservicevmcsample"
---
This example and associated artifacts will allow you to leverage vRA Terraform Service for creating a VMware on AWS' SDDC Enviroment with an additional cluster resources
The following Artifacts are used and/or genrated:

+ Terraform File: main.tf 
  - Standard Terraform HCL code leveraging Terraform VMC provider.
+ Terraform File: variables.tf 
  - Standard Terraform Variable Definition.
+ vRA Terraform Cloud Template: CloudTemplate_VMC  
  - vRA Cloud Template "Front Ends" and Enhances the Terraform Configuration Files by using vRA Standard Capabilities.

This Terraform configuration file _main.tf_  
Please note that the vCenter URL is returned via _output_ instructions defined at the botton:

```python
---
provider "vmc" {
refresh_token = var.api_token
org_id = var.org_id
}

data "vmc_connected_accounts" "my_accounts" {
account_number = var.aws_account_number
}

data "vmc_customer_subnets" "my_subnets" {
connected_account_id = data.vmc_connected_accounts.my_accounts.id
region               = replace(upper(var.sddc_region), "-", "_")
}

resource "vmc_sddc" "sddc_terra" {
sddc_name           = var.sddc_name
vpc_cidr            = var.vpc_cidr
num_host            = var.sddc_num_hosts
provider_type       = var.provider_type
region              = data.vmc_customer_subnets.my_subnets.region
vxlan_subnet        = var.vxlan_subnet
delay_account_link  = false
skip_creating_vxlan = false
sso_domain          = "vmc.local"
deployment_type = "SingleAZ"

host_instance_type = var.host_instance_type

#  storage_capacity = var.storage_capacity

account_link_sddc_config {
    customer_subnet_ids  = [data.vmc_customer_subnets.my_subnets.ids[0]]
    connected_account_id = data.vmc_connected_accounts.my_accounts.id
}
timeouts {
    create = "300m"
    update = "300m"
    delete = "180m"
}
}

resource "vmc_cluster" "cluster_1" {
sddc_id = vmc_sddc.sddc_terra.id
num_hosts = var.cluster_num_hosts
}

output "vc_url" { value = "${vmc_sddc.sddc_terra.vc_url}" }
```


> **############Several Variables are used in the code above which are defined at _variables.tf_  as displayed below######**



```python
---
variable "api_token" {
description = "API token used to authenticate when calling the VMware Cloud Services API."
default = ""
}

variable "org_id" {
description = "Organization Identifier."
default = ""
}

variable "aws_account_number" {
description = "The AWS account number."
default     = ""
}

variable "sddc_name"{
description = "Name of SDDC."
default = "Terraform-SDDC-Bis"
}

variable "sddc_region" {
description = "The AWS  or VMC specific region."
default     = "US_WEST_1"
}

variable "vpc_cidr" {
description = "SDDC management network CIDR. Only prefix of 16, 20 and 23 are supported."
default     = "172.31.48.0/20"
}

variable "vxlan_subnet" {
description = "A logical network segment that will be created with the SDDC under the compute gateway."
default     = "192.168.1.0/24"
}

variable provider_type {
description = "Determines what additional properties are available based on cloud provider. Default value : AWS"
default     = "ZEROCLOUD"
}

variable host_instance_type {
description = "The instance type for the ESX hosts in the primary cluster of the SDDC. Possible values: I3_METAL, R5_METAL."
default     = "I3_METAL"
}

variable storage_capacity {
description = "The storage capacity value to be requested for the SDDC primary cluster. This variable is only for R5.METAL. Possible values are 15TB, 20TB, 25TB, 30TB, 35TB per host."
default     = "15TB"
}

variable cluster_num_hosts {
description = "The number of hosts in the cluster."
default = 3
}

variable sddc_num_hosts {
description = "The number of hosts in SDDC."
default     = 3
}
```

By Following the same instructions as presented at [Terraform Service in vRA](https://blogs.vmware.com/management/2020/09/terraform-service-in-vra.html) 
but feeding the appropiate files presented here you could generate a Cloud Template similar to:

```yaml
---
inputs:
api_token:
    type: string
org_id:
    type: string
aws_account_number:
    type: string
sddc_name:
    type: string
    default: Terraform-SDDC
sddc_region:
    type: string
    default: US_WEST_1
vpc_cidr:
    type: string
    default: 172.31.48.0/20
vxlan_subnet:
    type: string
    default: 192.168.1.0/24
resources:
terraform:
    type: Cloud.Terraform.Configuration
    properties:
    variables:
        api_token: '${input.api_token}'
        org_id: '${input.org_id}'
        aws_account_number: '${input.aws_account_number}'
        sddc_name: '${input.sddc_name}'
        sddc_region: '${input.sddc_region}'
        vpc_cidr: '${input.vpc_cidr}'
        vxlan_subnet: '${input.vxlan_subnet}'
    providers: []
    terraformVersion: 0.12.29
    configurationSource:
        repositoryId: 19916daa-4d0d-4aa9-9887-f08b21dc0980
        commitId: 8243e868564692656d4dcc7c348a630c3ee0120b
        sourceDirectory: /vmc
```

 **Please note**
 + You could simple re-use this sample by updating the assosicated _Commit ID_, _Repository ID_, _Folder_ and _Constraints Tag_ from your enviroment.
 + The Tested Terraform CLI version were _0.12.29_ with the latest Terraform AWS Provider


