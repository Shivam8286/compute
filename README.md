
# Google Cloud Compute Engine Provisioning with Terraform

This repository demonstrates how to use **Terraform** to provision a Google Compute Engine (GCE) instance on Google Cloud Platform (GCP). Infrastructure as Code (IaC) enables repeatable, reliable, and scalable cloud infrastructure management.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
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

This project provisions a GCE virtual machine using Terraform. You define the infrastructure as code, allowing for easy provisioning, management, and version control of cloud resources.

---

## Prerequisites

- A Google Cloud project with billing enabled
- Project ID for your GCP project
- **Terraform** installed (pre-installed in Cloud Shell)
- **gcloud** CLI installed and authenticated (pre-installed in Cloud Shell)

---

## Setup Instructions

### 1. Configure Google Cloud Shell

- Activate Cloud Shell from the Google Cloud Console.
- Cloud Shell provides a persistent 5 GB home directory and comes pre-installed with development tools.
- Authenticate and set your project:
  ```sh
  gcloud auth login
  gcloud config set project PROJECT_ID
  ```

- Verify installations:
  ```sh
  terraform version
  gcloud version
  ```

### 2. Set Up Terraform State Storage

For collaborative and reliable state management, use a Google Cloud Storage (GCS) bucket:

```sh
gsutil mb -l REGION gs://PROJECT_ID-tf-state
gsutil versioning set on gs://PROJECT_ID-tf-state
```

### 3. Create Terraform Configuration Files

**main.tf**
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
  backend "gcs" {
    bucket = "PROJECT_ID-tf-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_instance" "default" {
  name         = "terraform-instance"
  machine_type = "e2-micro"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    subnetwork = "default"
    access_config {}
  }
}
```

**variables.tf**
```hcl
variable "project_id" {
  description = "The ID of the Google Cloud project"
  type        = string
  default     = "PROJECT_ID"
}

variable "region" {
  description = "The region to deploy resources in"
  type        = string
  default     = "REGION"
}

variable "zone" {
  description = "The zone to deploy resources in"
  type        = string
  default     = "ZONE"
}
```

### 4. Run Terraform Commands

Initialize and apply Terraform configuration:

```sh
terraform init
terraform plan
terraform apply -auto-approve
```

---

## Verifying the Deployment

- In the Google Cloud Console, navigate to **Compute Engine > VM instances** to verify the instance creation.
- Or run:
  ```sh
  gcloud compute instances list
  ```

---

## Cleanup

To avoid unnecessary charges, destroy the infrastructure when finished:

```sh
terraform destroy -auto-approve
```

---

## References

- [Terraform Documentation](https://www.terraform.io/docs/)
- [Google Cloud Compute Engine](https://cloud.google.com/compute/docs/)
- [gcloud CLI Overview](https://cloud.google.com/sdk/gcloud)

---

Would you like me to update the README file in your repository with this improved content? If yes, please confirm or let me know if you want any customization (like project name, region, or example values filled in).
