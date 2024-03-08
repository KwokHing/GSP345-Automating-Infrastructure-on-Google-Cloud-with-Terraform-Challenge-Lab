## (GSP 345) Automating Infrastructure on Google Cloud with Terraform: Challenge Lab

**Task 1 : Create the configuration files** <br/>
Make the empty files and directories in _Cloud Shell_ or the _Cloud Shell Editor_.

```
touch main.tf
touch variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf
touch outputs.tf
touch variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf
touch outputs.tf
touch variables.tf
cd
```

Add the following to each of the _variables.tf_ files. Substitute with your project's specific **REGION**, **ZONE** and  **GCP PROJECT ID** values:
```
variable "region" {
 default = "<REGION>"
}

variable "zone" {
 default = "<ZONE>"
}

variable "project_id" {
 default = "<GCP PROJECT ID>"
}
```
Find the latest version of the Terraform Google provider at https://registry.terraform.io/providers/hashicorp/google/latest/docs by clicking the _Use Provider_ button and add the following to the _main.tf_ file:
```
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "<LATEST VERSION>"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}


module "instances" {
  source     = "./modules/instances"
}
```
Run "_terraform init_" in Cloud Shell from the root directory to initialize terraform.
```
terraform init
```
<br/> **Task 2: Import infrastructure** <br/>
Navigate to _Compute Engine > VM Instances_. Click on both instances and note the following values under the _Details_ tab: 

 - **Instance ID**
 - **Machine Type**
 - **Boot Disk Image**

Next, navigate to _modules/instances/instances.tf_. Copy the following configuration into the file and substitute the values were needed:
```
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "<MACHINE TYPE>"

  boot_disk {
    initialize_params {
      image = "<BOOT DISK IMAGE>"
    }
  }

  network_interface {
    network = "default"
  }

  metadata_startup_script = <<-EOT
  #!/bin/bash
  EOT

  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "<MACHINE TYPE>"

  boot_disk {
    initialize_params {
      image = "<BOOT DISK IMAGE>"
    }
  }

  network_interface {
    network = "default"
  }

  metadata_startup_script = <<-EOT
  #!/bin/bash
  EOT

  allow_stopping_for_update = true
}
```
To import the first instance, run the following command from the root directory using the Instance ID for _tf-instance-1_.
```
terraform import module.instances.google_compute_instance.tf-instance-1 <INSTANCE 1 ID>
```
To import the first instance, run the following command from the root directory using the Instance ID for _tf-instance-2_.
```
terraform import module.instances.google_compute_instance.tf-instance-2 <INSTANCE 2 ID>
```
The two instances have now been imported into your terraform configuration. You can now run the commands from the root directory to update the state of Terraform. Type _yes_ at the dialogue after you run the apply command to accept the state changes.
```
terraform plan
terraform apply
```
<br/> **Task 3: Configure a remote backend** <br/>
Add the following code to the _modules/storage/storage.tf_ file, and fill in the **Bucket Name** provided in the assignment:
```
resource "google_storage_bucket" "storage-bucket" {
  name          = "<BUCKET NAME>"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}
```
Next, add the following to the _main.tf_ file:
```
module "storage" {
  source     = "./modules/storage"
}
```
Run the following commands in the root directory to initialize the module and create the storage bucket resource. Type _yes_ at the dialogue after you run the apply command to accept the state changes.
```
terraform init
terraform apply
```
Next, update the _main.tf_ file so that the terraform block looks like the following. 
```
terraform {
  backend "gcs" {
    bucket  = "<BUCKET NAME>"
    prefix  = "terraform/state"
  }
  ...
}
```
Run the following to initialize the remote backend. Type _yes_ at the prompt.
```
terraform init
```
<br/> **Task 4: Modify and update infrastructure** <br/>
Navigate to _modules/instances/instance.tf_. Modifiy the values in the existing instances as specified in the assignment then add a third instance:
```
resource "google_compute_instance" "tf-instance-1" {
  machine_type = "<INSTANCE 1 MACHINE TYPE>"
  ...
}

resource "google_compute_instance" "tf-instance-2" {
  machine_type = "<INSTANCE 2 MACHINE TYPE>"
  ...
}

resource "google_compute_instance" "<INSTANCE 3 ID>" {
  name         = "<INSTANCE 3 ID>"
  machine_type = "<INSTANCE 3 MACHINE TYPE>"

  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "<BOOT DISK IMAGE>"
    }
  }

  network_interface {
   network = "default"
  }
}
```
Run the following commands from the root directory to initialize the module and create/update the instance resources. Type _yes_ at the dialogue after you run the apply command to accept the state changes.
```
terraform init
terraform apply
```
<br/> **Task 5: Destroy resources** <br/>
Remove the newly added resource by deleting the following code chunk from _modules/instances/instance.tf_.
```
resource "google_compute_instance" "<INSTANCE 3 ID>" {
  ...
}
```
Run the following commands from the root directory to apply the changes. Type _yes_ at the prompt.
```
terraform apply
```
<br/> **Task 6: Use a module from the Registry** <br/>
Copy and paste the following to the end of _main.tf_ file, fill in _Version Number_, _Network Name_ and the subnet IP values instructed by the challenge:
```
module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> <VERSION NUMBER>"

    project_id   = var.project_id
    network_name = "<NETWORK NAME>"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "<SUBNET-01 IP>"
            subnet_region         = var.region
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "<SUBNET-02 IP>"
            subnet_region         = var.region
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        }
    ]
}
```
Run the following commands to initialize the module and create the VPC. Type _yes_ at the prompt. If there is an error with the version of the network provider then use the latest version number (found here: https://registry.terraform.io/modules/terraform-google-modules/network/google/latest) and re-run the commands.
```
terraform init
terraform apply
```
Navigate to _modules/instances/instances.tf_ and update the instances with the following subnet values.
```
resource "google_compute_instance" "tf-instance-1" {
  ...
  network_interface {
    ...
      subnetwork = "subnet-01"
    }
   ...
}

resource "google_compute_instance" "tf-instance-1" {
  ...
  network_interface {
    ...
      subnetwork = "subnet-02"
    }
   ...
}
```
Run the following commands to initialize the module and update the instances. Type _yes_ at the prompt.
```
terraform init
terraform apply
```
<br/> **Task 7: Configure a firewall** <br/>
Add the following resource to the _main.tf_ file using the _Network Name_ from the assignment:
```
resource "google_compute_firewall" "tf-firewall" {
    name    = "tf-firewall"
    network = "projects/${var.project_id}/global/networks/<NETWORK NAME>"

    allow {
        protocol = "tcp"
        ports    = ["80"]
    }
    source_ranges = ["0.0.0.0/0"]
}
```
Run the following commands to configure the firewall. Type _yes_ at the prompt.
```
terraform init
terraform apply
```
