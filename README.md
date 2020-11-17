# terraform-cloudmap
AWS Cloud Map
Service discovery for cloud resources
Define user-friendly names for your cloud resources so that your applications can quickly and dynamically discover them.

Script to create AWS Cloud Map with CNAME Record

Put the following under variable.tf file in the same place where you will run you terraform. 

variable "cluster_name" {
default = "your k8s cluster"
}
variable "env" {
default = "environment"
}
variable "team" {
default = "team who handles this"
}
variable "region" {
default = "region your k8s is located"
}

-----------------------------------------------------------------------------------------------
Now create another file which will actually create Cloud Map while using the above variable. 


terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 2.70"
    }
  }
}

provider "aws" {
  profile = "default"
  region  = "us-east-2"
}

resource "aws_vpc" "my_vpc" {
  cidr_block           = "10.24.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
}

resource "aws_service_discovery_private_dns_namespace" "my_namespace" {
  name        = "${var.cluster_name}.${var.env}.${var.team}.${var.region}.website.com"
  description = "This is a Cloud MAP for the above name"
  vpc         = aws_vpc.my_vpc.id
}

resource "aws_service_discovery_service" "my_service" {
  name = "Service Name"

  dns_config {
    namespace_id = aws_service_discovery_private_dns_namespace.my_namespace.id

    dns_records {
      ttl  = 100
      type = "CNAME"
    }

    routing_policy = "MULTIVALUE" # Change this as per the DNS type 
  }
