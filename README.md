Google Cloud Compute Engine Provisioning with Terraform

This repository demonstrates how to provision a Google Compute Engine (GCE) virtual machine on Google Cloud Platform (GCP) using Terraform.
With Infrastructure as Code (IaC), you can automate and standardize cloud infrastructure deployment, ensuring repeatability and reducing manual errors.

Table of Contents

Overview

Prerequisites

Architecture

Setup Instructions

1. Configure Google Cloud Shell

2. Set Up Terraform State Storage

3. Create Terraform Configuration Files

4. Run Terraform Commands

Verifying the Deployment

Cleanup

References

Overview

This project provisions:

A Google Compute Engine (VM) running Debian 12.

Remote Terraform state storage in Google Cloud Storage (GCS).

Configurable region, zone, and project via variables.

The setup is modular and can be extended to include networking, firewall rules, and more.

Prerequisites

Before starting, ensure you have:

A Google Cloud Project with billing enabled.

The Project ID.

Access to Google Cloud Shell (pre-installed with Terraform and gcloud CLI).

Permissions to create VM instances and GCS buckets.

Architecture
[Terraform]
   |
   v
[Google Cloud Storage] <-- Stores Terraform state
   |
   v
[Google Compute Engine VM]


Setup Instructions
1. Configure Google Cloud Shell

Open Google Cloud Shell from the GCP Console. Then:

# Authenticate
gcloud auth login

# Set your project
gcloud config set project PROJECT_ID

# Verify tools
terraform version
gcloud version

2. Set Up Terraform State Storage

Terraform uses a state file to track deployed resources.
We store it in a GCS bucket for collaboration and safety.

# Create bucket for Terraform state
gsutil mb -l REGION gs://PROJECT_ID-tf-state

# Enable versioning to track state changes
gsutil versioning set on gs://PROJECT_ID-tf-state

3. Create Terraform Configuration Files
main.tf
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

variables.tf
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

outputs.tf (Optional but Recommended)
# Display VM's external IP after deployment
output "vm_external_ip" {
  description = "The external IP address of the VM"
  value       = google_compute_instance.default.network_interface[0].access_config[0].nat_ip
}

4. Run Terraform Commands
# Initialize Terraform (downloads providers, sets up backend)
terraform init

# Preview changes before applying
terraform plan

# Apply configuration (deploy resources)
terraform apply -auto-approve

Verifying the Deployment

Via Console:
Navigate to Compute Engine > VM instances in the Google Cloud Console.

Via CLI:

gcloud compute instances list

Cleanup

To avoid costs:

terraform destroy -auto-approve

References

Terraform Documentation

Google Cloud Compute Engine

gcloud CLI Overview
