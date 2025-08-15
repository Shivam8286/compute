Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

Click Activate Cloud Shell Activate Cloud Shell icon at the top of the Google Cloud console.
When you are connected, you are already authenticated, and the project is set to your PROJECT_ID. The output contains a line that declares the PROJECT_ID for this session:

Your Cloud Platform project in this session is set to YOUR_PROJECT_ID
gcloud is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

(Optional) You can list the active account name with this command:
gcloud auth list
Copied!
Click Authorize.

Your output should now look like this:

Output:

ACTIVE: *
ACCOUNT: student-01-xxxxxxxxxxxx@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`
(Optional) You can list the project ID with this command:
gcloud config list project
Copied!
Output:

[core]
project = <project_ID>
Example output:

[core]
project = qwiklabs-gcp-44776a13dea667a6
Note: For full documentation of gcloud, in Google Cloud, refer to the gcloud CLI overview guide.
Overview
This lab demonstrates how to use Terraform to create a Google Compute Engine (GCE) instance. You will define your infrastructure as code, allowing you to easily provision and manage your resources. This lab assumes you have basic knowledge of Google Cloud and Terraform.

Task 1. Prerequisites
Before you begin, ensure you have the following:

A Google Cloud project with billing enabled. You will need the Project ID for subsequent steps. The project ID will be: PROJECT_ID

Your Project ID is: "PROJECT_ID"
Note:
Make note of this ID; you will need it in the next steps.
Terraform installed on your local machine or in Cloud Shell. If using Cloud Shell, Terraform is pre-installed.

terraform version
Copied!
Note:
Verify Terraform is installed.
The Google Cloud SDK (gcloud) installed and configured. If using Cloud Shell, gcloud is pre-installed and authenticated.

gcloud version
Copied!
Note:
Verify gcloud is installed.
Authenticate to your Google Cloud project. In Cloud Shell, this might be done automatically.

gcloud auth login
Copied!
Note:
Authenticate gcloud to access your Google Cloud resources.
Set your Project ID to be: PROJECT_ID

gcloud config set project "PROJECT_ID"
Copied!
Note:
Set the current project.
Task 2. Create a Cloud Storage Bucket for Terraform State
Terraform uses a state file to track the resources it manages. For collaborative projects and increased reliability, it's best to store this state remotely in a Cloud Storage bucket.

Create a Cloud Storage bucket. The bucket name must be globally unique and should include your project ID as a prefix.

gsutil mb -l "REGION" gs://"PROJECT_ID"-tf-state
Copied!
Note:
Create the Cloud Storage bucket to store Terraform state.
Enable versioning on the bucket. This allows you to revert to previous states if necessary.

gsutil versioning set on gs://"PROJECT_ID"-tf-state
Copied!
Note:
Enable versioning for state file history.
Task 3. Create Terraform Configuration Files
Now, you will create the Terraform configuration files that define your GCE instance.

Create a file named main.tf with the following content:

terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
  backend "gcs" {
    bucket = ""PROJECT_ID"-tf-state"
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

    access_config {
    }
  }
}
Copied!
Note:
This file defines the Terraform provider, backend, and the GCE instance resource. A `variables.tf` configuration will be used to define the PROJECT_ID and REGION.
Create a file named variables.tf (optional, but recommended for defining variables):

variable "project_id" {
  type        = string
  description = "The ID of the Google Cloud project"
  default = ""PROJECT_ID""
}

variable "region" {
  type        = string
  description = "The region to deploy resources in"
  default     = ""REGION""
}

variable "zone" {
  type        = string
  description = "The zone to deploy resources in"
  default     = ""ZONE""
}
Copied!
Note:
This file defines variables for your project ID, region, and zone. Note that it contains defaults.
Task 4. Initialize, Plan, and Apply Terraform
With the configuration files created, you can now initialize, plan, and apply your Terraform configuration.

Initialize Terraform. This downloads the necessary provider plugins.

terraform init
Copied!
Note:
Initialize Terraform to download plugins.
Plan the changes. This shows you what Terraform will do before it makes any actual changes.

terraform plan
Copied!
Note:
Review the planned changes.
Apply the changes. This creates the GCE instance.

terraform apply -auto-approve
Copied!
Note:
Apply the Terraform configuration to create the instance. The `-auto-approve` flag automatically approves the changes. Be cautious when using this flag in production environments.
Task 5. Verify the Instance
Once Terraform has finished, verify that the GCE instance has been created.

In the Google Cloud Console, navigate to Compute Engine > VM instances. You should see an instance named 'terraform-instance'.

Alternatively, use the gcloud command to list instances.

gcloud compute instances list
Copied!
Note:
Verify the instance using the gcloud CLI.
Task 6. Destroy the Infrastructure
When you are finished, destroy the infrastructure to avoid incurring unnecessary costs.

Destroy the resources created by Terraform.

terraform destroy -auto-approve
Copied!
Note:
Destroy the resources created by Terraform.
