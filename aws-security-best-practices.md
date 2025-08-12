# AWS Security Best Practices Demo using Terraform & Bash

## Introduction
Provision a secure AWS architecture using Terraform that includes:
-EC2 instance (Ubuntu, hardened with Bash script)
-Custom SSH key pair generation
-S3 bucket with encryption and policies
-IAM roles and policies for least-privilege access
- Region: us-east-1


Project Structure (GitHub-friendly)

aws-security-best-practices-demo/
├── main.tf
├── variables.tf
├── provider.tf
├── outputs.tf
├── user-data.sh
├── keypair.pem (auto-generated)
├── README.md


---


### 1. Create directory and cd to that directory
a. mkdir name of directory
b. cd the name of your directory


---


### 2. Create  **configuration files**

a. Create **provider.tf** file and the following below: 

```
provider "aws" {
  region = var.aws_region
}
```
**Note**
Specifies AWS as the provider and uses a variable for region (us-east-1).


<img width="653" height="213" alt="Image" src="https://github.com/user-attachments/assets/0f836519-804a-4701-bb58-2540c97a5bb3" />


b. Create **main.tf** file and the following below: 

```
resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "generated_key" {
  key_name   = var.key_name
  public_key = tls_private_key.ec2_key.public_key_openssh
}

resource "local_file" "private_key" {
  content  = tls_private_key.ec2_key.private_key_pem
  filename = "${var.key_name}.pem"
  file_permission = "0600"
}

resource "aws_iam_role" "ec2_role" {
  name = "ec2-secure-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = "sts:AssumeRole",
        Effect = "Allow",
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "ec2_policy" {
  name = "ec2-secure-policy"
  role = aws_iam_role.ec2_role.id

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = [
          "s3:ListBucket",
          "s3:GetObject",
          "s3:PutObject"
        ],
        Effect   = "Allow",
        Resource = "*"
      }
    ]
  })
}

resource "aws_instance" "secure_ec2" {
  ami           = "ami-07d9b9ddc6cd8dd30" # Ubuntu 22.04 in us-east-1
  instance_type = var.instance_type
  key_name      = aws_key_pair.generated_key.key_name
  user_data     = file("user-data.sh")

  iam_instance_profile = aws_iam_instance_profile.ec2_profile.name

  tags = {
    Name = "SecureEC2Instance"
  }
}

resource "aws_iam_instance_profile" "ec2_profile" {
  name = "ec2-secure-profile"
  role = aws_iam_role.ec2_role.name
}

resource "aws_s3_bucket" "secure_bucket" {
  bucket = var.bucket_name

  tags = {
    Name = "SecureS3Bucket"
  }
}

resource "aws_s3_bucket_versioning" "bucket_versioning" {
  bucket = aws_s3_bucket.secure_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "bucket_encryption" {
  bucket = aws_s3_bucket.secure_bucket.bucket

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

**Note**
Generates a new SSH key pair locally.
Creates an EC2 instance (Ubuntu 22.04).
Attaches IAM role with S3 permissions.
Creates a secure S3 bucket with versioning and AES256 encryption.


<img width="557" height="711" alt="Image" src="https://github.com/user-attachments/assets/779ca2b8-0d53-404f-9058-c7eb5b0ffb15" />


c. Create and configure **variables.tf** file
```
variable "aws_region" {
  default = "us-east-1"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "bucket_name" {
  description = "Name of the S3 bucket"
  default     = "secure-s3-bucket-demo-001"
}

variable "key_name" {
  description = "Name of the SSH key pair"
  default     = "secure-demo-key"
}
```

**Note**
Declares all values you can easily update, like region, EC2 type, and names.


<img width="675" height="485" alt="Image" src="https://github.com/user-attachments/assets/b1cd0d32-fcf4-4067-a8ff-5beef73080c4" />


d. Create and Configure Outputs.tf file
```
output "ec2_public_ip" {
  value = aws_instance.secure_ec2.public_ip
}

output "ssh_command" {
  value = "ssh -i ${var.key_name}.pem ubuntu@${aws_instance.secure_ec2.public_ip}"
}
```

**Note**
Outputs the public IP and ready-to-use SSH command.


<img width="710" height="269" alt="Image" src="https://github.com/user-attachments/assets/2a58d053-c27e-4aba-9f05-5f91af4a2964" />



e. Create and Configure **user-data.s**h file
```
#!/bin/bash
sudo apt-get update -y
sudo apt-get upgrade -y

# Basic hardening
sudo ufw allow OpenSSH
sudo ufw enable
sudo apt-get install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```
Updates the OS, enables a firewall, installs Fail2Ban for brute-force protection.


<img width="680" height="309" alt="Image" src="https://github.com/user-attachments/assets/389d8e76-44d1-46fc-bf2b-5ded5998ca32" />


---

### 2. Initialize Terraform
Run

```
terraform init
```


<img width="625" height="723" alt="Image" src="https://github.com/user-attachments/assets/045a8768-a2a1-49bf-9252-783c5216a045" />


---

### 3. Validate & Plan

Run

```
terraform validate
```


<img width="410" height="95" alt="Image" src="https://github.com/user-attachments/assets/d41b0ab5-cc8a-4390-a1cc-b6955b9f9590" />


Run

```
terraform plan
```


<img width="625" height="725" alt="Image" src="https://github.com/user-attachments/assets/b63f4a25-b9cb-4409-b822-7bd04052050c" />


---

### 4. Deploy Resources


```
terraform apply
```
Type and enter **yes** when prompted.


<img width="612" height="725" alt="Image" src="https://github.com/user-attachments/assets/5a0486f8-3296-4046-90e0-a9beaa6005e2" />


---

### 5. Take note of the output 
Terraform will print the following:

ec2_public_ip = "IP address"

ssh command = = "ssh command"

Copy the IP address. Open your browser, paste the IP address and press the Enter key:


<img width="407" height="74" alt="Image" src="https://github.com/user-attachments/assets/f472ab10-35e3-4ec8-a936-66d61956befc" />




---

###. 6. Confirm resources provisioned on terraform

Run

```
terraform state list
```

<img width="562" height="343" alt="Image" src="https://github.com/user-attachments/assets/39375dac-ab0d-45fd-95d6-ca042b8972ec" />


```
aws s3 ls
```

<img width="987" height="144" alt="Image" src="https://github.com/user-attachments/assets/2775a9a7-bf78-40d3-91b0-eeb1a9f555c7" />


---

### 7. Cleanup (Avoid Charges)
a. To destroy all provisioned infrastructure:


Run
```
terraform destroy
```



b. Run the following to confirm
bash
```
terraform state list
```
and
```
terraform show
```




---

###. 6. Confirm resources provisioned and destroyed from the Console

![Image](https://github.com/user-attachments/assets/b68fe000-c52c-4fd1-97ad-c5c298c79bc7)


![Image](https://github.com/user-attachments/assets/0b7bfa99-52bc-4593-a91e-5d661f91f425)


![Image](https://github.com/user-attachments/assets/2b0f9372-8b74-4a04-851b-3e106754d5a8)


![Image](https://github.com/user-attachments/assets/8a1230fc-9254-489c-8a35-279154172a7e)


![Image](https://github.com/user-attachments/assets/e5b502ad-aea4-4e91-a951-1852e97e2085)


![Image](https://github.com/user-attachments/assets/ab0ff83c-6995-4890-af93-730c8683e418)


![Image](https://github.com/user-attachments/assets/007439fd-220f-4df4-89db-8ffc8c0096ab)


![Image](https://github.com/user-attachments/assets/80a71bef-8b00-4996-b27a-b8d0c06c0c75)


![Image](https://github.com/user-attachments/assets/3b8224f0-85d7-4aa4-a707-43784792136d)


![Image](https://github.com/user-attachments/assets/f2226b4b-0b6d-4a3f-9d14-52232a006bd6)
