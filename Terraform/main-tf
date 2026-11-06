provider "aws" {
  region = "us-east-1"
}

# Create the VPC
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.19.0"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  enable_dns_hostnames = true
  enable_dns_support   = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = "1"
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = "1"
  }
}

# Create the EKS cluster and worker nodes
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "sukumar"
  cluster_version = "1.31"

  cluster_endpoint_public_access = true

  # Use the IAM roles you already created
  create_iam_role = false
  iam_role_arn    = "arn:aws:iam::183533420201:role/eks-cluster-role"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    eks-node-group = {
      instance_types = ["c7i-flex.large"]

      min_size     = 1
      max_size     = 4
      desired_size = 2

      create_iam_role = false
      iam_role_arn    = "arn:aws:iam::183533420201:role/eks-worker-role"
    }
  }

  tags = {
    Environment = "dev"
    Terraform   = "true"
  }
}
