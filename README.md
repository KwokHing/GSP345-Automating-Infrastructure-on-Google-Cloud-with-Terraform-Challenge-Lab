## Automating Infrastructure on Google Cloud with Terraform: Challenge Lab (GSP345) ##

Setup : Create the configuration files <br/>
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

Add the following to the each _variables.tf_ file, and fill in the GCP Project ID:
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
