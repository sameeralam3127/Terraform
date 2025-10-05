## Learn Terraform with LocalStack — No AWS Account Needed!

- Write a simple **Terraform script (`main.tf`)**
- Use **LocalStack** as a fake AWS
- Verify infrastructure with the **AWS CLI**

---

## Step 1: Prerequisites

Before starting, install the following tools:

| Tool       | Purpose                           | Command (Linux/Mac example)                         |
| ---------- | --------------------------------- | --------------------------------------------------- |
| Docker     | Runs LocalStack container         | `sudo apt install docker.io`                        |
| Terraform  | Infrastructure as Code            | `brew install terraform` or download from HashiCorp |
| AWS CLI    | Test and interact with LocalStack | `brew install awscli`                               |
| LocalStack | Fake AWS services locally         | `pip install localstack`                            |

> **Tip:** Check installation with
> `terraform -v`, `aws --version`, and `localstack --version`.

---

## Step 2: Start LocalStack

Run this in your terminal:

```bash
localstack start
```

You should see:

```
LocalStack version x.x.x
Ready.
```

LocalStack runs on:

```
http://localhost:4566
```

That’s your **fake AWS endpoint**.

---

## Step 3: Create the Terraform configuration

Create a new folder:

```bash
mkdir terraform-localstack-demo
cd terraform-localstack-demo
```

Then create a file named **`main.tf`**:

---

### `main.tf`

```hcl
##############################################
# Terraform + LocalStack Demo
# Author: Your Name
# Description: Create a simple S3 bucket locally
##############################################

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# --- Provider Configuration ---
provider "aws" {
  # Dummy credentials for LocalStack (it doesn't check them)
  access_key                  = "test"
  secret_key                  = "test"
  region                      = "us-east-1"

  # Important: These settings tell Terraform to connect to LocalStack
  s3_use_path_style           = true
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true

  # Point all AWS API calls to LocalStack
  endpoints {
    s3 = "http://localhost:4566"
  }
}

# --- Create a local S3 bucket ---
resource "aws_s3_bucket" "demo_bucket" {
  bucket = "localstack-demo-bucket"
  tags = {
    Name        = "LocalStack Bucket"
    Environment = "Learning"
  }
}

# --- Output value ---
output "bucket_name" {
  value = aws_s3_bucket.demo_bucket.bucket
}
```

---

## Step 4: Understand the configuration

| Block                       | Description                                                                        |
| --------------------------- | ---------------------------------------------------------------------------------- |
| **terraform**               | Defines provider requirements.                                                     |
| **provider "aws"**          | Configures Terraform to use AWS — but points it to LocalStack instead of real AWS. |
| **access_key / secret_key** | Fake credentials (LocalStack doesn’t check them).                                  |
| **endpoints**               | The key part — tells Terraform to send API calls to `http://localhost:4566`.       |
| **resource**                | Creates an S3 bucket inside LocalStack.                                            |
| **output**                  | Prints the bucket name after creation.                                             |

---

## Step 5: Initialize and apply Terraform

Run these commands:

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

Expected output:

```
Plan: 1 to add, 0 to change, 0 to destroy.
aws_s3_bucket.demo_bucket: Creation complete after 0s
Apply complete! Resources: 1 added.
```

🎉 Congrats — you just created an **S3 bucket locally**, not in AWS!

---

## Step 6: Verify with AWS CLI

Run this command to see your S3 buckets inside LocalStack:

```bash
aws --endpoint-url=http://localhost:4566 s3 ls
```

Expected output:

```
2025-10-05  localstack-demo-bucket
```

If you want to upload a file:

```bash
echo "hello world" > hello.txt
aws --endpoint-url=http://localhost:4566 s3 cp hello.txt s3://localstack-demo-bucket/
```

Check contents:

```bash
aws --endpoint-url=http://localhost:4566 s3 ls s3://localstack-demo-bucket/
```

---

## Step 7: Clean up

Destroy your fake AWS infrastructure:

```bash
terraform destroy -auto-approve
```

Stop LocalStack:

```bash
localstack stop
```

---

## Test Phase Summary

| Step             | Command                                          | What It Does                |
| ---------------- | ------------------------------------------------ | --------------------------- |
| Start LocalStack | `localstack start`                               | Launches local AWS emulator |
| Init Terraform   | `terraform init`                                 | Downloads providers         |
| Plan             | `terraform plan`                                 | Previews changes            |
| Apply            | `terraform apply`                                | Creates resources           |
| Test             | `aws --endpoint-url=http://localhost:4566 s3 ls` | Verifies bucket creation    |
| Clean            | `terraform destroy`                              | Deletes resources           |

---

## Next Steps

Once comfortable:

- Add more services (Lambda, DynamoDB, SQS)
- Store Terraform **state** in a file or remote backend
- Try real AWS Free Tier (just update provider to real AWS endpoint)

---
