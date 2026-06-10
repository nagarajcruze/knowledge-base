# Terraform Modules & State Management

## 1. Terraform Modules

Modules are self-contained packages of Terraform configurations that manage related groups of infrastructure. They allow you to write dry (Don't Repeat Yourself), reusable, and organized code.

- **Root Module**: The directory containing the `.tf` files where you run `terraform apply`.
- **Child Modules**: External directories or remote registries called by the root module using a `module` block.

### Calling a Local Module
To call a local folder module, supply the relative filesystem path in the `source` parameter:
```hcl
module "vpc" {
  source       = "./modules/vpc"
  
  # Pass input variables required by the child module
  cidr_block   = "10.0.0.0/16"
  environment  = "production"
}
```

### Calling a Remote Registry Module
You can download verified modules from the official Terraform Registry:
```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"
  
  # reference an output exported by the vpc child module
  vpc_id  = module.vpc.vpc_id
}
```

---

## 2. Managing State

The state file (`terraform.tfstate`) is critical to how Terraform operates. By default, state is stored locally as a JSON file in your project directory. 

> [!CAUTION]
> Never commit `terraform.tfstate` or `terraform.tfstate.backup` to public or private version control systems (like Git). State files can contain unencrypted secret keys, database passwords, and environment data. Add them to `.gitignore`.

### Remote State Backends
In a team environment, local state files lead to conflicts. Configuring a **Remote Backend** stores the state database in a shared cloud storage service (like Amazon S3, Google Cloud Storage, or Terraform Cloud).

#### S3 Backend with DynamoDB Locking
By using S3 for storage and Amazon DynamoDB for locking, you ensure that:
1. The state database is encrypted and stored off-site.
2. Concurrent apply commands from different engineers are blocked (state locking) to prevent corrupting the state database.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-company-tf-state-bucket"
    key            = "production/network/terraform.tfstate"
    region         = "us-east-1"
    
    # DynamoDB Table must have a primary key named 'LockID' (type String)
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

### State Management Commands
If a resource gets out of sync or needs to be deleted from Terraform management without destroying the cloud resource, use the `state` utility CLI commands:

- **List all managed resources in the state file**:
  ```bash
  terraform state list
  ```
- **Show detailed state metadata for a specific resource**:
  ```bash
  terraform state show module.vpc.aws_vpc.main
  ```
- **Remove a resource from the state file** (forces Terraform to forget the resource; it won't be deleted in the cloud):
  ```bash
  terraform state rm aws_instance.web
  ```
- **Rename or move a resource in the state file**:
  ```bash
  terraform state mv aws_instance.web aws_instance.app_server
  ```
