
# LAB: Creating Virtual Machines

# OBJECTIVES

In this lab, you explore the available options for VMs and see the differences between locations.

In this lab, you will learn how to perform the following tasks:

- Create several standard VMs

- Create advanced VMs

# PREREQUISITE

GCLOUD SDK OR CLOUD SHELL must be present and running, with all required authorization given.
NOTE: This project assumes you have the required tools need to run this command-line.
Hence would not go into details of Installing, Authorizing and Enabling the SDK or Cloud shell environment.
For details check the Google cloud documentation for help.

# TASK 1: CREATING A UTILITY VIRTUAL MACHINE

- Create a VM:

STEPS:

    *   Create a Virtual Machine, Using the following commands

        gcloud beta compute instances create my-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address --image=debian-10-buster-v20200910 --image-project=debian-cloud

    *   Explore the VM details

        gcloud compute instances describe my-vm --zone=us-central1-c

        Result:
        you would be able to locate the CPU platform, Availability policies and their values.

# TASK 2: CREATING A WINDOWS VIRTUAL MACHINE

- Create a VM:

STEPS:  

    *   Create instance

        gcloud beta compute instances create my-windows-vm-2 --zone=europe-west2-a --machine-type=n1-standard-1 --subnet=default --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200908 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd

    *   For Firewall, allow HTTP traffic and HTTPS traffic

        gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

        gcloud compute firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server

    *   Set the password for the VM

        gcloud compute reset-windows-password my-windows-vm-2  --user=student_03_4ae9bd942 --zone=europe-west2-a

        Type "yes" at the prompt

        Enter new password of you choice.

# TASK 3: CREATING A CUSTOM VIRTUAL MACHINE

- Create a VM:

STEPS:  

    *   Click Create instance

        gcloud beta compute instances create my-custom-vm-3 --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default --network-tier=PREMIUM --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard

    *   Connect via SSH to your custom VM

        gcloud compute ssh instance-2 --zone=us-west1-b

    *   To see information about unused and used memory and swap space on your custom VM, run the following command:

        free

    *   To see details about the RAM installed on your VM, run the following command:

        sudo dmidecode -t 17

    *   To verify the number of processors, run the following command:

        nproc

    *   To see details about the CPUs installed on your VM, run the following command:

        lscpu

# Congratulations

############################################################################################################################################
A Summary of all the codes that was executed:

# TASK 1

    gcloud beta compute instances create my-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address --image=debian-10-buster-v20200910 --image-project=debian-cloud

    gcloud compute instances describe my-vm --zone=us-central1-c

# TASK 2

    gcloud beta compute instances create my-windows-vm-2 --zone=europe-west2-a --machine-type=n1-standard-1 --subnet=default --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200908 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd

    gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

    gcloud compute firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server

    gcloud compute reset-windows-password my-windows-vm-2 --zone=europe-west2-a

# TASK 3

    gcloud beta compute instances create my-custom-vm-3 --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default --network-tier=PREMIUM --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard

    gcloud compute ssh my-custom-vm-3 --zone=us-west1-b
    
    free

    sudo dmidecode -t 17

    nproc

    lscpu

Done
