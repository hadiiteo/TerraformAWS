# TerraformAWS
Setup Terraform To Connect To AWS

Each Terraform configuration must be in its own working directory. Create a directory for your configuration.
```
$ mkdir learn-terraform-aws-instance
```
Change into the directory.
```
$ cd learn-terraform-aws-instance
```
Create a file to define your infrastructure.
```
$ touch main.tf
```
Open main.tf in your text editor, paste in the configuration below, and save the file.
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region  = "us-west-2"
  profile = "XXX"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

I am using a custom profile so i am defining profile = "XXX"

i encountered the following error message
```
adminjanesmith@DESKTOP-IMMDGJC:~/learn-terraform-aws-instance$ aws ec2 describe-images --region ap-southeast-1 --image-ids ami-830c94e3

An error occurred (InvalidAMIID.NotFound) when calling the DescribeImages operation: The image id '[ami-830c94e3]' does not exist
adminjanesmith@DESKTOP-IMMDGJC:~/learn-terraform-aws-instance$
```

The error confirms that ami-830c94e3 does not exist in the ap-southeast-1 (Singapore) region.

Run this AWS CLI command to find the latest Ubuntu AMI in ap-southeast-1:

```bash
aws ec2 describe-images \
  --region ap-southeast-1 \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
  --output text
This will return a valid AMI like:
→ ami-0a9d27a9f4f5c0efc (example, check for latest)

i modified back the main.tf and corrected the ami value


```
Initialize the directory.
```
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 4.16"...
- Installing hashicorp/aws v4.17.0...
- Installed hashicorp/aws v4.17.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Format your configuration. Terraform will print out the names of the files it modified, if any. In this case, your configuration file was already formatted correctly, so Terraform won't return any file names.
```
$ terraform fmt
```
You can also make sure your configuration is syntactically valid and internally consistent by using the terraform validate command.

Validate your configuration. The example configuration provided above is valid, so Terraform will return a success message.
```
$ terraform validate
Success! The configuration is valid.
```


Apply the configuration now with the terraform apply command. Terraform will print output similar to what is shown below. We have truncated some of the output to save space.
```
$ terraform apply

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.app_server will be created
  + resource "aws_instance" "app_server" {
      + ami                          = "ami-830c94e3"
      + arn                          = (known after apply)
##...

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:

  Enter a value: yes

aws_instance.app_server: Creating...
aws_instance.app_server: Still creating... [10s elapsed]
aws_instance.app_server: Still creating... [20s elapsed]
aws_instance.app_server: Still creating... [30s elapsed]
aws_instance.app_server: Creation complete after 36s [id=i-01e03375ba238b384]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

```


