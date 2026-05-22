# Terraform Basics

Infrastructure as Code (IaC) tool.

## Core Workflow

```bash
terraform init      # Download providers
terraform plan      # Preview changes
terraform apply     # Apply changes
terraform destroy   # Tear down
```

## HCL Example

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  tags = { Name = "web-server" }
}

variable "instance_type" {
  default = "t3.micro"
}

output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

| Concept | Description |
|---------|-------------|
| **Provider** | Cloud/service API plugin |
| **Resource** | Infrastructure component |
| **Variable** | Parameterized input |
| **Output** | Values shown after apply |
| **State** | Config ↔ real-world mapping |
