# Terraform Remote State Storage & Locking
##SELF NOTE:
1)By storing the state file in remote we can give aacess to all developers to perform opearations, if you store it in local we can'target
2)for this we need a remote backen storage, in AWS s3 act as a remote backend
3)here we have two types backed ends *)enchanced backedns(means can to both storage and performing the operations ie.local and remote) (eg for remote backends:terraform cloud,terraform enterprise)
**)standard backends(means to can do only stor the sate and it will relay on the local backends to perform opetions----(eg: aws s3,azure rm,etcd,...)
4)to resolve conflict operation like multiple developers doing operations at a time we have to use state lock using dynamo db
## Step-01: Introduction
- Understand Terraform Backends
- Understand about Remote State Storage and its advantages
- This state is stored by default in a local file named "terraform.tfstate", but it can also be stored remotely, which works better in a team environment.
- Create AWS S3 bucket to store `terraform.tfstate` file and enable backend configurations in terraform settings block
- Understand about **State Locking** and its advantages
- Create DynamoDB Table and  implement State Locking by enabling the same in Terraform backend configuration

## Step-02: Create S3 Bucket
- Go to Services -> S3 -> Create Bucket
- **Bucket name:** terraform-stacksimplify
- **Region:** US-East (N.Virginia)
- **Bucket settings for Block Public Access:** leave to defaults
- **Bucket Versioning:** Enable
- Rest all leave to **defaults**
- Click on **Create Bucket**
- **Create Folder**
  - **Folder Name:** dev
  - Click on **Create Folder**


## Step-03: Terraform Backend Configuration
- **Reference Sub-folder:** terraform-manifests
- [Terraform Backend as S3](https://www.terraform.io/docs/language/settings/backends/s3.html)
- Add the below listed Terraform backend block in `Terrafrom Settings` block in `main.tf`
```t
# Terraform Backend Block
  backend "s3" {
    bucket = "terraform-stacksimplify"
    key    = "dev/terraform.tfstate"
    region = "us-east-1"    
  }
```

## Step-04: Test with Remote State Storage Backend
```t
# Initialize Terraform
terraform init

Observation: 
Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Validate Terraform configuration files
terraform validate

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Format Terraform configuration files
terraform fmt

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Review the terraform plan
terraform plan 

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Create Resources 
terraform apply -auto-approve

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate
Observation: Finally at this point you should see the terraform.tfstate file in s3 bucket

# Access Application
http://<Public-DNS>
```

## Step-05: Bucket Versioning - Test
- Update in `c2-variables.tf` 
```t
variable "instance_type" {
  description = "EC2 Instance Type - Instance Sizing"
  type = string
  #default = "t2.micro"
  default = "t2.small"
}
```
- Execute Terraform Commands
```t
# Review the terraform plan
terraform plan 

# Create Resources 
terraform apply -auto-approve

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate
Observation: New version of terraform.tfstate file will be created
```


## Step-06: Destroy Resources
- Destroy Resources and Verify Bucket Versioning
```t
# Destroy Resources
terraform destroy -auto-approve
```
## Step-07: Terraform State Locking Introduction
- Understand about Terraform State Locking Advantages

## Step-08: Add State Locking Feature using DynamoDB Table
- Create Dynamo DB Table
  - **Table Name:** terraform-dev-state-table
  - **Partition key (Primary Key):** LockID (Type as String)
  - **Table settings:** Use default settings (checked)
  - Click on **Create**

## Step-09: Update DynamoDB Table entry in backend in c1-versions.tf
- Enable State Locking in `c1-versions.tf`
```t
  # Adding Backend as S3 for Remote State Storage with State Locking
  backend "s3" {
    bucket = "terraform-stacksimplify"
    key    = "dev2/terraform.tfstate"
    region = "us-east-1"  

    # For State Locking
    dynamodb_table = "terraform-dev-state-table"
  }
```

## Step-10: Execute Terraform Commands
```t
# Initialize Terraform 
terraform init

# Review the terraform plan
terraform plan 
Observation: 
1) Below messages displayed at start and end of command
Acquiring state lock. This may take a few moments...
Releasing state lock. This may take a few moments...
2) Verify DynamoDB Table -> Items tab

# Create Resources 
terraform apply -auto-approve

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev2/terraform.tfstate
Observation: New version of terraform.tfstate file will be created in dev2 folder
```

## Step-11: Destroy Resources
- Destroy Resources and Verify Bucket Versioning
```t
# Destroy Resources
terraform destroy -auto-approve

# Clean-Up Files
rm -rf .terraform*
rm -rf terraform.tfstate*  # This step not needed as e are using remote state storage here
```

## References 
- [AWS S3 Backend](https://www.terraform.io/docs/language/settings/backends/s3.html)
- [Terraform Backends](https://www.terraform.io/docs/language/settings/backends/index.html)
- [Terraform State Storage](https://www.terraform.io/docs/language/state/backends.html)
- [Terraform State Locking](https://www.terraform.io/docs/language/state/locking.html)
- [Remote Backends - Enhanced](https://www.terraform.io/docs/language/settings/backends/remote.html)
