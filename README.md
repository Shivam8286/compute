# Google Cloud Compute Engine Provisioning with Terraform

This repository demonstrates how to provision a **Google Compute Engine (GCE)** virtual machine on **Google Cloud Platform (GCP)** using **Terraform**.  
With **Infrastructure as Code (IaC)**, you can automate and standardize cloud infrastructure deployment, ensuring repeatability and reducing manual errors.

---

## Table of Contents
- [Overview](#overview)  
- [Prerequisites](#prerequisites)  
- [Architecture](#architecture)  
- [Setup Instructions](#setup-instructions)  
  - [1. Configure Google Cloud Shell](#1-configure-google-cloud-shell)  
  - [2. Set Up Terraform State Storage](#2-set-up-terraform-state-storage)  
  - [3. Create Terraform Configuration Files](#3-create-terraform-configuration-files)  
  - [4. Run Terraform Commands](#4-run-terraform-commands)  
- [Verifying the Deployment](#verifying-the-deployment)  
- [Cleanup](#cleanup)  
- [References](#references)  

---

## Overview
This project provisions:
- A **Google Compute Engine (VM)** running Debian 12.
- Remote Terraform state storage in **Google Cloud Storage (GCS)**.
- Configurable **region**, **zone**, and **project** via variables.

The setup is **modular** and can be extended to include networking, firewall rules, and more.

---

## Prerequisites
Before starting, ensure you have:
- A **Google Cloud Project** with **billing enabled**.
- The **Project ID**.
- Access to **Google Cloud Shell** (pre-installed with Terraform and gcloud CLI).
- Permissions to create **VM instances** and **GCS buckets**.

---

## Architecture
```
[Terraform]
   |
   v
[Google Cloud Storage] <-- Stores Terraform state
   |
   v
[Google Compute Engine VM]
```

---

## Setup Instructions

### 1. Configure Google Cloud Shell
Open Google Cloud Shell from the GCP Console. Then:

```sh
# Authenticate
gcloud auth login

# Set your project
gcloud config set project PROJECT_ID

# Verify tools
terraform version
gcloud version
```

---

### 2. Set Up Terraform State Storage
Terraform uses a **state file** to track deployed resources.  
We store it in a **GCS bucket** for collaboration and safety.

```sh
# Create bucket for Terraform state
gsutil mb -l REGION gs://PROJECT_ID-tf-state

# Enable versioning to track state changes
gsutil versioning set on gs://PROJECT_ID-tf-state
```

---

### 3. Create Terraform Configuration Files

#### `main.tf`
```hcl
# ------------------------------
# Terraform settings
# ------------------------------
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"  # Google Cloud provider from HashiCorp
      version = "~> 4.0"            # Accept any 4.x.x version
    }
  }

  backend "gcs" {
    bucket = "PROJECT_ID-tf-state"  # Remote state storage in GCS
    prefix = "terraform/state"      # Folder path inside the bucket
  }
}

# ------------------------------
# Provider configuration
# ------------------------------
provider "google" {
  project = var.project_id  # Project ID
  region  = var.region      # Default region
}

# ------------------------------
# Create a VM instance
# ------------------------------
resource "google_compute_instance" "default" {
  name         = "terraform-instance"  # VM name
  machine_type = "e2-micro"             # Free-tier eligible in some regions
  zone         = var.zone               # Deployment zone

  # Boot disk configuration
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"  # OS image
    }
  }

  # Networking configuration
  network_interface {
    subnetwork   = "default"   # Using default subnetwork
    access_config {}           # Assigns external IP
  }
}
```

---

#### `variables.tf`
```hcl
variable "project_id" {
  description = "The ID of the Google Cloud project"
  type        = string
}

variable "region" {
  description = "The region to deploy resources in"
  type        = string
}

variable "zone" {
  description = "The zone to deploy resources in"
  type        = string
}
```

---

#### `outputs.tf` *(Optional but Recommended)*
you can create outputs.tf in similiar way after opening the editor. 
```hcl
# Display VM's external IP after deployment
output "vm_external_ip" {
  description = "The external IP address of the VM"
  value       = google_compute_instance.default.network_interface[0].access_config[0].nat_ip
}
```
<img width="914" height="541" alt="Image" src="https://github.com/user-attachments/assets/85a9978e-540b-4347-b771-2a34c7dcf640" />

---

### 4. Run Terraform Commands
```sh
# Initialize Terraform (downloads providers, sets up backend)
terraform init

# Preview changes before applying
terraform plan

# Apply configuration (deploy resources)
terraform apply -auto-approve
```
<img width="916" height="677" alt="Image" src="https://github.com/user-attachments/assets/9ebefe5d-5437-477d-8d77-525703fb6685" />

<img width="945" height="199" alt="Image" src="https://github.com/user-attachments/assets/6730b3d1-ce02-4ca1-9a3a-ec055de73123" />




---

## Verifying the Deployment
**Via Console**:  
Navigate to **Compute Engine > VM instances** in the Google Cloud Console.

**Via CLI**:
```sh
gcloud compute instances list
```
<img width="1306" height="266" alt="Image" src="https://github.com/user-attachments/assets/01321764-9cc2-44e4-b6a2-f76925cc7e5b" />
---

## Cleanup
To avoid costs:
```sh
terraform destroy -auto-approve
```
<img width="945" height="253" alt="Image" src="https://github.com/user-attachments/assets/eb7d7230-0283-4c66-8ee2-fbad77c0f31e" />

---

## References
- [Terraform Documentation](https://www.terraform.io/docs/)  
- [Google Cloud Compute Engine](https://cloud.google.com/compute/docs/)  
- [gcloud CLI Overview](https://cloud.google.com/sdk/gcloud)  
