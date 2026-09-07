# Terraform Blocks

Terraform uses **blocks** as the main structural elements of a configuration file. Blocks define infrastructure resources, providers, variables, outputs, modules, and other Terraform configurations.

## Basic Block Syntax

```hcl
block_type "label1" "label2" {

  argument = value

  nested_block {
    argument = value
  }
}
```

---

# Types of Terraform Blocks

The most commonly used Terraform blocks are:

1. **terraform**
2. **provider**
3. **resource**
4. **data**
5. **variable**
6. **output**
7. **locals**
8. **module**
9. **dynamic**

---

## 1. Terraform Block

The `terraform` block configures Terraform itself.

It is commonly used to specify:

* Required Terraform version
* Required providers
* Provider versions

### Example

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### Explanation

* `terraform` → Defines Terraform configuration.
* `required_version` → Specifies the Terraform version requirement.
* `required_providers` → Defines which providers Terraform needs.
* `aws` → AWS provider.
* `source` → Provider source.
* `version` → Required provider version.

---

# 2. Provider Block

A `provider` block configures the cloud or service provider Terraform will use.

For example:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

### Explanation

* `provider` → Defines a provider.
* `"aws"` → Provider name.
* `region` → AWS region where resources will be created.

---

# 3. Resource Block

The `resource` block is used to create and manage infrastructure.

### Example

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

### Syntax

```hcl
resource "RESOURCE_TYPE" "RESOURCE_NAME" {

  argument = value

}
```

### Explanation

```text
aws_instance
     ↓
Resource Type

web
 ↓
Resource Name
```

Terraform identifies the resource using:

```text
aws_instance.web
```

For example:

```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

---

# 4. Data Block

The `data` block is used to **read existing information** from a provider.

It does not create a new resource.

### Example

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true

  owners = ["099720109477"]
}
```

### Explanation

* `data` → Reads existing information.
* `aws_ami` → AWS AMI data source.
* `ubuntu` → Local name used to reference the data.
* `most_recent = true` → Gets the latest matching AMI.
* `owners` → Specifies the AMI owner.

---

# 5. Variable Block

The `variable` block is used to create **input variables**.

Variables make Terraform configurations reusable.

### Example

```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```

The variable can be used like this:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = var.instance_type
}
```

### Explanation

* `variable` → Defines an input variable.
* `"instance_type"` → Variable name.
* `type = string` → Variable accepts text.
* `default` → Default value.

---

# 6. Output Block

The `output` block displays useful information after Terraform creates infrastructure.

### Example

```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

After:

```bash
terraform apply
```

Terraform can display:

```text
instance_id = "i-0123456789abcdef"
```

### Common Output Examples

```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

```hcl
output "public_ip" {
  value = aws_instance.web.public_ip
}
```

```hcl
output "instance_arn" {
  value = aws_instance.web.arn
}
```

---

# 7. Locals Block

The `locals` block defines values that can be reused inside Terraform configuration.

### Example

```hcl
locals {
  environment = "dev"
  app_name    = "webapp"
}
```

You can use them with:

```hcl
resource "aws_instance" "web" {

  tags = {
    Name        = local.app_name
    Environment = local.environment
  }

}
```

### Important

For variables:

```hcl
var.instance_type
```

For locals:

```hcl
local.app_name
```

---

# 8. Module Block

The `module` block is used to call a **Terraform module**.

Modules help us create reusable Terraform configurations.

### Example

```hcl
module "vpc" {

  source = "./modules/vpc"

  cidr_block = "10.0.0.0/16"

}
```

### Explanation

* `module` → Defines a module.
* `"vpc"` → Module name.
* `source` → Location of the module.
* `cidr_block` → Input passed to the module.

Example folder structure:

```text
terraform-project/
│
├── main.tf
│
└── modules/
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

# 9. Dynamic Block

The `dynamic` block is used when we need to create multiple nested blocks dynamically.

### Example

```hcl
dynamic "ingress" {

  for_each = var.ingress_rules

  content {

    from_port   = ingress.value.port
    to_port     = ingress.value.port
    protocol    = "tcp"
    cidr_blocks = ingress.value.cidr_blocks

  }
}
```

`dynamic` blocks are especially useful when working with multiple:

* Security group rules
* Firewall rules
* Network rules
* Similar nested configurations

---

# Blocks vs Arguments

This is very important in Terraform.

### Argument

An argument is a **key-value pair**.

```hcl
bucket = "my-bucket"
```

Here:

```text
bucket  → argument name
my-bucket → value
```

### Block

A block contains a group of configuration settings.

```hcl
resource "aws_s3_bucket" "example" {

  bucket = "my-bucket"

}
```

Here:

```text
resource
   ↓
Block

bucket = "my-bucket"
   ↓
Argument
```

---

# Nested Blocks

A block can contain another block.

Example:

```hcl
resource "aws_security_group" "web" {

  name = "web-sg"

  ingress {

    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]

  }

}
```

Here:

```text
resource
   │
   ├── name = "web-sg"       → Argument
   │
   └── ingress { }           → Nested Block
           │
           ├── from_port     → Argument
           ├── to_port       → Argument
           ├── protocol      → Argument
           └── cidr_blocks   → Argument
```

---

# Terraform Block Structure

A typical Terraform project can contain:

```text
Terraform Configuration
│
├── terraform
│
├── provider
│
├── variable
│
├── locals
│
├── data
│
├── resource
│
├── module
│
└── output
```

---

# Quick Reference

| Block       | Purpose                                    |
| ----------- | ------------------------------------------ |
| `terraform` | Configure Terraform and required providers |
| `provider`  | Configure cloud/service provider           |
| `resource`  | Create/manage infrastructure               |
| `data`      | Read existing information                  |
| `variable`  | Define input variables                     |
| `output`    | Display useful values                      |
| `locals`    | Define reusable local values               |
| `module`    | Use reusable Terraform modules             |
| `dynamic`   | Generate nested blocks dynamically         |

---

# Simple Terraform Example

Here is a simple example combining multiple blocks:

```hcl
terraform {

  required_providers {

    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }

  }

}

provider "aws" {
  region = "us-east-1"
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

locals {
  environment = "dev"
}

resource "aws_instance" "web" {

  ami           = "ami-12345678"
  instance_type = var.instance_type

  tags = {
    Name        = "web-server"
    Environment = local.environment
  }

}

output "instance_id" {
  value = aws_instance.web.id
}
```

## Important Points

* **Block** → Defines a section of Terraform configuration.
* **Argument** → Assigns a value.
* **Nested block** → A block inside another block.
* **Label** → Identifies a block, such as `aws_instance` and `web`.
* **Reference** → Allows one Terraform object to use another.

Example:

```hcl
var.instance_type
```

```hcl
local.environment
```

```hcl
aws_instance.web.id
```

