# googlecloud-cheatsheet

## Terraform (Getting Started Guide)

#### Install  
wget URL
unzip FILENAME
cd $HOME
source ~/.bashrc

<hr>

#### Launch Code Editor from Cloud Shell
Launch Cloud Shell<br>
Open code editor

<hr>

#### Create Terraform files
mkdir DIRECTORY_NAME<br>
Open code editor<br>
Create following files under DIRECTORY_NAME:<br>

- provider.tf
- management.tf
- ./instance/main.tf
- privatenet.tf (add later)
- mynetwork.tf  (add later)

<details>
<summary>template</summary>
  
```
Details go here
```
</details>

<details>
<summary>provider.tf</summary>
  
```
provider "google" {}
```
</details>

<details>
<summary>main.tf</summary>
  
```
variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "n1-standard-1"
  }
variable "instance_subnetwork" {}

resource "google_compute_instance" "vm_instance" {
  name         = "${var.instance_name}"
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
      }
  }
  network_interface {
    subnetwork = "${var.instance_subnetwork}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
```
</details>

<details>
<summary>management.tf</summary>
  
```
# Create the managementnet network
resource "google_compute_network" "managementnet" {
  name                    = "managementnet"
  auto_create_subnetworks = "false"
}

# Create managementsubnet-us subnetwork
resource "google_compute_subnetwork" "managementsubnet-us" {
  name          = "managementsubnet-us"
  region        = "us-central1"
  network       = google_compute_network.managementnet.self_link
  ip_cidr_range = "10.130.0.0/20"
}

# Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet
resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
  name    = "managementnet-allow-http-ssh-rdp-icmp"
  network = google_compute_network.managementnet.self_link
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }
  allow {
    protocol = "icmp"
  }
}

# Add the managementnet-us-vm instance
module "managementnet-us-vm" {
  source              = "./instance"
  instance_name       = "managementnet-us-vm"
  instance_zone       = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.managementsubnet-us.self_link
}
```
</details>

<details>
<summary>mynetwork.tf</summary>
  
```
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
  name                    = "mynetwork"
  auto_create_subnetworks = true
}

# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork_allow_http_ssh_rdp_icmp" {
  name    = "mynetwork-allow-http-ssh-rdp-icmp"
  network = google_compute_network.mynetwork.self_link
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }
  allow {
    protocol = "icmp"
  }
}

# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source              = "./instance"
  instance_name       = "mynet-us-vm"
  instance_zone       = "us-central1-a"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}

# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source              = "./instance"
  instance_name       = "mynet-eu-vm"
  instance_zone       = "europe-west1-d"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}
```
</details>

<hr>

#### Initialize Terraform
<pre>
terraform init
</pre>

<hr>

#### Lint files
<pre>
terraform fmt
</pre>

<hr>

#### Create an execution plan
<pre>
terraform plan
</pre>

<hr>

#### Apply the desired changes
<pre>
terraform apply
</pre>

<hr>

#### Create a new VPC network
<details>
<summary>privatenet.tf</summary>
  
```
# Create privatenet network
resource "google_compute_network" "privatenet" {
  name                    = "privatenet"
  auto_create_subnetworks = false
}

# Create privatesubnet-us subnetwork
resource "google_compute_subnetwork" "privatesubnet-us" {
  name          = "privatesubnet-us"
  region        = "us-central1"
  network       = google_compute_network.privatenet.self_link
  ip_cidr_range = "172.16.0.0/24"
}

# Create privatesubnet-eu subnetwork
resource "google_compute_subnetwork" "privatesubnet-eu" {
  name          = "privatesubnet-eu"
  region        = "europe-west1"
  network       = google_compute_network.privatenet.self_link
  ip_cidr_range = "172.20.0.0/24"
}

# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on privatenet
resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
  name    = "privatenet-allow-http-ssh-rdp-icmp"
  network = google_compute_network.privatenet.self_link
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }
  allow {
    protocol = "icmp"
  }
}

# Add the privatenet-us-vm instance
module "privatenet-us-vm" {
  source              = "./instance"
  instance_name       = "privatenet-us-vm"
  instance_zone       = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.privatesubnet-us.self_link
}
```
</details>

<hr>

#### Re-initialize, fmt, plan, and apply
<pre>
terraform init
terraform fmt
terraform plan
terraform apply
</pre>

<hr>

#### Rinse and repeat 
If you want to create more VPC networks with VM's, simply re-use one of the existing .tf templates (eg: managementnet.tf)

<hr>

#### Verify
Go to the console and verify all objects have been created (eg: VPC networks, subnetworks, firewall rules, and virtual machines).
