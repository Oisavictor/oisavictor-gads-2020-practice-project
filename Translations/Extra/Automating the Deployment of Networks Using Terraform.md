
# LAB:  Automating the Deployment of Networks Using Terraform

# OBJECTIVES

In this lab, you learn how to perform the following tasks:

- Create a configuration for a custom-mode network

- Create a configuration for a firewall rule

- Create a module for VM instances

- Create a configuration for an auto-mode network

- Create and deploy a configuration

- Verify the deployment of a configuration

# PREREQUISITE

GCLOUD SDK OR CLOUD SHELL must be present and running, with all required authorization given.
Note: This project assumes you have the required tools need to run this command-line.
Hence would not go into details of Installing, Authorizing and Enabling the SDK or Cloud shell environment.
For details check the Google cloud documentation for help.

# TASK 1: SETTING UP TERRAFORM ENVIRONMENT

- Set up Terraform and Cloud Shell

STEPS:

    *   Download Terraform

        Use the wget command to download the installation file from terraform website

        wget https://releases.hashicorp.com/terraform/0.12.24/terraform_0.12.24_linux_amd64.zip

        Note: This creates a folder terraform in the HOME folder, i.e /Home/terraform

    *   Next, unzip the file with the command

        unzip terraform_0.13.2_linux_amd64.zip

    *   we set the path environment variable,for the terraform binaires as follows

        export PATH="$PATH:$HOME/terraform"

        cd /usr/bin

        sudo ln -s $HOME/terraform

        cd $HOME

        source ~/ .bashrc

    *   Confirm that Terraform is installed by running the following command

        terraform --version

        Result:
        Terraform v0.12.24

    *   Create an environment variable for the Project_ID, with the command

        export GOOGLE_PROJECT=$(gcloud config get-value project)
    
    *   Create a folder where we would store all the configurations we are going to write, with the command

        mkdir tfnet

        enter into the folder by excuting the command

        cd tfnet

    *   Oncce we are in the tfnet folder, we initialize Terraform by setting Google as the provider,by creating a file called provider.tf

        touch provider.tf

        We edit the provider.tf file by using the nano file editor

        sudo nano provider.tf

        We now paste this code into the file

        provider "google" {}

        We save and exit the file, with the command Crtl + O , press enter and ctrl + X respectively.

    *   Initialize Terraform by running the following command

        terraform init

        Result:
        * provider.google: version = "~> 3.36"
        Terraform has been successfully initialized!

    *   You are now ready to work with Terraform in Cloud Shell

# TASK 2: CREATE A CONFIGURATION FOR A CUSTOM-MODE NETWORK AND FIREWALL RULE

- Create the custom-mode network managementnet along with its firewall rule and VM instance

STEPS:

    *   Create a new configuration and define managementnet network and firewall rules

        touch Management.tf

    *   We edit the Management.tf file by using the nano file editor

        sudo nano Management.tf

    *   We now paste this code into the file

        # Create the managementnet network
        resource "google_compute_network" "managementnet" {
          name                    = "managementnet"
          auto_create_subnetworks = "false"
        }

        # Create managementsubnet-us subnetwork
        resource "google_compute_subnetwork" "managementsubnet-us" {
          name          = "managementsubnet-us"
          region        = "us-central1"
          network       = "${google_compute_network.managementnet.self_link}
          ip_cidr_range = "10.130.0.0/20"
        }

        # Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet
        resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
          name    = "managementnet-allow-http-ssh-rdp-icmp"
          network = "${google_compute_network.managementnet.self_link}
          allow {
            protocol = "tcp"
            ports    = ["22", "80", "3389"]
          }
          allow {
            protocol = "icmp"
          }
        }


    *   We save and exit the file, with the command Crtl + O , press enter and ctrl + X respectively.

# TASK 3: CREATE A MODULE FOR VM INSTANCE

- Define the VM instance by creating a VM instance module.

STEPS

        Create a VM instance module. A module is a reusable configuration inside a folder. You will use this module for all VM instances

    *   create a new folder inside tfnet, using the mkdir command

        mkdir instance

    *   change into the insatnce directory and create a new file named main.tf, with the following command

        cd instance

        touch main.tf

    *   Then open main.tf file by using the nano file editor

        sudo nano main.tf

    *   We now paste this code into the file

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

    *   We save and exit the file, with the command Crtl + O , press enter and ctrl + X respectively.


    *  Now go back and Add the following VM instance to managementnet.tf by navigating to the tfnet folder

        cd ..

       sudo nano Management.tf

    *   We now paste this code into the file

        # Add the managementnet-us-vm instance
        module "managementnet-us-vm" {
          source              = "./instance"
          instance_name       = "managementnet-us-vm"
          instance_zone       = "us-central1-a"
          instance_subnetwork = google_compute_subnetwork.managementsubnet-us.self_link
        }

    *   We save and exit the file, with the command Crtl + O , press enter and ctrl + X respectively.

# TASK 4: CREATE A DEPLOYMENT FOR THE CUSTOM-MODE NETWORK

- Create managementnet and its resources

STEPS:

    *   Rewrite the Terraform configurations files to a canonical format and style by running the following command

        terraform fmt

    *   Initialize Terraform by running the following command

        terraform init

        Result:
        Initializing modules...
        - managementnet-us-vm in instance
        ...
        * provider.google: version = "~> 3.36"
        Terraform has been successfully initialized!

    *   Create an execution plan by running the following command

        terraform plan

        Result
        ...
        Plan: 4 to add, 0 to change, 0 to destroy.
        ...

    *   Apply the desired changes by running the following command:

        terraform apply

    *   Confirm the planned actions by typing:

        yes

        Result:
        ...
        Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
    
    *   Verify managementnet and its resources that were created.

        gcloud compute networks list
        
        gcloud compute networks subnets list-usable

        Result:
        You should see both the management network and its subnetwork. As shown below:
        
        NAME           SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
        default        AUTO         REGIONAL
        managementnet  CUSTOM       REGIONAL

        PROJECT                       REGION                   NETWORK        SUBNET                 RANGE          SECONDARY_RANGES
        qwiklabs-gcp-00-69fa058272e9  us-central1              default        default               10.130.0.0/20
        qwiklabs-gcp-00-69fa058272e9  us-central1              managementnet  managementsubnet-us   10.130.0.0/20

    *   Verify mananagementnet_allow_http_ssh_rdp_icmp firewall rule for the VPC network that was created.

        gcloud compute firewall-rules list

        Result:
        You should see the management_allow_http_ssh_rdp rule
        NAME                                   NETWORK        DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
        default-allow-icmp                     default        INGRESS    65534     icmp                                False
        default-allow-internal                 default        INGRESS    65534     tcp:0-65535,udp:0-65535,icmp        False
        default-allow-rdp                      default        INGRESS    65534     tcp:3389                            False
        default-allow-ssh                      default        INGRESS    65534     tcp:22                              False
        managementnet-allow-http-ssh-rdp-icmp  managementnet  INGRESS    1000      icmp,tcp:22,tcp:80,tcp:3389         False

    *   Verify VM instances

        gcloud compute instances list

        Result:
        NAME                 ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
        managementnet-us-vm  us-central1-a  n1-standard-1               10.130.0.2   34.123.157.120  RUNNING
    
    -   CREATE PRIVATENET AND IT'S RESOURCES

    *   Repeat the above process and creat a new file with the name privatenet.tf file
        Note this file should be located in the tfnet folder also

        The COMPLETED FILE is shown below

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

        Save the file with ctrl + O and exit the nano editor

    *   Rewrite the Terraform configurations files to a canonical format and style by running the following command

        terraform fmt

    *   Initialize Terraform by running the following command

        terraform init

        Result:
        Initializing modules...
        - privatenet-us-vm in instance
        ...
        * provider.google: version = "~> 3.36"
        Terraform has been successfully initialized!

    *   Create an execution plan by running the following command

        terraform plan

        Result
        ...
        Plan: 5 to add, 0 to change, 0 to destroy.
        ...

    *   Apply the desired changes by running the following command:

        terraform apply

    *   Confirm the planned actions by typing:

        yes

        Result:
        ...
        Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

    *   Verify privatetnet and its resources that were created.

        gcloud compute networks list
        
        gcloud compute networks subnets list-usable

        Result:
        You should see both the privatenet network and its subnetwork.
        NAME           SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
        default        AUTO         REGIONAL
        managementnet  CUSTOM       REGIONAL
        privatenet     CUSTOM       REGIONAL

        PROJECT                       REGION                   NETWORK        SUBNET               RANGE          SECONDARY_RANGES
        qwiklabs-gcp-00-69fa058272e9  europe-west1             privatenet     privatesubnet-eu     172.20.0.0/24
        qwiklabs-gcp-00-69fa058272e9  us-central1              privatenet     privatesubnet-us     172.16.0.0/24
        qwiklabs-gcp-00-69fa058272e9  us-central1              managementnet  managementsubnet-us  10.130.0.0/20

    *   Verify privatenet_allow_http_ssh_rdp_icmp firewall rule for the VPC network that was created.

        gcloud compute firewall-rules list

        Result:
        You should see the privatenet_allow_http_ssh_rdp rule
        NAME                                   NETWORK        DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
        default-allow-icmp                     default        INGRESS    65534     icmp                                False
        default-allow-internal                 default        INGRESS    65534     tcp:0-65535,udp:0-65535,icmp        False
        default-allow-rdp                      default        INGRESS    65534     tcp:3389                            False
        default-allow-ssh                      default        INGRESS    65534     tcp:22                              False
        managementnet-allow-http-ssh-rdp-icmp  managementnet  INGRESS    1000      icmp,tcp:22,tcp:80,tcp:3389         False
        privatenet-allow-http-ssh-rdp-icmp     privatenet     INGRESS    1000      icmp,tcp:22,tcp:80,tcp:3389         False

    *   Verify VM instances

        gcloud compute instances list

        Result:
        NAME                 ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
        managementnet-us-vm  us-central1-a  n1-standard-1               10.130.0.2   34.123.157.120  RUNNING
        privatenet-us-vm     us-central1-a  n1-standard-1               172.16.0.2   34.123.113.154  RUNNING

# TASK 5: CREATE A CONFIGURATION FOR AN AUTO-MODE NETWORK

- Create mynetwork and its resources

STEPS
    
    *   Repeat the above process and creat a new file with the name mynetwork.tf file
        Note this file should be located in the tfnet folder also

        The COMPLETED FILE is shown below #(Notice that the auto_create_subnetworks flag is set to true)

    *   Also do same for mynetwork

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

    *   Save the file with ctrl + O and exit the nano editor

    *   Rewrite the Terraform configurations files to a canonical format and style by running the following command

        terraform fmt

    *   Initialize Terraform by running the following command

        terraform init

        Result:
        Initializing modules...
        - mynet-eu-vm in instance
        - mynet-us-vm on instance
        ...
        * provider.google: version = "~> 3.36"
        Terraform has been successfully initialized!

    *   Create an execution plan by running the following command

        terraform plan

        Result
        ...
        Plan: 4 to add, 0 to change, 0 to destroy.
        ...

    *   Apply the desired changes by running the following command:

        terraform apply

    *   Confirm the planned actions by typing:

        yes

        Result:
        ...
        Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

    *   Verify mynetworknet and its resources that were created.

        gcloud compute networks list
        
        gcloud compute networks subnets list-usable

        Result:
        You should see both the privatenet network and its subnetwork.
        NAME           SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
        default        AUTO         REGIONAL
        managementnet  CUSTOM       REGIONAL
        mynetwork      AUTO         REGIONAL
        privatenet     CUSTOM       REGIONAL

        PROJECT                       REGION                   NETWORK        SUBNET               RANGE          SECONDARY_RANGES
        qwiklabs-gcp-00-69fa058272e9  asia-southeast2          mynetwork      mynetwork            10.184.0.0/20
        qwiklabs-gcp-00-69fa058272e9  us-west4                 mynetwork      mynetwork            10.182.0.0/20
        .
        .
        .
        qwiklabs-gcp-00-69fa058272e9  us-central1              mynetwork      mynetwork            10.128.0.0/20
        qwiklabs-gcp-00-69fa058272e9  europe-west1             privatenet     privatesubnet-eu     172.20.0.0/24
        qwiklabs-gcp-00-69fa058272e9  us-central1              privatenet     privatesubnet-us     172.16.0.0/24

    *   Verify mynetwork_allow_http_ssh_rdp_icmp firewall rule for the VPC network that was created.

        gcloud compute firewall-rules list

        Result:
        You should see the mynetwork_allow_http_ssh_rdp rule
        NAME                                   NETWORK        DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
        default-allow-icmp                     default        INGRESS    65534     icmp                                False
        default-allow-internal                 default        INGRESS    65534     tcp:0-65535,udp:0-65535,icmp        False
        default-allow-rdp                      default        INGRESS    65534     tcp:3389                            False
        default-allow-ssh                      default        INGRESS    65534     tcp:22                              False
        managementnet-allow-http-ssh-rdp-icmp  managementnet  INGRESS    1000      icmp,tcp:22,tcp:80,tcp:3389         False
        mynetwork-allow-http-ssh-rdp-icmp      mynetwork      INGRESS    1000      icmp,tcp:22,tcp:80,tcp:3389         False
        privatenet-allow-http-ssh-rdp-icmp     privatenet     INGRESS    1000      icmp,tcp:22,tcp:80,tcp:3389         False

    *   Verify VM instances

        gcloud compute instances list

        Result:
        You should see mynet-us-vm and mynet-eu-vm instances.
        NAME                 ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
        mynet-eu-vm          europe-west1-d  n1-standard-1               10.132.0.2   34.76.202.170   RUNNING
        managementnet-us-vm  us-central1-a   n1-standard-1               10.130.0.2   34.123.157.120  RUNNING
        mynet-us-vm          us-central1-a   n1-standard-1               10.128.0.2   34.123.186.211  RUNNING
        privatenet-us-vm     us-central1-a   n1-standard-1               172.16.0.2   34.123.113.154  RUNNING

        Note the internal IP addresses for mynet-eu-vm.

    *   To test connectivity to mynet-eu-vm's internal IP address, run the following command in the SSH terminal   

        gcloud compute ssh mynet-us-vm --zone=us-central1-a

        ping -c 3 <Enter mynet-eu-vm's internal IP here>

        ping -c 3 10.132.0.2

        Result:
        This should work because both VM instances are on the same network, and ICMP traffic is allowed!

# Congratulations

############################################################################################################################################
A Summary of all the codes that was executed:

# TASK 1

    wget https://releases.hashicorp.com/terraform/0.12.24/terraform_0.12.24_linux_amd64.zip

    unzip terraform_0.13.2_linux_amd64.zip

    export PATH="$PATH:$HOME/terraform"

    cd /usr/bin

    sudo ln -s $HOME/terraform

    cd $HOME

    source ~/ .bashrc

    terraform --version

    export GOOGLE_PROJECT=$(gcloud config get-value project)

    terraform --version
    
    mkdir tfnet

    cd tfnet
    
    echo provider "google" {} > provider.tf

    terraform init

# TASK 2

    touch managementnet.tf

    sudo nano managementnet.tf

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

# TASK 3

    mkdir instance

    cd instance
    
    touch main.tf

    sudo nano main.tf

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

# TASK 4

    cd ..
    
    terraform fmt

    terraform init

    terraform plan

    terraform apply -auto-approve

    gcloud compute networks list
        
    gcloud compute networks subnets list-usable

    gcloud compute firewall-rules list

    gcloud compute instances list

    touch privatenet.tf

    Sudo nano privatenet.tf

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

    terraform fmt

    terraform init

    terraform plan

    terraform apply -auto-approve

    gcloud compute networks list
        
    gcloud compute networks subnets list-usable

    gcloud compute firewall-rules list

    gcloud compute instances list

# TASK 5

    touch mynetwork.tf

    Sudo nano mynetwork.tf

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

    terraform fmt

    terraform init

    terraform plan

    terraform apply --auto-approve

    gcloud compute networks list
        
    gcloud compute networks subnets list-usable

    gcloud compute firewall-rules list

    gcloud compute instances list

    gcloud compute ssh mynet-us-vm --zone=us-central1-a

    ping -c 3 10.132.0.2
