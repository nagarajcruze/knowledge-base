# Terraform: Infrastructure as Code Basics

## 1. Introduction to Terraform

Terraform is an open-source **Infrastructure as Code (IaC)** tool developed by HashiCorp. It allows you to define, provision, and configure cloud and on-premises physical resources using a declarative configuration language known as **HashiCorp Configuration Language (HCL)**.

### Key Concepts

- **Declarative Approach**: You define the *desired state* of your infrastructure (e.g., "I want 3 web servers and 1 database"). Terraform figures out the dependencies and steps required to build it.
- **State File (`terraform.tfstate`)**: A JSON database file where Terraform records mappings between your HCL code declarations and the real-world resources deployed in the cloud. It acts as the source of truth for planning modifications.
- **Providers**: Plugins that translate HCL declarations into API calls for specific platforms (e.g., AWS, Azure, Google Cloud, Docker, GitHub).

---

## 2. Core Workflow Commands

### The standard workflow consists of four phases:
1. **Write**: Author your infrastructure code in `.tf` files.
2. **Init**: Initialize the working directory.
3. **Plan**: Review the execution plan of what changes will be applied.
4. **Apply**: Execute the changes to provision the infrastructure.

### Command Reference
- **`terraform init`**: Downloads the required provider plugins and initializes the backend configuration. Run this first when starting a project or adding providers.
- **`terraform validate`**: Checks the syntax of configurations for errors before planning.
- **`terraform fmt`**: Rewrites configuration files to follow the canonical formatting and alignment standards.
- **`terraform plan`**: Performs a dry-run to show exactly what resources will be created, updated, or destroyed.
- **`terraform apply`**: Executes the changes required to reach the desired state. By default, it prompts for manual confirmation (`yes`).
  - *Auto-approve*: `terraform apply -auto-approve` (skips the confirmation prompt).
- **`terraform destroy`**: Removes all infrastructure resources managed by the current Terraform configuration.
- **`terraform state list`**: Lists all resources currently tracked in the state file.

### Installation (Linux/Debian)
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

---

## 3. HCL Block Syntax

An HCL configuration file consists of block types:

```hcl
# 1. Provider configuration
provider "aws" {
  region = var.aws_region
}

# 2. Local variables
locals {
  service_name = "web-portal"
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# 3. Data Source (look up resources existing in the cloud)
data "aws_ami" "ubuntu" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  owners = ["099720109477"] # Canonical
}

# 4. Resource definition
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  tags          = local.common_tags
}
```

---

## 4. Input Variables (`variable`)

Variables allow you to parameterize your configurations, making them reusable across environments (e.g. dev, prod).

### Declaring Variables
```hcl
variable "aws_region" {
  type        = string
  description = "The target AWS region for deployment"
  default     = "us-east-1"
}

variable "instance_type" {
  type        = string
  description = "EC2 instance size"
  default     = "t3.micro"
}

variable "port_configurations" {
  type        = list(number)
  description = "Allowed network ports"
  default     = [22, 80, 443]
}
```

### Assigning Variable Values
You can pass values into variables in several ways, ordered here from **lowest to highest precedence**:

1. **Default values**: Fallback values configured inside the `variable` block.
2. **Environment Variables**: Prefixed with `TF_VAR_`.
   *Example*: `export TF_VAR_instance_type="t3.medium"`
3. **The `terraform.tfvars` file**: A file in the project root containing variable assignments:
   ```hcl
   # terraform.tfvars
   aws_region    = "us-west-2"
   instance_type = "t3.small"
   ```
4. **Any `*.auto.tfvars` files**: Automatically loaded files.
5. **Command Line Flags**: Passed directly to CLI commands (overrides all other assignments).
   *Example*: `terraform apply -var="instance_type=t3.large"`

---

## 5. Local Values (`locals`)

Local values act as temporary internal variables. Unlike input variables, users cannot set them directly. They are computed dynamically inside your code to prevent repeating complex calculations.

### Declaration & Reference
```hcl
locals {
  # Concatenate project prefix with environment name
  resource_prefix = "${var.project_name}-${var.environment}"
}

# Access locals using the 'local.' prefix
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "${local.resource_prefix}-vpc"
  }
}
```

---

## 6. Data Sources (`data`)

Data sources allow Terraform to fetch data from external systems or query resources that were created outside of Terraform.

### Usage Example
To spin up an instance using the latest Amazon Linux 2 AMI without hardcoding the ID:
```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "app_server" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
}
```

---

## 7. Outputs (`output`)

Outputs are values exposed by Terraform after an apply operation. They are printed on the terminal console and can be used to pass details to other scripts or configurations.

```hcl
output "web_server_public_ip" {
  value       = aws_instance.web.public_ip
  description = "The public IP address of the main web server"
}

output "database_password" {
  value       = aws_db_instance.db.password
  description = "The master password for the database"
  sensitive   = true # Hides the value from stdout prints to prevent logging secrets
}
```
*Note: Setting `sensitive = true` prevents the value from displaying in stdout, but the value is still saved in plain text in the `terraform.tfstate` file.*
