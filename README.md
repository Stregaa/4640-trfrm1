# 4640-trfrm1

To download and install Terraform for Linux in WSL Ubuntu on a Windows machine:  
1. Go to https://developer.hashicorp.com/terraform/downloads  
2. Click the download button next to the AMD64 version to download the zipped Terraform binary.  
3. Log into your Ubuntu instance.  
4. Move the zipped file from wherever you downloaded it to the Ubuntu instance:  
```sudo mv /mnt/c/<downloaded file> <intended location>``` 
5. Install unzip if it's not already installed:  
```
sudo apt-get update
sudo apt-get install unzip
```
6. Create a ~/bin directory:  
```sudo mkdir ~/bin```
7. Unzip the zipped file into your intended directory:  
```unzip <zipped file> <intended location>```
8. Source the .profile:  
```source ~/.profile```

Setting up Terraform with DigitalOcean:  
1. Create and save a new personal access API token on DigitalOcean.  
2. Create a new .env file in the project directory, and include the new API token you just made:  
```export TF__VAR__do_token=<API Token>```
3. Source the new token:  
```source .env```
4. Create a new configuration main.tf file for Terraform, and include:  
```
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

# Set the variable value in *.tfvars file
# or using -var="do_token=..." CLI option
variable "do_token" {}

# Configure the DigitalOcean Provider
provider "digitalocean" {
  token = var.do_token
}
```
5. Initialize Terraform:  
```terraform init```
6. Add a new data block to main.tf for an SSH key:  
```
data "digitalocean_ssh_key" "droplet_ssh_key" {
  name = "ATTEMPT1000"
}
```
7. Add a new data block to main.tf for a new project:  
```
data "digitalocean_project" "lab_project" {
  name = "first-project"
}
```
8. Add a new resource block to main.tf for a new tag:  
```
resource "digitalocean_tag" "do_tag" {
  name = "Web"
}
```
9. Add a new resource block to main.tf for a new VPC:
```
resource "digitalocean_vpc" "web_vpc" {
  name     = "4640labs"
  region   = "sfo3"
}
```
10. Add a new resource block to main.tf for a new droplet:  
```
resource "digitalocean_droplet" "web" {
  image  = "rockylinux-9-x64"
  name   = "web-1"
  tags   = [digitalocean_tag.do_tag.id]
  region = "sfo3"
  size   = "s-1vcpu-512mb-10gb"
  ssh_keys = [data.digitalocean_ssh_key.droplet_ssh_key.id]
  vpc_uuid = digitalocean_vpc.web_vpc.id
}
```
11. Add a new resource block to main.tf for a new digital ocean project resource:  
```
resource "digitalocean_project_resources" "project_attach" {
  project = data.digitalocean_project.lab_project.id
  resources = [
    digitalocean_droplet.web.urn
  ]
}
```
12. Add a new output block to main.tf: 
```
output "server_ip" {
  value = digitalocean_droplet.web.ipv4_address
}
```
13. Validate main.tf:  
```terraform validate```
14. Check the terraform plan:  
```terraform plan```
15. Apply terraform and create the resources:  
```terraform apply```
16. To delete the created resources [THERE IS NO UNDO]:  
```terraform destroy```
