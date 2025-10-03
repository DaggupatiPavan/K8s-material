# Terraform - Infrastructure as Code (IaC)

## Overview

Terraform is an open-source infrastructure as code (IaC) software tool created by HashiCorp. It enables users to define and provision infrastructure using a declarative configuration language. Terraform manages external resources (such as public cloud infrastructure, private cloud infrastructure, network appliances, etc.) with a high-level configuration syntax.

### Key Features
- **Infrastructure as Code**: Define infrastructure in configuration files
- **Execution Plans**: Preview changes before applying them
- **Resource Graph**: Understand resource dependencies
- **Change Automation**: Apply changes with minimal manual intervention
- **Multi-Cloud**: Support for multiple cloud providers
- **State Management**: Track infrastructure state
- **Modular Design**: Reusable infrastructure modules

## Core Concepts

### Terraform Architecture
- **Configuration Files**: HCL (HashiCorp Configuration Language) files
- **Providers**: Plugins that interact with APIs of services
- **Resources**: Infrastructure components (VMs, networks, etc.)
- **State**: JSON file that maps resources to configuration
- **Backend**: Storage for Terraform state
- **Modules**: Reusable collections of resources
- **Variables**: Input parameters for configurations
- **Outputs**: Return values from configurations

### Key Components
- **Terraform CLI**: Command-line interface
- **HCL**: Declarative configuration language
- **Providers**: AWS, Azure, GCP, etc.
- **State Management**: Local and remote state
- **Workspace**: Multiple environments
- **Data Sources**: Read existing infrastructure
- **Provisioners**: Execute scripts on resources

### Terraform Workflow
1. **Write**: Define infrastructure in HCL
2. **Plan**: Preview changes
3. **Apply**: Create or update infrastructure
4. **Destroy**: Remove infrastructure

## Installation & Setup

### Prerequisites
- **Operating System**: Windows, macOS, Linux
- **Memory**: Minimum 512MB RAM (1GB recommended)
- **Disk**: Minimum 100MB free space
- **Network**: Internet connection for provider downloads

### Installation Methods

#### Using Package Manager (Ubuntu/Debian)
```bash
# Add HashiCorp GPG key
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

# Add HashiCorp repository
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

# Update package list
sudo apt update

# Install Terraform
sudo apt install terraform

# Verify installation
terraform --version
```

#### Using Package Manager (CentOS/RHEL)
```bash
# Add HashiCorp repository
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo

# Install Terraform
sudo yum install terraform

# Verify installation
terraform --version
```

#### Using Homebrew (macOS)
```bash
# Install Terraform
brew install terraform

# Verify installation
terraform --version
```

#### Using Chocolatey (Windows)
```powershell
# Install Terraform
choco install terraform

# Verify installation
terraform --version
```

#### Manual Installation
```bash
# Download Terraform
wget https://releases.hashicorp.com/terraform/1.1.9/terraform_1.1.9_linux_amd64.zip

# Extract Terraform
unzip terraform_1.1.9_linux_amd64.zip

# Move to PATH
sudo mv terraform /usr/local/bin/

# Verify installation
terraform --version
```

## Terraform Configuration

### Basic Configuration Structure
```
my-project/
├── main.tf           # Main configuration file
├── variables.tf      # Variable definitions
├── outputs.tf        # Output definitions
├── terraform.tfvars  # Variable values
├── provider.tf       # Provider configuration
└── modules/          # Custom modules
```

### Basic Configuration Example
```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}

output "instance_public_ip" {
  value = aws_instance.web_server.public_ip
}
```

### Provider Configuration
```hcl
# provider.tf
provider "aws" {
  region = "us-west-2"
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
}

provider "azurerm" {
  features {}
  subscription_id = var.azure_subscription_id
  tenant_id       = var.azure_tenant_id
}

provider "google" {
  project = var.gcp_project
  region  = var.gcp_region
}
```

### Variable Definitions
```hcl
# variables.tf
variable "aws_access_key" {
  description = "AWS access key"
  type        = string
  sensitive   = true
}

variable "aws_secret_key" {
  description = "AWS secret key"
  type        = string
  sensitive   = true
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod"
  }
}

variable "ports" {
  description = "List of ports to open"
  type        = list(number)
  default     = [80, 443]
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default = {
    Project     = "MyProject"
    ManagedBy   = "Terraform"
  }
}
```

### Variable Values
```hcl
# terraform.tfvars
aws_access_key = "AKIAIOSFODNN7EXAMPLE"
aws_secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
instance_type  = "t3.micro"
environment    = "dev"
ports          = [80, 443, 8080]
tags = {
  Environment = "Development"
  Team        = "DevOps"
}
```

### Output Definitions
```hcl
# outputs.tf
output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web_server.public_ip
}

output "instance_private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.web_server.private_ip
}

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web_server.id
}

output "security_group_id" {
  description = "ID of the security group"
  value       = aws_security_group.web_sg.id
}
```

## Terraform Resources

### AWS Resources
```hcl
# EC2 Instance
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-web-server"
    }
  )
  
  user_data = <<-EOF
              #!/bin/bash
              apt-get update
              apt-get install -y nginx
              systemctl start nginx
              systemctl enable nginx
              EOF
}

# S3 Bucket
resource "aws_s3_bucket" "my_bucket" {
  bucket = "${var.environment}-my-bucket-${random_id.bucket_id.hex}"
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-my-bucket"
    }
  )
}

resource "aws_s3_bucket_acl" "my_bucket_acl" {
  bucket = aws_s3_bucket.my_bucket.id
  acl    = "private"
}

# RDS Database
resource "aws_db_instance" "my_database" {
  identifier           = "${var.environment}-my-database"
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  allocated_storage    = 20
  storage_type         = "gp2"
  username             = var.db_username
  password             = var.db_password
  db_subnet_group_name = aws_db_subnet_group.my_db_subnet_group.name
  vpc_security_group_ids = [aws_security_group.db_sg.id]
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-my-database"
    }
  )
}

# Lambda Function
resource "aws_lambda_function" "my_lambda" {
  filename      = "lambda_function.zip"
  function_name = "${var.environment}-my-lambda"
  role          = aws_iam_role.lambda_role.arn
  handler       = "lambda_function.lambda_handler"
  runtime       = "python3.8"
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-my-lambda"
    }
  )
}
```

### Azure Resources
```hcl
# Resource Group
resource "azurerm_resource_group" "my_rg" {
  name     = "${var.environment}-my-rg"
  location = var.location
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-my-rg"
    }
  )
}

# Virtual Network
resource "azurerm_virtual_network" "my_vnet" {
  name                = "${var.environment}-my-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.my_rg.location
  resource_group_name = azurerm_resource_group.my_rg.name
}

# Subnet
resource "azurerm_subnet" "my_subnet" {
  name                 = "${var.environment}-my-subnet"
  resource_group_name  = azurerm_resource_group.my_rg.name
  virtual_network_name = azurerm_virtual_network.my_vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Virtual Machine
resource "azurerm_virtual_machine" "my_vm" {
  name                  = "${var.environment}-my-vm"
  location              = azurerm_resource_group.my_rg.location
  resource_group_name   = azurerm_resource_group.my_rg.name
  network_interface_ids = [azurerm_network_interface.my_nic.id]
  vm_size               = "Standard_DS1_v2"
  
  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
  
  storage_os_disk {
    name              = "${var.environment}-my-os-disk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }
  
  os_profile {
    computer_name  = "${var.environment}-my-vm"
    admin_username = var.admin_username
    admin_password = var.admin_password
  }
  
  os_profile_linux_config {
    disable_password_authentication = false
  }
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-my-vm"
    }
  )
}
```

### Google Cloud Resources
```hcl
# Compute Instance
resource "google_compute_instance" "my_instance" {
  name         = "${var.environment}-my-instance"
  machine_type = "e2-micro"
  zone         = "${var.gcp_region}-a"
  
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }
  
  network_interface {
    network = "default"
    access_config {
      // Ephemeral IP
    }
  }
  
  metadata = {
    foo = "bar"
  }
  
  metadata_startup_script = "echo hi > /test.txt"
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-my-instance"
    }
  )
}

# Cloud Storage Bucket
resource "google_storage_bucket" "my_bucket" {
  name     = "${var.environment}-my-bucket-${random_id.bucket_id.hex}"
  location = var.gcp_region
  
  uniform_bucket_level_access = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-my-bucket"
    }
  )
}

# Cloud SQL Database
resource "google_sql_database_instance" "my_database" {
  name             = "${var.environment}-my-database"
  database_version = "MYSQL_8_0"
  region           = var.gcp_region
  
  settings {
    tier = "db-f1-micro"
  }
  
  deletion_protection = false
}
```

## Terraform Modules

### Module Structure
```
modules/
├── ec2_instance/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
└── vpc/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

### Module Example
```hcl
# modules/ec2_instance/main.tf
resource "aws_instance" "web_server" {
  ami           = var.ami
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  vpc_security_group_ids = [var.security_group_id]
  
  tags = merge(
    var.tags,
    {
      Name = var.name
    }
  )
  
  user_data = var.user_data
}

# modules/ec2_instance/variables.tf
variable "ami" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "Instance type for the EC2 instance"
  type        = string
  default     = "t2.micro"
}

variable "subnet_id" {
  description = "Subnet ID where the instance will be launched"
  type        = string
}

variable "security_group_id" {
  description = "Security group ID for the instance"
  type        = string
}

variable "name" {
  description = "Name tag for the instance"
  type        = string
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}

variable "user_data" {
  description = "User data script to run on instance launch"
  type        = string
  default     = ""
}

# modules/ec2_instance/outputs.tf
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web_server.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web_server.public_ip
}

output "instance_private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.web_server.private_ip
}
```

### Using Modules
```hcl
# main.tf
module "web_server" {
  source = "./modules/ec2_instance"
  
  ami                = "ami-0c55b159cbfafe1f0"
  instance_type      = var.instance_type
  subnet_id          = aws_subnet.public.id
  security_group_id  = aws_security_group.web_sg.id
  name               = "${var.environment}-web-server"
  tags               = var.tags
  user_data          = <<-EOF
                      #!/bin/bash
                      apt-get update
                      apt-get install -y nginx
                      systemctl start nginx
                      systemctl enable nginx
                      EOF
}

# Access module outputs
output "web_server_public_ip" {
  value = module.web_server.instance_public_ip
}
```

## Terraform State Management

### Local State
```hcl
# terraform.tfstate (automatically created)
{
  "version": 4,
  "terraform_version": "1.1.9",
  "serial": 1,
  "lineage": "12345678-1234-1234-1234-123456789012",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "web_server",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t2.micro",
            "public_ip": "1.2.3.4"
          }
        }
      ]
    }
  ]
}
```

### Remote State
```hcl
# Configure remote state
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}

# Or use Azure Storage
terraform {
  backend "azurerm" {
    resource_group_name  = "my-rg"
    storage_account_name = "mystorageaccount"
    container_name       = "terraform-state"
    key                  = "prod/terraform.tfstate"
  }
}

# Or use Google Cloud Storage
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "prod"
  }
}
```

## Terraform Commands

### Basic Commands
```bash
# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Format configuration files
terraform fmt

# Plan changes
terraform plan

# Apply changes
terraform apply

# Apply changes with auto-approval
terraform apply -auto-approve

# Destroy infrastructure
terraform destroy

# Destroy with auto-approval
terraform destroy -auto-approve

# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web_server

# Import existing resource
terraform import aws_instance.web_server i-1234567890abcdef0

# Remove resource from state
terraform state rm aws_instance.web_server

# Move resource in state
terraform state mv aws_instance.web_server aws_instance.web_server_new
```

### Advanced Commands
```bash
# Plan with specific variables
terraform plan -var="instance_type=t3.micro"

# Plan with variable file
terraform plan -var-file="production.tfvars"

# Create execution plan
terraform plan -out=tfplan

# Apply execution plan
terraform apply tfplan

# Refresh state
terraform refresh

# Force unlock state
terraform force-unlock LOCK_ID

# Taint resource
terraform taint aws_instance.web_server

# Untaint resource
terraform untaint aws_instance.web_server

# Workspace management
terraform workspace new dev
terraform workspace select dev
terraform workspace list
terraform workspace show
terraform workspace delete dev

# Get provider documentation
terraform providers

# Get module documentation
terraform providers mirror ./providers
```

## Terraform Functions

### String Functions
```hcl
# Join strings
output "joined_string" {
  value = join("-", ["hello", "world"])
}

# Split string
output "split_string" {
  value = split("-", "hello-world")
}

# Replace string
output "replaced_string" {
  value = replace("hello-world", "-", " ")
}

# Upper case
output "upper_case" {
  value = upper("hello world")
}

# Lower case
output "lower_case" {
  value = lower("HELLO WORLD")
}

# Trim string
output "trimmed_string" {
  value = trim("  hello world  ")
}

# String length
output "string_length" {
  value = length("hello world")
}
```

### Numeric Functions
```hcl
# Absolute value
output "absolute_value" {
  value = abs(-10)
}

# Maximum value
output "max_value" {
  value = max(10, 20, 30)
}

# Minimum value
output "min_value" {
  value = min(10, 20, 30)
}

# Power
output "power" {
  value = pow(2, 3)
}

# Sign
output "sign" {
  value = sign(-10)
}

# Ceiling
output "ceiling" {
  value = ceil(3.14)
}

# Floor
output "floor" {
  value = floor(3.14)
}

# Parse integer
output "parsed_int" {
  value = tonumber("123")
}
```

### Collection Functions
```hcl
# Length of collection
output "list_length" {
  value = length(["a", "b", "c"])
}

# Element from list
output "first_element" {
  value = element(["a", "b", "c"], 0)
}

# Check if element exists
output "contains_element" {
  value = contains(["a", "b", "c"], "b")
}

# Concatenate lists
output "concatenated_list" {
  value = concat(["a", "b"], ["c", "d"])
}

# Slice list
output "sliced_list" {
  value = slice(["a", "b", "c", "d"], 1, 3)
}

# Merge maps
output "merged_map" {
  value = merge({a = 1}, {b = 2})
}

# Keys from map
output "map_keys" {
  value = keys({a = 1, b = 2})
}

# Values from map
output "map_values" {
  value = values({a = 1, b = 2})
}
```

### Date and Time Functions
```hcl
# Current timestamp
output "current_timestamp" {
  value = timestamp()
}

# Format timestamp
output "formatted_timestamp" {
  value = formatdate("YYYY-MM-DD", timestamp())
}

# Time add
output "time_add" {
  value = timeadd(timestamp(), "1h")
}

# Timecmp
output "time_compare" {
  value = timecmp(timestamp(), timeadd(timestamp(), "1h"))
}
```

## Terraform Workspaces

### Workspace Management
```bash
# Create new workspace
terraform workspace new dev

# Switch to workspace
terraform workspace select dev

# List workspaces
terraform workspace list

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

### Workspace Configuration
```hcl
# Use workspace-specific variables
variable "environment" {
  description = "Environment name"
  type        = string
  default     = terraform.workspace
}

# Use workspace-specific resource names
resource "aws_instance" "web_server" {
  ami           = var.ami
  instance_type = var.instance_type
  
  tags = merge(
    var.tags,
    {
      Name = "${terraform.workspace}-web-server"
    }
  )
}

# Use workspace-specific backend configuration
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "${terraform.workspace}/terraform.tfstate"
    region = "us-west-2"
  }
}
```

## Terraform Testing

### Using Terratest
```go
// main_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformAwsExample(t *testing.T) {
    t.Parallel()
    
    terraformOptions := &terraform.Options{
        TerraformDir: "../",
    }
    
    defer terraform.Destroy(t, terraformOptions)
    
    terraform.InitAndApply(t, terraformOptions)
    
    instanceId := terraform.Output(t, terraformOptions, "instance_id")
    assert.NotEmpty(t, instanceId)
}
```

## Interview Questions

### Beginner Level
1. **What is Terraform?**
   - Terraform is an infrastructure as code (IaC) tool that allows you to define and provision infrastructure using a declarative configuration language

2. **What are the main benefits of Terraform?**
   - Infrastructure as code, execution plans, resource graphs, change automation, and multi-cloud support

3. **What is HCL?**
   - HCL (HashiCorp Configuration Language) is the declarative configuration language used by Terraform

4. **What is a Terraform provider?**
   - A provider is a plugin that enables Terraform to interact with APIs of services like AWS, Azure, GCP, etc.

### Intermediate Level
1. **What is the difference between Terraform and CloudFormation?**
   - Terraform is cloud-agnostic and supports multiple providers, while CloudFormation is AWS-specific

2. **What is Terraform state?**
   - Terraform state is a JSON file that maps resources to your configuration and tracks the metadata of your infrastructure

3. **What are Terraform modules?**
   - Modules are reusable collections of resources that can be used to organize and encapsulate infrastructure

4. **What is the difference between resource and data source in Terraform?**
   - Resources create and manage infrastructure components, while data sources read existing infrastructure

### Advanced Level
1. **How does Terraform handle dependencies?**
   - Terraform automatically determines dependencies between resources and creates them in the correct order

2. **What is the purpose of Terraform workspaces?**
   - Workspaces allow you to manage multiple environments with the same configuration, each with its own state

3. **How do you handle secrets in Terraform?**
   - Use environment variables, encrypted storage, or services like AWS Secrets Manager or HashiCorp Vault

4. **What is the difference between local and remote state in Terraform?**
   - Local state is stored on your filesystem, while remote state is stored in a shared storage system like S3 or Azure Storage

## Best Practices

### Configuration Structure
- **Modular Design**: Use modules to organize and reuse code
- **Variable Management**: Use variables for configuration values
- **Output Management**: Use outputs to expose resource attributes
- **Documentation**: Document your modules and configurations
- **Version Control**: Store all Terraform code in version control

### State Management
- **Remote State**: Use remote state for team collaboration
- **State Locking**: Enable state locking to prevent conflicts
- **State Backup**: Regularly backup your state files
- **State Review**: Review state changes before applying
- **State Cleanup**: Remove unused resources from state

### Security
- **Secret Management**: Don't store secrets in configuration files
- **Access Control**: Use proper access controls for state files
- **Encryption**: Encrypt sensitive data at rest and in transit
- **Audit Logging**: Enable audit logging for all operations
- **Regular Reviews**: Regularly review and update security configurations

## Troubleshooting

### Common Issues
```bash
# Check Terraform version
terraform --version

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Check provider versions
terraform providers

# Show current state
terraform show

# Check for resource drift
terraform plan

# Debug with verbose output
terraform apply -verbose

# Check state file
terraform state list

# Import existing resource
terraform import aws_instance.web_server i-1234567890abcdef0

# Remove resource from state
terraform state rm aws_instance.web_server
```

### Debugging Tips
- **Use -verbose flag**: Get detailed output during operations
- **Check provider documentation**: Refer to official provider documentation
- **Use Terraform console**: Test expressions and functions
- **Review execution plan**: Always review the plan before applying
- **Check state file**: Ensure state is consistent with actual infrastructure

## Resources

### Official Documentation
- [Terraform Documentation](https://www.terraform.io/docs)
- [Terraform Providers](https://registry.terraform.io/providers)
- [Terraform Modules](https://registry.terraform.io/modules)

### Learning Resources
- [Terraform Tutorials](https://learn.hashicorp.com/terraform)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/best-practices/index.html)
- [Terraform Certification](https://www.hashicorp.com/certification/terraform-associate)

### Community
- [Terraform Forums](https://discuss.hashicorp.com/c/terraform-core)
- [Terraform GitHub](https://github.com/hashicorp/terraform)
- [Terraform Stack Overflow](https://stackoverflow.com/questions/tagged/terraform)