# Setting up HPE Hybrid Cloud on AWS with Terraform

**Task 1: Manual EC2 Instance Setup with Terraform**

1. **Launch EC2 Instance**:
   - **AWS Console**:
     - Region: `us-east-1`
   - **Instance Configuration**:
     - Name: `"terraform-aws"`
     - OS: `Ubuntu 22.04`
     - Type: `t2.large`
     - KeyPair: `terraform-aws-KeyPair`
     - Security Group: `terraform-aws-sg` with ports `22 (SSH)` and `80 (HTTP)` allowed.
     - Storage: `12 GiB`
   - Use the following script in the Userdata section to install Terraform directly on the server:

```bash
#!/bin/bash

# Change the HostName
sudo hostnamectl set-hostname terraform-aws

# Update and upgrade packages, install AWS CLI, and Terraform
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install python3-pip -y
sudo apt-get install -y awscli

# Install the latest version of Terraform
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update -y
sudo apt-get install -y terraform

# Verify installations
aws --version
terraform version
```

**Task 2: Configure AWS CLI**

Configure AWS CLI:

```shell
aws configure
```

Provide your credentials, including the `Access Key` and `Secret Access Key`.

```shell
aws s3 ls
```

**Task 3: Create a Working Directory and Write Terraform Configuration**

```bash
mkdir terraform-aws
cd terraform-aws
```

Create `main.tf` and add your Terraform configuration:

```shell
vi main.tf
```

```hcl
# Set the AWS provider configuration
provider "aws" {
  region = "us-east-1"  # Change this to your desired AWS region
}

# Create 3 EC2 instances for Kubernetes (Change instance details as needed)
resource "aws_instance" "kubernetes" {
  count         = 3
  ami           = "ami-053b0d53c279acc90"  # Replace with your desired AMI
  instance_type = "t2.large"  # Adjust instance type as needed
  key_name      = "terraform-aws-KeyPair"  # Replace with your SSH key name
  tags = {
    Name = "kubernetes-instance-${count.index + 1}"
  }
}

# Create an AWS VPN
resource "aws_vpn_gateway" "vpn" {
  # Configure VPN options as needed
  tags = {
    Name = "vpn-gateway"
  }
}

# Create an S3 bucket
resource "aws_s3_bucket" "example" {
  bucket = "aws-modules-bucket"
}

resource "aws_s3_bucket_ownership_controls" "example" {
  bucket = aws_s3_bucket.example.id
  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

resource "aws_s3_bucket_public_access_block" "example" {
  bucket = aws_s3_bucket.example.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_acl" "example" {
  depends_on = [
    aws_s3_bucket_ownership_controls.example,
    aws_s3_bucket_public_access_block.example,
  ]

  bucket = aws_s3_bucket.example.id
  acl    = "public-read"
}
```

**To Apply Configuration, Run the Following Commands:**

```bash
terraform init
```

```bash
terraform fmt
```

```bash
terraform validate
```

```bash
terraform plan
```

```bash
terraform apply
```

```bash
aws ec2 describe-instances
```

---

**Task 4: Optional - Destroy Resources** (If the resources are no longer needed)

If you no longer need the AWS resources and want to clean up, you can use Terraform to destroy them:

```bash
terraform destroy
```

Be cautious when using this command, as it will delete the specified resources. Confirm by typing `"yes"` when prompted.

**Task 5: Cleanup**

After destroying the resources (if needed), it's essential to remove your Terraform state files. Run the following commands:

```bash
terraform state rm aws_instance.kubernetes
```

```bash
terraform state rm aws_vpn_gateway.vpn
```

```bash
terraform state rm aws_s3_bucket.example
```

**Note:** Replace the resource names as needed. This removes the resources from the Terraform state. Afterward, you can safely delete your Terraform working directory.

---
