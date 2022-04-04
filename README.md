## Automating Infrastructure on Google Cloud with Terraform: Challenge Lab (GSP345) ##

**Setup : Create the configuration files** <br/>
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

Add the following to the each _variables.tf_ file, and fill in the _GCP Project ID_:
```
variable "region" {
 default = "us-central1"
}

variable "zone" {
 default = "us-central1-a"
}

variable "project_id" {
 default = "<FILL IN PROJECT ID>"
}
```
Add the following to the _main.tf_ file:
```
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
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
Run "_terraform init_" in Cloud Shell in the root directory to initialize terraform.
```
terraform init
```
<br/> **TASK 1: Import infrastructure** <br/>
Navigate to _Compute Engine > VM Instances_. Click on _tf-instance-1_. Copy the _Instance ID_ down somewhere to use later. <br/>
Navigate to _Compute Engine > VM Instances_. Click on _tf-instance-2_. Copy the _Instance ID_ down somewhere to use later. <br/>
Next, navigate to _modules/instances/instances.tf_. Copy the following configuration into the file:
```
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
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
  machine_type = "n1-standard-1"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
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
To import the first instance, use the following command, using the Instance ID for _tf-instance-1_ you copied down earlier.
```
terraform import module.instances.google_compute_instance.tf-instance-1 <FILL IN INSTANCE 1 ID>
```
To import the second instance, use the following command, using the Instance ID for _tf-instance-2_ you copied down earlier.
```
terraform import module.instances.google_compute_instance.tf-instance-2 <FILL IN INSTANCE 2 ID>
```
The two instances have now been imported into your terraform configuration. You can now run the commands to update the state of Terraform. Type _yes_ at the dialogue after you run the apply command to accept the state changes.
```
terraform plan
terraform apply
```
<br/> **TASK 2: Configure a remote backend** <br/>
Add the following code to the _modules/storage/storage.tf_ file, and fill in the _Bucket Name_:
```
resource "google_storage_bucket" "storage-bucket" {
  name          = "<FILL IN BUCKET NAME>"
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
Run the following commands to initialize the module and create the storage bucket resource. Type _yes_ at the dialogue after you run the apply command to accept the state changes.
```
terraform init
terraform apply
```
Next, update the _main.tf_ file so that the terraform block looks like the following. Fill in your _GCP Project ID_ for the bucket argument definition.
```
terraform {
  backend "gcs" {
    bucket  = "<FILL IN PROJECT ID>"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}
```
Run the following to initialize the remote backend. Type _yes_ at the prompt.
```
terraform init
```
<br/> **TASK 3: Modify and update infrastructure** <br/>
Navigate to _modules/instances/instance.tf_. Replace the entire contents of the file with the following, and fill in your _Instance 3 ID_:
```
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "<FILL IN INSTANCE 3 NAME>" {
  name         = "<FILL IN INSTANCE 3 NAME>"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```
Run the following commands to initialize the module and create/update the instance resources. Type _yes_ at the dialogue after you run the apply command to accept the state changes.
```
terraform init
terraform apply
```
<br/> **TASK 4: Taint and destroy resources** <br/>
Taint the _tf-instance-3_ resource by running the following command, and fill in your _Instance 3 ID_:
```
terraform taint module.instances.google_compute_instance.<FILL IN INSTANCE 3 NAME>
```
Run the following commands to apply the changes:
```
terraform init
terraform apply
```
Remove the _tf-instance-3_ resource from the _instances.tf_ file. Delete the following code chunk from the file.
```
resource "google_compute_instance" "<FILL IN INSTANCE 3 NAME>" {
  name         = "<FILL IN INSTANCE 3 NAME>"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```
Run the following commands to apply the changes. Type _yes_ at the prompt.
```
terraform apply
```
<br/> **TASK 5: Use a module from the Registry** <br/>
Copy and paste the following to the end of _main.tf_ file, fill in _Version Number_ and _Network Name_ instructed in the challenge:
```
module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> <FILL IN VERSION NUMBER>"

    project_id   = "qwiklabs-gcp-04-f2c1c01a09d3"
    network_name = "<FILL IN NETWORK NAME>"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-central1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-central1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        }
    ]
}
```
Run the following commands to initialize the module and create the VPC. Type _yes_ at the prompt.
```
terraform init
terraform apply
```
Navigate to _modules/instances/instances.tf_. Replace the entire contents of the file with the following:
```
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "<FILL IN NETWORK NAME>"
    subnetwork = "subnet-01"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "<FILL IN NETWORK NAME>"
    subnetwork = "subnet-02"
  }
}
```
Run the following commands to initialize the module and update the instances. Type _yes_ at the prompt.
```
terraform init
terraform apply
```
<br/> **TASK 6: Configure a firewall** <br/>
Add the following resource to the _main.tf_ file, fill in the _GCP Project ID_ and _Network Name_:
```
resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-firewall"
 network = "projects/<FILL IN PROJECT_ID>/global/networks/<FILL IN NETWORK NAME>"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}
```
Run the following commands to configure the firewall. Type _yes_ at the prompt.
```
terraform init
terraform apply
```
