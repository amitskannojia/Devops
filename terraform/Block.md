1. terraform Block

Configures Terraform itself, such as required providers and version constraints.

terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }          
  }
}
2. provider Block

Configures the provider used to manage infrastructure.

provider "aws" {
  region = "us-east-1"
}
3. resource Block

Defines an infrastructure resource.

resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
4. data Block

Retrieves information from existing infrastructure.

data "aws_ami" "ubuntu" {
  most_recent = true

  owners = ["099720109477"]
}
5. variable Block

Declares input variables.

variable "instance_type" {
  type    = string
  default = "t2.micro"
}
6. output Block

Displays values after terraform apply.

output "instance_id" {
  value = aws_instance.web.id
}
7. locals Block

Defines local values for reuse.

locals {
  environment = "dev"
  app_name    = "webapp"
}
8. module Block

Uses a reusable Terraform module.

module "vpc" {
  source = "./modules/vpc"

  cidr_block = "10.0.0.0/16"
}
9. dynamic Block

Generates nested blocks dynamically.

dynamic "ingress" {
  for_each = var.ingress_rules

  content {
    from_port   = ingress.value.port
    to_port     = ingress.value.port
    protocol    = "tcp"
    cidr_blocks = ingress.value.cidr_blocks
  }
}

Types of Blocks
Top-level blocks: terraform, provider, resource, module, variable, output, locals, data
Nested blocks: Blocks defined inside another block (e.g., required_providers, lifecycle, provisioner, connection, content)
Blocks vs. Arguments

A block groups related configuration:

resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"    # Argument

  versioning {            # Nested block
    enabled = true
  }
}
bucket = "my-bucket" is an argument (name-value pair).
versioning { ... } is a nested block (contains its own configuration).
