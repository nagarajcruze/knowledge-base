# Terraform Modules & State

## Modules

Reusable packages of Terraform config.

```hcl
module "vpc" {
  source       = "./modules/vpc"
  cidr_block   = "10.0.0.0/16"
  environment  = "production"
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"
  vpc_id  = module.vpc.vpc_id
}
```

## Remote State

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

> **Important:** Never commit `terraform.tfstate` to version control.
