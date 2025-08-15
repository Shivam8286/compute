# Google Cloud Compute Engine Provisioning with Terraform

This repository demonstrates how to use **Terraform** to provision resources on Google Cloud Platform (GCP), specifically a Compute Engine (GCE) virtual machine. By leveraging Infrastructure as Code (IaC), you can manage, version, and scale your cloud infrastructure in a reliable and automated manner.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Code Explanation](#code-explanation)
- [Verifying the Deployment](#verifying-the-deployment)
- [Cleanup](#cleanup)
- [References](#references)

---

## Overview

This project provisions a GCE virtual machine using Terraform. All resources are defined as code, enabling reproducible and auditable deployments.

---

## Prerequisites

- A Google Cloud project with billing enabled
- Project ID for your GCP project
- **Terraform** installed (Cloud Shell includes this by default)
- **gcloud** CLI installed and authenticated (Cloud Shell includes this by default)

---

## Setup Instructions

1. **Configure Google Cloud Shell**
   - Open Cloud Shell in the Google Cloud Console.
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

2. **Set Up Terraform State Storage**
   - Create a Google Cloud Storage (GCS) bucket for storing Terraform state:
     ```sh
     gsutil mb -l REGION gs://PROJECT_ID-tf-state
     gsutil versioning set on gs://PROJECT_ID-tf-state
     ```

3. **Edit the Terraform Configuration Files**
   - Update `variables.tf` to set your desired `project_id`, `region`, and `zone`.

4. **Run Terraform**
   ```sh
   terraform init
   terraform plan
   terraform apply -auto-approve
   ```

---

## Code Explanation

The main Terraform configuration (`main.tf`) accomplishes the following:

- **Specifies the provider**: Configures the Google provider with the required version and project/region settings.
- **Configures the backend**: Stores Terraform state in a remote GCS bucket for reliability and collaboration.
- **Provisions a Compute Engine VM**: 
  - Creates a VM instance with the name `terraform-instance`.
  - Sets the machine type (e2-micro, suitable for low-cost workloads).
  - Chooses the boot disk image (Debian 12).
  - Attaches the VM to the default network and assigns a public IP via `access_config`.

**Example main.tf snippet:**
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
---

## Verifying the Deployment

- In the Google Cloud Console, go to **Compute Engine > VM instances** to see your running instance.
- Or use:
  ```sh
  gcloud compute instances list
  ```

---

## Cleanup

To avoid incurring charges, destroy the resources when you are finished:
```sh
terraform destroy -auto-approve
```

---

## References

- [Terraform Documentation](https://www.terraform.io/docs/)
- [Google Cloud Compute Engine](https://cloud.google.com/compute/docs/)
- [gcloud CLI Overview](https://cloud.google.com/sdk/gcloud)
