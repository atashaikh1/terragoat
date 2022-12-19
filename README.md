# TerraGoat - Vulnerable Terraform Infrastructure

[![Maintained by Bridgecrew.io](https://img.shields.io/badge/maintained%20by-bridgecrew.io-blueviolet)](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat)
[![Infrastructure Tests](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/general)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=INFRASTRUCTURE+SECURITY)
[![CIS Azure](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/cis_azure)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=CIS+AZURE+V1.1)
[![CIS GCP](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/cis_gcp)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=CIS+GCP+V1.1)
[![CIS AWS](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/cis_aws)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=CIS+AWS+V1.2)
[![PCI](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/pci)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=PCI-DSS+V3.2)
![Terraform Version](https://img.shields.io/badge/tf-%3E%3D0.12.0-blue.svg) 
[![slack-community](https://slack.bridgecrew.io/badge.svg)](https://slack.bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat)


TerraGoat is Bridgecrew's "Vulnerable by Design" Terraform repository.
![Terragoat](terragoat-logo.png)

TerraGoat is Bridgecrew's "Vulnerable by Design" Terraform repository.
TerraGoat is a learning and training project that demonstrates how common configuration errors can find their way into production cloud environments.

## Table of Contents

* [Introduction](#introduction)
* [Getting Started](#getting-started)
  * [AWS](#aws-setup)
  * [Azure](#azure-setup)
  * [GCP](#gcp-setup)
* [Contributing](#contributing)
* [Support](#support)

## Introduction

TerraGoat was built to enable DevSecOps design and implement a sustainable misconfiguration prevention strategy. It can be used to test a policy-as-code framework like [Bridgecrew](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat) & [Checkov](https://github.com/bridgecrewio/checkov/), inline-linters, pre-commit hooks or other code scanning methods.

TerraGoat follows the tradition of existing *Goat projects that provide a baseline training ground to practice implementing secure development best practices for cloud infrastructure.

## Important notes

* **Where to get help:** the [Bridgecrew Community Slack](https://slack.bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat)

Before you proceed please take a not of these warning:
> :warning: TerraGoat creates intentionally vulnerable AWS resources into your account. **DO NOT deploy TerraGoat in a production environment or alongside any sensitive AWS resources.**

## Requirements

* Terraform 0.12
* aws cli
* azure cli

To prevent vulnerable infrastructure from arriving to production see: [Bridgecrew](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat) & [checkov](https://github.com/bridgecrewio/checkov/), the open source static analysis tool for infrastructure as code.

## Getting started

### AWS Setup

#### Installation (AWS)

You can deploy multiple TerraGoat stacks in a single AWS account using the parameter `TF_VAR_environment`.

#### Create an S3 Bucket backend to keep Terraform state

```bash
export TERRAGOAT_STATE_BUCKET="mydevsecops-bucket"
export TF_VAR_company_name=acme
export TF_VAR_environment=mydevsecops
export TF_VAR_region="us-west-2"

aws s3api create-bucket --bucket $TERRAGOAT_STATE_BUCKET \
    --region $TF_VAR_region --create-bucket-configuration LocationConstraint=$TF_VAR_region

# Enable versioning
aws s3api put-bucket-versioning --bucket $TERRAGOAT_STATE_BUCKET --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption --bucket $TERRAGOAT_STATE_BUCKET --server-side-encryption-configuration '{
  "Rules": [
    {
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms"
      }
    }
  ]
}'
```

#### Apply TerraGoat (AWS)

```bash
cd terraform/aws/
terraform init \
-backend-config="bucket=$TERRAGOAT_STATE_BUCKET" \
-backend-config="key=$TF_VAR_company_name-$TF_VAR_environment.tfstate" \
-backend-config="region=$TF_VAR_region"

terraform apply
```

#### Remove TerraGoat (AWS)

```bash
terraform destroy
```

#### Creating multiple TerraGoat AWS stacks

```bash
cd terraform/aws/
export TERRAGOAT_ENV=$TF_VAR_environment
export TERRAGOAT_STACKS_NUM=5
for i in $(seq 1 $TERRAGOAT_STACKS_NUM)
do
    export TF_VAR_environment=$TERRAGOAT_ENV$i
    terraform init \
    -backend-config="bucket=$TERRAGOAT_STATE_BUCKET" \
    -backend-config="key=$TF_VAR_company_name-$TF_VAR_environment.tfstate" \
    -backend-config="region=$TF_VAR_region"

    terraform apply -auto-approve
done
```

#### Deleting multiple TerraGoat stacks (AWS)

```bash
cd terraform/aws/
export TF_VAR_environment = $TERRAGOAT_ENV
for i in $(seq 1 $TERRAGOAT_STACKS_NUM)
do
    export TF_VAR_environment=$TERRAGOAT_ENV$i
    terraform init \
    -backend-config="bucket=$TERRAGOAT_STATE_BUCKET" \
    -backend-config="key=$TF_VAR_company_name-$TF_VAR_environment.tfstate" \
    -backend-config="region=$TF_VAR_region"

    terraform destroy -auto-approve
done
```

### Azure Setup

#### Installation (Azure)

You can deploy multiple TerraGoat stacks in a single Azure subscription using the parameter `TF_VAR_environment`.

#### Create an Azure Storage Account backend to keep Terraform state

```bash
export TERRAGOAT_RESOURCE_GROUP="TerraGoatRG"
export TERRAGOAT_STATE_STORAGE_ACCOUNT="mydevsecopssa"
export TERRAGOAT_STATE_CONTAINER="mydevsecops"
export TF_VAR_environment="dev"
export TF_VAR_region="westus"

# Create resource group
az group create --location $TF_VAR_region --name $TERRAGOAT_RESOURCE_GROUP

# Create storage account
az storage account create --name $TERRAGOAT_STATE_STORAGE_ACCOUNT --resource-group $TERRAGOAT_RESOURCE_GROUP --location $TF_VAR_region --sku Standard_LRS --kind StorageV2 --https-only true --encryption-services blob

# Get storage account key
ACCOUNT_KEY=$(az storage account keys list --resource-group $TERRAGOAT_RESOURCE_GROUP --account-name $TERRAGOAT_STATE_STORAGE_ACCOUNT --query [0].value -o tsv)

# Create blob container
az storage container create --name $TERRAGOAT_STATE_CONTAINER --account-name $TERRAGOAT_STATE_STORAGE_ACCOUNT --account-key $ACCOUNT_KEY
```

#### Apply TerraGoat (Azure)

```bash
cd terraform/azure/
terraform init -reconfigure -backend-config="resource_group_name=$TERRAGOAT_RESOURCE_GROUP" \
    -backend-config "storage_account_name=$TERRAGOAT_STATE_STORAGE_ACCOUNT" \
    -backend-config="container_name=$TERRAGOAT_STATE_CONTAINER" \
    -backend-config "key=$TF_VAR_environment.terraform.tfstate"

terraform apply
```

#### Remove TerraGoat (Azure)

```bash
terraform destroy
```

### GCP Setup

#### Installation (GCP)

You can deploy multiple TerraGoat stacks in a single GCP project using the parameter `TF_VAR_environment`.

#### Create a GCS backend to keep Terraform state

To use terraform, a Service Account and matching set of credentials are required.
If they do not exist, they must be manually created for the relevant project.
To create the Service Account:
1. Sign into your GCP project, go to `IAM` > `Service Accounts`.
2. Click the `CREATE SERVICE ACCOUNT`.
3. Give a name to your service account (for example - `terragoat`) and click `CREATE`.
4. Grant the Service Account the `Project` > `Editor` role and click `CONTINUE`.
5. Click `DONE`.

To create the credentials:
1. Sign into your GCP project, go to `IAM` > `Service Accounts` and click on the relevant Service Account.
2. Click `ADD KEY` > `Create new key` > `JSON` and click `CREATE`. This will create a `.json` file and download it to your computer.

We recommend saving the key with a nicer name than the auto-generated one (i.e. `terragoat_credentials.json`), and storing the resulting JSON file inside `terraform/gcp` directory of terragoat.
Once the credentials are set up, create the BE configuration as follows:

```bash
export TF_VAR_environment="dev"
export TF_TERRAGOAT_STATE_BUCKET=remote-state-bucket-terragoat
export TF_VAR_credentials_path=<PATH_TO_CREDNETIALS_FILE> # example: export TF_VAR_credentials_path=terragoat_credentials.json
export TF_VAR_project=<YOUR_PROJECT_NAME_HERE>

# Create storage bucket
gsutil mb gs://${TF_TERRAGOAT_STATE_BUCKET}
```

#### Apply TerraGoat (GCP)

```bash
cd terraform/gcp/
terraform init -reconfigure -backend-config="bucket=$TF_TERRAGOAT_STATE_BUCKET" \
    -backend-config "credentials=$TF_VAR_credentials_path" \
    -backend-config "prefix=terragoat/${TF_VAR_environment}"

terraform apply
```

#### Remove TerraGoat (GCP)

```bash
terraform destroy
```

## Bridgecrew's IaC herd of goats

* [CfnGoat](https://github.com/bridgecrewio/cfngoat) - Vulnerable by design Cloudformation template
* [TerraGoat](https://github.com/bridgecrewio/terragoat) - Vulnerable by design Terraform stack
* [CDKGoat](https://github.com/bridgecrewio/cdkgoat) - Vulnerable by design CDK application

## Contributing

Contribution is welcomed!

We would love to hear about more ideas on how to find vulnerable infrastructure-as-code design patterns.

## Support

[Bridgecrew](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat) builds and maintains TerraGoat to encourage the adoption of policy-as-code.

If you need direct support you can contact us at [info@bridgecrew.io](mailto:info@bridgecrew.io).

## Existing vulnerabilities (Auto-Generated)
### terraform scan results:

|     | check_id      | file                          | resource                                            | check_name                                                                                                   | guideline                                                                                                                                    |
|-----|---------------|-------------------------------|-----------------------------------------------------|--------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
|   0 | CKV_AWS_226   | /aws/db-app.tf                | aws_db_instance.default                             | Ensure DB instance gets all minor upgrades automatically                                                     | https://docs.bridgecrew.io/docs/ensure-aws-db-instance-gets-all-minor-upgrades-automatically                                                 |
|   1 | CKV_AWS_129   | /aws/db-app.tf                | aws_db_instance.default                             | Ensure that respective logs of Amazon Relational Database Service (Amazon RDS) are enabled                   | https://docs.bridgecrew.io/docs/ensure-that-respective-logs-of-amazon-relational-database-service-amazon-rds-are-enabled                     |
|   2 | CKV_AWS_118   | /aws/db-app.tf                | aws_db_instance.default                             | Ensure that enhanced monitoring is enabled for Amazon RDS instances                                          | https://docs.bridgecrew.io/docs/ensure-that-enhanced-monitoring-is-enabled-for-amazon-rds-instances                                          |
|   3 | CKV_AWS_157   | /aws/db-app.tf                | aws_db_instance.default                             | Ensure that RDS instances have Multi-AZ enabled                                                              | https://docs.bridgecrew.io/docs/general_73                                                                                                   |
|   4 | CKV_AWS_133   | /aws/db-app.tf                | aws_db_instance.default                             | Ensure that RDS instances has backup policy                                                                  | https://docs.bridgecrew.io/docs/ensure-that-rds-instances-have-backup-policy                                                                 |
|   5 | CKV_AWS_161   | /aws/db-app.tf                | aws_db_instance.default                             | Ensure RDS database has IAM authentication enabled                                                           | https://docs.bridgecrew.io/docs/ensure-rds-database-has-iam-authentication-enabled                                                           |
|   6 | CKV_AWS_23    | /aws/db-app.tf                | aws_security_group.default                          | Ensure every security groups rule has a description                                                          | https://docs.bridgecrew.io/docs/networking_31                                                                                                |
|   7 | CKV_AWS_23    | /aws/db-app.tf                | aws_security_group_rule.ingress                     | Ensure every security groups rule has a description                                                          | https://docs.bridgecrew.io/docs/networking_31                                                                                                |
|   8 | CKV_AWS_23    | /aws/db-app.tf                | aws_security_group_rule.egress                      | Ensure every security groups rule has a description                                                          | https://docs.bridgecrew.io/docs/networking_31                                                                                                |
|   9 | CKV_AWS_79    | /aws/db-app.tf                | aws_instance.db_app                                 | Ensure Instance Metadata Service Version 1 is not enabled                                                    | https://docs.bridgecrew.io/docs/bc_aws_general_31                                                                                            |
|  10 | CKV_AWS_8     | /aws/db-app.tf                | aws_instance.db_app                                 | Ensure all data stored in the Launch configuration or instance Elastic Blocks Store is securely encrypted    | https://docs.bridgecrew.io/docs/general_13                                                                                                   |
|  11 | CKV_AWS_135   | /aws/db-app.tf                | aws_instance.db_app                                 | Ensure that EC2 is EBS optimized                                                                             | https://docs.bridgecrew.io/docs/ensure-that-ec2-is-ebs-optimized                                                                             |
|  12 | CKV_AWS_126   | /aws/db-app.tf                | aws_instance.db_app                                 | Ensure that detailed monitoring is enabled for EC2 instances                                                 | https://docs.bridgecrew.io/docs/ensure-that-detailed-monitoring-is-enabled-for-ec2-instances                                                 |
|  13 | CKV_AWS_79    | /aws/ec2.tf                   | aws_instance.web_host                               | Ensure Instance Metadata Service Version 1 is not enabled                                                    | https://docs.bridgecrew.io/docs/bc_aws_general_31                                                                                            |
|  14 | CKV_AWS_46    | /aws/ec2.tf                   | aws_instance.web_host                               | Ensure no hard-coded secrets exist in EC2 user data                                                          | https://docs.bridgecrew.io/docs/bc_aws_secrets_1                                                                                             |
|  15 | CKV_AWS_8     | /aws/ec2.tf                   | aws_instance.web_host                               | Ensure all data stored in the Launch configuration or instance Elastic Blocks Store is securely encrypted    | https://docs.bridgecrew.io/docs/general_13                                                                                                   |
|  16 | CKV_AWS_135   | /aws/ec2.tf                   | aws_instance.web_host                               | Ensure that EC2 is EBS optimized                                                                             | https://docs.bridgecrew.io/docs/ensure-that-ec2-is-ebs-optimized                                                                             |
|  17 | CKV_AWS_126   | /aws/ec2.tf                   | aws_instance.web_host                               | Ensure that detailed monitoring is enabled for EC2 instances                                                 | https://docs.bridgecrew.io/docs/ensure-that-detailed-monitoring-is-enabled-for-ec2-instances                                                 |
|  18 | CKV_AWS_189   | /aws/ec2.tf                   | aws_ebs_volume.web_host_storage                     | Ensure EBS Volume is encrypted by KMS using a customer managed Key (CMK)                                     | https://docs.bridgecrew.io/docs/bc_aws_general_109                                                                                           |
|  19 | CKV_AWS_260   | /aws/ec2.tf                   | aws_security_group.web-node                         | Ensure no security groups allow ingress from 0.0.0.0:0 to port 80                                            | https://docs.bridgecrew.io/docs/ensure-aws-security-groups-do-not-allow-ingress-from-00000-to-port-80                                        |
|  20 | CKV_AWS_23    | /aws/ec2.tf                   | aws_security_group.web-node                         | Ensure every security groups rule has a description                                                          | https://docs.bridgecrew.io/docs/networking_31                                                                                                |
|  21 | CKV_AWS_24    | /aws/ec2.tf                   | aws_security_group.web-node                         | Ensure no security groups allow ingress from 0.0.0.0:0 to port 22                                            | https://docs.bridgecrew.io/docs/networking_1-port-security                                                                                   |
|  22 | CKV_AWS_51    | /aws/ecr.tf                   | aws_ecr_repository.repository                       | Ensure ECR Image Tags are immutable                                                                          | https://docs.bridgecrew.io/docs/bc_aws_general_24                                                                                            |
|  23 | CKV_AWS_136   | /aws/ecr.tf                   | aws_ecr_repository.repository                       | Ensure that ECR repositories are encrypted using KMS                                                         | https://docs.bridgecrew.io/docs/ensure-that-ecr-repositories-are-encrypted                                                                   |
|  24 | CKV_AWS_37    | /aws/eks.tf                   | aws_eks_cluster.eks_cluster                         | Ensure Amazon EKS control plane logging enabled for all log types                                            | https://docs.bridgecrew.io/docs/bc_aws_kubernetes_4                                                                                          |
|  25 | CKV_AWS_58    | /aws/eks.tf                   | aws_eks_cluster.eks_cluster                         | Ensure EKS Cluster has Secrets Encryption Enabled                                                            | https://docs.bridgecrew.io/docs/bc_aws_kubernetes_3                                                                                          |
|  26 | CKV_AWS_92    | /aws/elb.tf                   | aws_elb.weblb                                       | Ensure the ELB has access logging enabled                                                                    | https://docs.bridgecrew.io/docs/bc_aws_logging_23                                                                                            |
|  27 | CKV_AWS_127   | /aws/elb.tf                   | aws_elb.weblb                                       | Ensure that Elastic Load Balancer(s) uses SSL certificates provided by AWS Certificate Manager               | https://docs.bridgecrew.io/docs/ensure-that-elastic-load-balancers-uses-ssl-certificates-provided-by-aws-certificate-manager                 |
|  28 | CKV_AWS_283   | /aws/es.tf                    | aws_iam_policy_document.policy                      | Ensure no IAM policies documents allow ALL or any AWS principal permissions to the resource                  |                                                                                                                                              |
|  29 | CKV_AWS_111   | /aws/es.tf                    | aws_iam_policy_document.policy                      | Ensure IAM policies does not allow write access without constraints                                          | https://docs.bridgecrew.io/docs/ensure-iam-policies-do-not-allow-write-access-without-constraint                                             |
|  30 | CKV_AWS_109   | /aws/es.tf                    | aws_iam_policy_document.policy                      | Ensure IAM policies does not allow permissions management / resource exposure without constraints            | https://docs.bridgecrew.io/docs/ensure-iam-policies-do-not-allow-permissions-management-resource-exposure-without-constraint                 |
|  31 | CKV_AWS_228   | /aws/es.tf                    | aws_elasticsearch_domain.monitoring-framework       | Verify Elasticsearch domain is using an up to date TLS policy                                                | https://docs.bridgecrew.io/docs/ensure-aws-elasticsearch-domain-uses-an-updated-tls-policy                                                   |
|  32 | CKV_AWS_137   | /aws/es.tf                    | aws_elasticsearch_domain.monitoring-framework       | Ensure that Elasticsearch is configured inside a VPC                                                         | https://docs.bridgecrew.io/docs/ensure-that-elasticsearch-is-configured-inside-a-vpc                                                         |
|  33 | CKV_AWS_84    | /aws/es.tf                    | aws_elasticsearch_domain.monitoring-framework       | Ensure Elasticsearch Domain Logging is enabled                                                               | https://docs.bridgecrew.io/docs/elasticsearch_7                                                                                              |
|  34 | CKV_AWS_248   | /aws/es.tf                    | aws_elasticsearch_domain.monitoring-framework       | Ensure that Elasticsearch is not using the default Security Group                                            | https://docs.bridgecrew.io/docs/ensure-aws-elasticsearch-does-not-use-the-default-security-group                                             |
|  35 | CKV_AWS_247   | /aws/es.tf                    | aws_elasticsearch_domain.monitoring-framework       | Ensure all data stored in the Elasticsearch is encrypted with a CMK                                          | https://docs.bridgecrew.io/docs/ensure-aws-all-data-stored-in-the-elasticsearch-domain-is-encrypted-using-a-customer-managed-key-cmk         |
|  36 | CKV_AWS_273   | /aws/iam.tf                   | aws_iam_user.user                                   | Ensure access is controlled through SSO and not AWS IAM defined users                                        |                                                                                                                                              |
|  37 | CKV_AWS_7     | /aws/kms.tf                   | aws_kms_key.logs_key                                | Ensure rotation for customer created CMKs is enabled                                                         | https://docs.bridgecrew.io/docs/logging_8                                                                                                    |
|  38 | CKV_AWS_50    | /aws/lambda.tf                | aws_lambda_function.analysis_lambda                 | X-ray tracing is enabled for Lambda                                                                          | https://docs.bridgecrew.io/docs/bc_aws_serverless_4                                                                                          |
|  39 | CKV_AWS_116   | /aws/lambda.tf                | aws_lambda_function.analysis_lambda                 | Ensure that AWS Lambda function is configured for a Dead Letter Queue(DLQ)                                   | https://docs.bridgecrew.io/docs/ensure-that-aws-lambda-function-is-configured-for-a-dead-letter-queue-dlq                                    |
|  40 | CKV_AWS_115   | /aws/lambda.tf                | aws_lambda_function.analysis_lambda                 | Ensure that AWS Lambda function is configured for function-level concurrent execution limit                  | https://docs.bridgecrew.io/docs/ensure-that-aws-lambda-function-is-configured-for-function-level-concurrent-execution-limit                  |
|  41 | CKV_AWS_45    | /aws/lambda.tf                | aws_lambda_function.analysis_lambda                 | Ensure no hard-coded secrets exist in lambda environment                                                     | https://docs.bridgecrew.io/docs/bc_aws_secrets_3                                                                                             |
|  42 | CKV_AWS_117   | /aws/lambda.tf                | aws_lambda_function.analysis_lambda                 | Ensure that AWS Lambda function is configured inside a VPC                                                   | https://docs.bridgecrew.io/docs/ensure-that-aws-lambda-function-is-configured-inside-a-vpc-1                                                 |
|  43 | CKV_AWS_272   | /aws/lambda.tf                | aws_lambda_function.analysis_lambda                 | Ensure AWS Lambda function is configured to validate code-signing                                            |                                                                                                                                              |
|  44 | CKV_AWS_173   | /aws/lambda.tf                | aws_lambda_function.analysis_lambda                 | Check encryption settings for Lambda environmental variable                                                  | https://docs.bridgecrew.io/docs/bc_aws_serverless_5                                                                                          |
|  45 | CKV_AWS_101   | /aws/neptune.tf               | aws_neptune_cluster.default                         | Ensure Neptune logging is enabled                                                                            | https://docs.bridgecrew.io/docs/bc_aws_logging_24                                                                                            |
|  46 | CKV_AWS_44    | /aws/neptune.tf               | aws_neptune_cluster.default                         | Ensure Neptune storage is securely encrypted                                                                 | https://docs.bridgecrew.io/docs/general_18                                                                                                   |
|  47 | CKV_AWS_280   | /aws/neptune.tf               | aws_neptune_cluster_snapshot.default                | Ensure Neptune snapshot is encrypted by KMS using a customer managed Key (CMK)                               |                                                                                                                                              |
|  48 | CKV_AWS_279   | /aws/neptune.tf               | aws_neptune_cluster_snapshot.default                | Ensure Neptune snapshot is securely encrypted                                                                |                                                                                                                                              |
|  49 | CKV_AWS_41    | /aws/providers.tf             | aws.plain_text_access_keys_provider                 | Ensure no hard coded AWS access key and secret key exists in provider                                        | https://docs.bridgecrew.io/docs/bc_aws_secrets_5                                                                                             |
|  50 | CKV_AZURE_172 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure autorotation of Secrets Store CSI Driver secrets for AKS clusters                                     |                                                                                                                                              |
|  51 | CKV_AZURE_141 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure AKS local admin account is disabled                                                                   | https://docs.bridgecrew.io/docs/ensure-azure-kubernetes-service-aks-local-admin-account-is-disabled                                          |
|  52 | CKV_AZURE_170 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure that AKS use the Paid Sku for its SLA                                                                 |                                                                                                                                              |
|  53 | CKV_AZURE_7   | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure AKS cluster has Network Policy configured                                                             | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_4                                                                                          |
|  54 | CKV_AZURE_5   | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure RBAC is enabled on AKS clusters                                                                       | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_2                                                                                          |
|  55 | CKV_AZURE_8   | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure Kubernetes Dashboard is disabled                                                                      | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_5                                                                                          |
|  56 | CKV_AZURE_115 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure that AKS enables private clusters                                                                     | https://docs.bridgecrew.io/docs/ensure-that-aks-enables-private-clusters                                                                     |
|  57 | CKV_AZURE_168 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure Azure Kubernetes Cluster (AKS) nodes should use a minimum number of 50 pods.                          |                                                                                                                                              |
|  58 | CKV_AZURE_116 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure that AKS uses Azure Policies Add-on                                                                   | https://docs.bridgecrew.io/docs/ensure-that-aks-uses-azure-policies-add-on                                                                   |
|  59 | CKV_AZURE_117 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure that AKS uses disk encryption set                                                                     | https://docs.bridgecrew.io/docs/ensure-that-aks-uses-disk-encryption-set                                                                     |
|  60 | CKV_AZURE_171 | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure AKS cluster upgrade channel is chosen                                                                 |                                                                                                                                              |
|  61 | CKV_AZURE_6   | /azure/aks.tf                 | azurerm_kubernetes_cluster.k8s_cluster              | Ensure AKS has an API Server Authorized IP Ranges enabled                                                    | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_3                                                                                          |
|  62 | CKV_AZURE_63  | /azure/app_service.tf         | azurerm_app_service.app-service1                    | Ensure that App service enables HTTP logging                                                                 | https://docs.bridgecrew.io/docs/ensure-that-app-service-enables-http-logging                                                                 |
|  63 | CKV_AZURE_80  | /azure/app_service.tf         | azurerm_app_service.app-service1                    | Ensure that 'Net Framework' version is the latest, if used as a part of the web app                          | https://docs.bridgecrew.io/docs/ensure-that-net-framework-version-is-the-latest-if-used-as-a-part-of-the-web-app                             |
|  64 | CKV_AZURE_65  | /azure/app_service.tf         | azurerm_app_service.app-service1                    | Ensure that App service enables detailed error messages                                                      | https://docs.bridgecrew.io/docs/tbdensure-that-app-service-enables-detailed-error-messages                                                   |
|  65 | CKV_AZURE_16  | /azure/app_service.tf         | azurerm_app_service.app-service1                    | Ensure that Register with Azure Active Directory is enabled on App Service                                   | https://docs.bridgecrew.io/docs/bc_azr_iam_1                                                                                                 |
|  66 | CKV_AZURE_71  | /azure/app_service.tf         | azurerm_app_service.app-service1                    | Ensure that Managed identity provider is enabled for app services                                            | https://docs.bridgecrew.io/docs/ensure-that-managed-identity-provider-is-enabled-for-app-services                                            |
|  67 | CKV_AZURE_66  | /azure/app_service.tf         | azurerm_app_service.app-service1                    | Ensure that App service enables failed request tracing                                                       | https://docs.bridgecrew.io/docs/ensure-that-app-service-enables-failed-request-tracing                                                       |
|  68 | CKV_AZURE_88  | /azure/app_service.tf         | azurerm_app_service.app-service1                    | Ensure that app services use Azure Files                                                                     | https://docs.bridgecrew.io/docs/ensure-that-app-services-use-azure-files                                                                     |
|  69 | CKV_AZURE_63  | /azure/app_service.tf         | azurerm_app_service.app-service2                    | Ensure that App service enables HTTP logging                                                                 | https://docs.bridgecrew.io/docs/ensure-that-app-service-enables-http-logging                                                                 |
|  70 | CKV_AZURE_80  | /azure/app_service.tf         | azurerm_app_service.app-service2                    | Ensure that 'Net Framework' version is the latest, if used as a part of the web app                          | https://docs.bridgecrew.io/docs/ensure-that-net-framework-version-is-the-latest-if-used-as-a-part-of-the-web-app                             |
|  71 | CKV_AZURE_65  | /azure/app_service.tf         | azurerm_app_service.app-service2                    | Ensure that App service enables detailed error messages                                                      | https://docs.bridgecrew.io/docs/tbdensure-that-app-service-enables-detailed-error-messages                                                   |
|  72 | CKV_AZURE_16  | /azure/app_service.tf         | azurerm_app_service.app-service2                    | Ensure that Register with Azure Active Directory is enabled on App Service                                   | https://docs.bridgecrew.io/docs/bc_azr_iam_1                                                                                                 |
|  73 | CKV_AZURE_71  | /azure/app_service.tf         | azurerm_app_service.app-service2                    | Ensure that Managed identity provider is enabled for app services                                            | https://docs.bridgecrew.io/docs/ensure-that-managed-identity-provider-is-enabled-for-app-services                                            |
|  74 | CKV_AZURE_66  | /azure/app_service.tf         | azurerm_app_service.app-service2                    | Ensure that App service enables failed request tracing                                                       | https://docs.bridgecrew.io/docs/ensure-that-app-service-enables-failed-request-tracing                                                       |
|  75 | CKV_AZURE_88  | /azure/app_service.tf         | azurerm_app_service.app-service2                    | Ensure that app services use Azure Files                                                                     | https://docs.bridgecrew.io/docs/ensure-that-app-services-use-azure-files                                                                     |
|  76 | CKV_AZURE_50  | /azure/instance.tf            | azurerm_linux_virtual_machine.linux_machine         | Ensure Virtual Machine Extensions are not Installed                                                          | https://docs.bridgecrew.io/docs/bc_azr_general_14                                                                                            |
|  77 | CKV_AZURE_178 | /azure/instance.tf            | azurerm_linux_virtual_machine.linux_machine         | Ensure Windows VM enables SSH with keys for secure communication                                             |                                                                                                                                              |
|  78 | CKV_AZURE_149 | /azure/instance.tf            | azurerm_linux_virtual_machine.linux_machine         | Ensure that Virtual machine does not enable password authentication                                          | https://docs.bridgecrew.io/docs/ensure-azure-virtual-machine-does-not-enable-password-authentication                                         |
|  79 | CKV_AZURE_1   | /azure/instance.tf            | azurerm_linux_virtual_machine.linux_machine         | Ensure Azure Instance does not use basic authentication(Use SSH Key Instead)                                 | https://docs.bridgecrew.io/docs/bc_azr_networking_1                                                                                          |
|  80 | CKV_AZURE_179 | /azure/instance.tf            | azurerm_linux_virtual_machine.linux_machine         | Ensure VM agent is installed                                                                                 |                                                                                                                                              |
|  81 | CKV_AZURE_50  | /azure/instance.tf            | azurerm_windows_virtual_machine.windows_machine     | Ensure Virtual Machine Extensions are not Installed                                                          | https://docs.bridgecrew.io/docs/bc_azr_general_14                                                                                            |
|  82 | CKV_AZURE_177 | /azure/instance.tf            | azurerm_windows_virtual_machine.windows_machine     | Ensure Windows VM enables automatic updates                                                                  |                                                                                                                                              |
|  83 | CKV_AZURE_179 | /azure/instance.tf            | azurerm_windows_virtual_machine.windows_machine     | Ensure VM agent is installed                                                                                 |                                                                                                                                              |
|  84 | CKV_AZURE_151 | /azure/instance.tf            | azurerm_windows_virtual_machine.windows_machine     | Ensure Windows VM enables encryption                                                                         | https://docs.bridgecrew.io/docs/ensure-azure-windows-vm-enables-encryption                                                                   |
|  85 | CKV_AZURE_109 | /azure/key_vault.tf           | azurerm_key_vault.example                           | Ensure that key vault allows firewall rules settings                                                         | https://docs.bridgecrew.io/docs/ensure-that-key-vault-allows-firewall-rules-settings                                                         |
|  86 | CKV_AZURE_40  | /azure/key_vault.tf           | azurerm_key_vault_key.generated                     | Ensure that the expiration date is set on all keys                                                           | https://docs.bridgecrew.io/docs/set-an-expiration-date-on-all-keys                                                                           |
|  87 | CKV_AZURE_112 | /azure/key_vault.tf           | azurerm_key_vault_key.generated                     | Ensure that key vault key is backed by HSM                                                                   | https://docs.bridgecrew.io/docs/ensure-that-key-vault-key-is-backed-by-hsm                                                                   |
|  88 | CKV_AZURE_114 | /azure/key_vault.tf           | azurerm_key_vault_secret.secret                     | Ensure that key vault secrets have "content_type" set                                                        | https://docs.bridgecrew.io/docs/ensure-that-key-vault-secrets-have-content_type-set                                                          |
|  89 | CKV_AZURE_41  | /azure/key_vault.tf           | azurerm_key_vault_secret.secret                     | Ensure that the expiration date is set on all secrets                                                        | https://docs.bridgecrew.io/docs/set-an-expiration-date-on-all-secrets                                                                        |
|  90 | CKV_AZURE_38  | /azure/logging.tf             | azurerm_monitor_log_profile.logging_profile         | Ensure audit profile captures all the activities                                                             | https://docs.bridgecrew.io/docs/ensure-audit-profile-captures-all-activities                                                                 |
|  91 | CKV_AZURE_37  | /azure/logging.tf             | azurerm_monitor_log_profile.logging_profile         | Ensure that Activity Log Retention is set 365 days or greater                                                | https://docs.bridgecrew.io/docs/set-activity-log-retention-to-365-days-or-greater                                                            |
|  92 | CKV_AZURE_10  | /azure/networking.tf          | azurerm_network_security_group.bad_sg               | Ensure that SSH access is restricted from the internet                                                       | https://docs.bridgecrew.io/docs/bc_azr_networking_3                                                                                          |
|  93 | CKV_AZURE_9   | /azure/networking.tf          | azurerm_network_security_group.bad_sg               | Ensure that RDP access is restricted from the internet                                                       | https://docs.bridgecrew.io/docs/bc_azr_networking_2                                                                                          |
|  94 | CKV_AZURE_12  | /azure/networking.tf          | azurerm_network_watcher_flow_log.flow_log           | Ensure that Network Security Group Flow Log retention period is 'greater than 90 days'                       | https://docs.bridgecrew.io/docs/bc_azr_logging_1                                                                                             |
|  95 | CKV_AZURE_39  | /azure/roles.tf               | azurerm_role_definition.example                     | Ensure that no custom subscription owner roles are created                                                   | https://docs.bridgecrew.io/docs/do-not-create-custom-subscription-owner-roles                                                                |
|  96 | CKV_AZURE_21  | /azure/security_center.tf     | azurerm_security_center_contact.contact             | Ensure that 'Send email notification for high severity alerts' is set to 'On'                                | https://docs.bridgecrew.io/docs/bc_azr_general_4                                                                                             |
|  97 | CKV_AZURE_20  | /azure/security_center.tf     | azurerm_security_center_contact.contact             | Ensure that security contact 'Phone number' is set                                                           | https://docs.bridgecrew.io/docs/bc_azr_general_3                                                                                             |
|  98 | CKV_AZURE_26  | /azure/sql.tf                 | azurerm_mssql_server_security_alert_policy.example  | Ensure that 'Send Alerts To' is enabled for MSSQL servers                                                    | https://docs.bridgecrew.io/docs/bc_azr_general_7                                                                                             |
|  99 | CKV_AZURE_25  | /azure/sql.tf                 | azurerm_mssql_server_security_alert_policy.example  | Ensure that 'Threat Detection types' is set to 'All'                                                         | https://docs.bridgecrew.io/docs/bc_azr_general_6                                                                                             |
| 100 | CKV_AZURE_53  | /azure/sql.tf                 | azurerm_mysql_server.example                        | Ensure 'public network access enabled' is set to 'False' for mySQL servers                                   | https://docs.bridgecrew.io/docs/ensure-public-network-access-enabled-is-set-to-false-for-mysql-servers                                       |
| 101 | CKV_AZURE_54  | /azure/sql.tf                 | azurerm_mysql_server.example                        | Ensure MySQL is using the latest version of TLS encryption                                                   | https://docs.bridgecrew.io/docs/ensure-mysql-is-using-the-latest-version-of-tls-encryption                                                   |
| 102 | CKV_AZURE_94  | /azure/sql.tf                 | azurerm_mysql_server.example                        | Ensure that My SQL server enables geo-redundant backups                                                      | https://docs.bridgecrew.io/docs/ensure-that-my-sql-server-enables-geo-redundant-backups                                                      |
| 103 | CKV_AZURE_127 | /azure/sql.tf                 | azurerm_mysql_server.example                        | Ensure that My SQL server enables Threat detection policy                                                    | https://docs.bridgecrew.io/docs/ensure-that-my-sql-server-enables-threat-detection-policy                                                    |
| 104 | CKV_AZURE_128 | /azure/sql.tf                 | azurerm_postgresql_server.example                   | Ensure that PostgreSQL server enables Threat detection policy                                                | https://docs.bridgecrew.io/docs/ensure-that-postgresql-server-enables-threat-detection-policy                                                |
| 105 | CKV_AZURE_68  | /azure/sql.tf                 | azurerm_postgresql_server.example                   | Ensure that PostgreSQL server disables public network access                                                 | https://docs.bridgecrew.io/docs/ensure-that-postgresql-server-disables-public-network-access                                                 |
| 106 | CKV_AZURE_147 | /azure/sql.tf                 | azurerm_postgresql_server.example                   | Ensure PostgreSQL is using the latest version of TLS encryption                                              | https://docs.bridgecrew.io/docs/ensure-azure-postgresql-uses-the-latest-version-of-tls-encryption                                            |
| 107 | CKV_AZURE_130 | /azure/sql.tf                 | azurerm_postgresql_server.example                   | Ensure that PostgreSQL server enables infrastructure encryption                                              | https://docs.bridgecrew.io/docs/ensure-that-postgresql-server-enables-infrastructure-encryption                                              |
| 108 | CKV_AZURE_102 | /azure/sql.tf                 | azurerm_postgresql_server.example                   | Ensure that PostgreSQL server enables geo-redundant backups                                                  | https://docs.bridgecrew.io/docs/ensure-that-postgresql-server-enables-geo-redundant-backups                                                  |
| 109 | CKV_AZURE_32  | /azure/sql.tf                 | azurerm_postgresql_configuration.thrtottling_config | Ensure server parameter 'connection_throttling' is set to 'ON' for PostgreSQL Database Server                | https://docs.bridgecrew.io/docs/bc_azr_networking_13                                                                                         |
| 110 | CKV_AZURE_30  | /azure/sql.tf                 | azurerm_postgresql_configuration.example            | Ensure server parameter 'log_checkpoints' is set to 'ON' for PostgreSQL Database Server                      | https://docs.bridgecrew.io/docs/bc_azr_networking_11                                                                                         |
| 111 | CKV_AZURE_2   | /azure/storage.tf             | azurerm_managed_disk.example                        | Ensure Azure managed disk has encryption enabled                                                             | https://docs.bridgecrew.io/docs/bc_azr_general_1                                                                                             |
| 112 | CKV_AZURE_93  | /azure/storage.tf             | azurerm_managed_disk.example                        | Ensure that managed disks use a specific set of disk encryption sets for the customer-managed key encryption | https://docs.bridgecrew.io/docs/ensure-that-managed-disks-use-a-specific-set-of-disk-encryption-sets-for-the-customer-managed-key-encryption |
| 113 | CKV_AZURE_33  | /azure/storage.tf             | azurerm_storage_account.example                     | Ensure Storage logging is enabled for Queue service for read, write and delete requests                      | https://docs.bridgecrew.io/docs/enable-requests-on-storage-logging-for-queue-service                                                         |
| 114 | CKV_AZURE_44  | /azure/storage.tf             | azurerm_storage_account.example                     | Ensure Storage Account is using the latest version of TLS encryption                                         | https://docs.bridgecrew.io/docs/bc_azr_storage_2                                                                                             |
| 115 | CKV_AZURE_36  | /azure/storage.tf             | azurerm_storage_account_network_rules.test          | Ensure 'Trusted Microsoft Services' is enabled for Storage Account access                                    | https://docs.bridgecrew.io/docs/enable-trusted-microsoft-services-for-storage-account-access                                                 |
| 116 | CKV_GCP_51    | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure PostgreSQL database 'log_checkpoints' flag is set to 'on'                                             | https://docs.bridgecrew.io/docs/bc_gcp_sql_2                                                                                                 |
| 117 | CKV_GCP_54    | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure PostgreSQL database 'log_lock_waits' flag is set to 'on'                                              | https://docs.bridgecrew.io/docs/bc_gcp_sql_5                                                                                                 |
| 118 | CKV_GCP_52    | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure PostgreSQL database 'log_connections' flag is set to 'on'                                             | https://docs.bridgecrew.io/docs/bc_gcp_sql_3                                                                                                 |
| 119 | CKV_GCP_108   | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure hostnames are logged for GCP PostgreSQL databases                                                     |                                                                                                                                              |
| 120 | CKV_GCP_53    | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure PostgreSQL database 'log_disconnections' flag is set to 'on'                                          | https://docs.bridgecrew.io/docs/bc_gcp_sql_4                                                                                                 |
| 121 | CKV_GCP_11    | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure that Cloud SQL database Instances are not open to the world                                           | https://docs.bridgecrew.io/docs/bc_gcp_networking_4                                                                                          |
| 122 | CKV_GCP_79    | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure SQL database is using latest Major version                                                            | https://docs.bridgecrew.io/docs/ensure-gcp-sql-database-uses-the-latest-major-version                                                        |
| 123 | CKV_GCP_110   | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure pgAudit is enabled for your GCP PostgreSQL database                                                   |                                                                                                                                              |
| 124 | CKV_GCP_60    | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure Cloud SQL database does not have public IP                                                            | https://docs.bridgecrew.io/docs/bc_gcp_sql_11                                                                                                |
| 125 | CKV_GCP_111   | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure GCP PostgreSQL logs SQL statements                                                                    |                                                                                                                                              |
| 126 | CKV_GCP_109   | /gcp/big_data.tf              | google_sql_database_instance.master_instance        | Ensure the GCP PostgreSQL database log levels are set to ERROR or lower                                      |                                                                                                                                              |
| 127 | CKV_GCP_81    | /gcp/big_data.tf              | google_bigquery_dataset.dataset                     | Ensure Big Query Tables are encrypted with Customer Supplied Encryption Keys (CSEK)                          | https://docs.bridgecrew.io/docs/ensure-gcp-big-query-tables-are-encrypted-with-customer-supplied-encryption-keys-csek-1                      |
| 128 | CKV_GCP_15    | /gcp/big_data.tf              | google_bigquery_dataset.dataset                     | Ensure that BigQuery datasets are not anonymously or publicly accessible                                     | https://docs.bridgecrew.io/docs/bc_gcp_general_3                                                                                             |
| 129 | CKV_GCP_78    | /gcp/gcs.tf                   | google_storage_bucket.terragoat_website             | Ensure Cloud storage has versioning enabled                                                                  | https://docs.bridgecrew.io/docs/ensure-gcp-cloud-storage-has-versioning-enabled                                                              |
| 130 | CKV_GCP_62    | /gcp/gcs.tf                   | google_storage_bucket.terragoat_website             | Bucket should log access                                                                                     | https://docs.bridgecrew.io/docs/bc_gcp_logging_2                                                                                             |
| 131 | CKV_GCP_28    | /gcp/gcs.tf                   | google_storage_bucket_iam_binding.allow_public_read | Ensure that Cloud Storage bucket is not anonymously or publicly accessible                                   | https://docs.bridgecrew.io/docs/bc_gcp_public_1                                                                                              |
| 132 | CKV_GCP_8     | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure Stackdriver Monitoring is set to Enabled on Kubernetes Engine Clusters                                | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_3                                                                                          |
| 133 | CKV_GCP_18    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure GKE Control Plane is not public                                                                       | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_10                                                                                         |
| 134 | CKV_GCP_69    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure the GKE Metadata Server is Enabled                                                                    | https://docs.bridgecrew.io/docs/ensure-the-gke-metadata-server-is-enabled                                                                    |
| 135 | CKV_GCP_19    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure GKE basic auth is disabled                                                                            | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_11                                                                                         |
| 136 | CKV_GCP_21    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure Kubernetes Clusters are configured with Labels                                                        | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_13                                                                                         |
| 137 | CKV_GCP_65    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Manage Kubernetes RBAC users with Google Groups for GKE                                                      | https://docs.bridgecrew.io/docs/manage-kubernetes-rbac-users-with-google-groups-for-gke                                                      |
| 138 | CKV_GCP_23    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure Kubernetes Cluster is created with Alias IP ranges enabled                                            | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_15                                                                                         |
| 139 | CKV_GCP_13    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure client certificate authentication to Kubernetes Engine Clusters is disabled                           | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_8                                                                                          |
| 140 | CKV_GCP_25    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure Kubernetes Cluster is created with Private cluster enabled                                            | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_6                                                                                          |
| 141 | CKV_GCP_1     | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure Stackdriver Logging is set to Enabled on Kubernetes Engine Clusters                                   | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_1                                                                                          |
| 142 | CKV_GCP_64    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure clusters are created with Private Nodes                                                               | https://docs.bridgecrew.io/docs/ensure-clusters-are-created-with-private-nodes                                                               |
| 143 | CKV_GCP_70    | /gcp/gke.tf                   | google_container_cluster.workload_cluster           | Ensure the GKE Release Channel is set                                                                        | https://docs.bridgecrew.io/docs/ensure-the-gke-release-channel-is-set                                                                        |
| 144 | CKV_GCP_69    | /gcp/gke.tf                   | google_container_node_pool.custom_node_pool         | Ensure the GKE Metadata Server is Enabled                                                                    | https://docs.bridgecrew.io/docs/ensure-the-gke-metadata-server-is-enabled                                                                    |
| 145 | CKV_GCP_22    | /gcp/gke.tf                   | google_container_node_pool.custom_node_pool         | Ensure Container-Optimized OS (cos) is used for Kubernetes Engine Clusters Node image                        | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_14                                                                                         |
| 146 | CKV_GCP_72    | /gcp/gke.tf                   | google_container_node_pool.custom_node_pool         | Ensure Integrity Monitoring for Shielded GKE Nodes is Enabled                                                | https://docs.bridgecrew.io/docs/ensure-integrity-monitoring-for-shielded-gke-nodes-is-enabled                                                |
| 147 | CKV_GCP_32    | /gcp/instances.tf             | google_compute_instance.server                      | Ensure 'Block Project-wide SSH keys' is enabled for VM instances                                             | https://docs.bridgecrew.io/docs/bc_gcp_networking_8                                                                                          |
| 148 | CKV_GCP_39    | /gcp/instances.tf             | google_compute_instance.server                      | Ensure Compute instances are launched with Shielded VM enabled                                               | https://docs.bridgecrew.io/docs/bc_gcp_general_y                                                                                             |
| 149 | CKV_GCP_30    | /gcp/instances.tf             | google_compute_instance.server                      | Ensure that instances are not configured to use the default service account                                  | https://docs.bridgecrew.io/docs/bc_gcp_iam_1                                                                                                 |
| 150 | CKV_GCP_38    | /gcp/instances.tf             | google_compute_instance.server                      | Ensure VM disks for critical VMs are encrypted with Customer Supplied Encryption Keys (CSEK)                 | https://docs.bridgecrew.io/docs/encrypt-boot-disks-for-instances-with-cseks                                                                  |
| 151 | CKV_GCP_37    | /gcp/instances.tf             | google_compute_disk.unencrypted_disk                | Ensure VM disks for critical VMs are encrypted with Customer Supplied Encryption Keys (CSEK)                 | https://docs.bridgecrew.io/docs/bc_gcp_general_x                                                                                             |
| 152 | CKV_GCP_26    | /gcp/networks.tf              | google_compute_subnetwork.public-subnetwork         | Ensure that VPC Flow Logs is enabled for every subnet in a VPC Network                                       | https://docs.bridgecrew.io/docs/bc_gcp_logging_1                                                                                             |
| 153 | CKV_GCP_76    | /gcp/networks.tf              | google_compute_subnetwork.public-subnetwork         | Ensure that Private google access is enabled for IPV6                                                        | https://docs.bridgecrew.io/docs/ensure-gcp-private-google-access-is-enabled-for-ipv6                                                         |
| 154 | CKV_GCP_74    | /gcp/networks.tf              | google_compute_subnetwork.public-subnetwork         | Ensure that private_ip_google_access is enabled for Subnet                                                   | https://docs.bridgecrew.io/docs/ensure-gcp-subnet-has-a-private-ip-google-access                                                             |
| 155 | CKV_GCP_75    | /gcp/networks.tf              | google_compute_firewall.allow_all                   | Ensure Google compute firewall ingress does not allow unrestricted FTP access                                | https://docs.bridgecrew.io/docs/ensure-azure-built-in-logging-for-azure-function-app-is-enabled                                              |
| 156 | CKV_GCP_3     | /gcp/networks.tf              | google_compute_firewall.allow_all                   | Ensure Google compute firewall ingress does not allow unrestricted rdp access                                | https://docs.bridgecrew.io/docs/bc_gcp_networking_2                                                                                          |
| 157 | CKV_GCP_88    | /gcp/networks.tf              | google_compute_firewall.allow_all                   | Ensure Google compute firewall ingress does not allow unrestricted mysql access                              | https://docs.bridgecrew.io/docs/ensure-gcp-compute-firewall-ingress-does-not-allow-unrestricted-mysql-access                                 |
| 158 | CKV_GCP_106   | /gcp/networks.tf              | google_compute_firewall.allow_all                   | Ensure Google compute firewall ingress does not allow unrestricted http port 80 access                       | https://docs.bridgecrew.io/docs/ensure-gcp-google-compute-firewall-ingress-does-not-allow-unrestricted-http-port-80-access                   |
| 159 | CKV_GCP_2     | /gcp/networks.tf              | google_compute_firewall.allow_all                   | Ensure Google compute firewall ingress does not allow unrestricted ssh access                                | https://docs.bridgecrew.io/docs/bc_gcp_networking_1                                                                                          |
| 160 | CKV_GCP_77    | /gcp/networks.tf              | google_compute_firewall.allow_all                   | Ensure Google compute firewall ingress does not allow on ftp port                                            | https://docs.bridgecrew.io/docs/ensure-gcp-google-compute-firewall-ingress-does-not-allow-ftp-port-20-access                                 |
| 161 | CKV_AWS_18    | /aws/ec2.tf                   | aws_s3_bucket.flowbucket                            | Ensure the S3 bucket has access logging enabled                                                              | https://docs.bridgecrew.io/docs/s3_13-enable-logging                                                                                         |
| 162 | CKV2_AWS_6    | /aws/ec2.tf                   | aws_s3_bucket.flowbucket                            | Ensure that S3 bucket has a Public Access block                                                              | https://docs.bridgecrew.io/docs/s3-bucket-should-have-public-access-blocks-defaults-to-false-if-the-public-access-block-is-not-attached      |
| 163 | CKV2_AWS_12   | /aws/ec2.tf                   | aws_vpc.web_vpc                                     | Ensure the default security group of every VPC restricts all traffic                                         | https://docs.bridgecrew.io/docs/networking_4                                                                                                 |
| 164 | CKV2_AWS_12   | /aws/eks.tf                   | aws_vpc.eks_vpc                                     | Ensure the default security group of every VPC restricts all traffic                                         | https://docs.bridgecrew.io/docs/networking_4                                                                                                 |
| 165 | CKV2_AWS_11   | /aws/eks.tf                   | aws_vpc.eks_vpc                                     | Ensure VPC flow logging is enabled in all VPCs                                                               | https://docs.bridgecrew.io/docs/logging_9-enable-vpc-flow-logging                                                                            |
| 166 | CKV2_AWS_41   | /aws/ec2.tf                   | aws_instance.web_host                               | Ensure an IAM role is attached to EC2 instance                                                               |                                                                                                                                              |
| 167 | CKV2_AZURE_16 | /azure/sql.tf                 | azurerm_mysql_server.example                        | Ensure that MySQL server enables customer-managed key for encryption                                         | https://docs.bridgecrew.io/docs/ensure-that-mysql-server-enables-customer-managed-key-for-encryption                                         |
| 168 | CKV_AZURE_120 | /azure/application_gateway.tf | azurerm_application_gateway.network                 | Ensure that Application Gateway enables WAF                                                                  | https://docs.bridgecrew.io/docs/ensure-that-application-gateway-enables-waf                                                                  |
| 169 | CKV_AWS_144   | /aws/ec2.tf                   | aws_s3_bucket.flowbucket                            | Ensure that S3 bucket has cross-region replication enabled                                                   | https://docs.bridgecrew.io/docs/ensure-that-s3-bucket-has-cross-region-replication-enabled                                                   |
| 170 | CKV_AWS_145   | /aws/ec2.tf                   | aws_s3_bucket.flowbucket                            | Ensure that S3 buckets are encrypted with KMS by default                                                     | https://docs.bridgecrew.io/docs/ensure-that-s3-buckets-are-encrypted-with-kms-by-default                                                     |
| 171 | CKV2_AZURE_7  | /azure/sql.tf                 | azurerm_sql_server.example                          | Ensure that Azure Active Directory Admin is configured                                                       | https://docs.bridgecrew.io/docs/ensure-that-azure-active-directory-admin-is-configured                                                       |
| 172 | CKV2_AZURE_18 | /azure/storage.tf             | azurerm_storage_account.example                     | Ensure that Storage Accounts use customer-managed key for encryption                                         | https://docs.bridgecrew.io/docs/ensure-that-storage-accounts-use-customer-managed-key-for-encryption                                         |
| 173 | CKV_AZURE_24  | /azure/sql.tf                 | azurerm_sql_server.example                          | Ensure that 'Auditing' Retention is 'greater than 90 days' for SQL servers                                   | https://docs.bridgecrew.io/docs/bc_azr_logging_3                                                                                             |
| 174 | CKV2_AZURE_1  | /azure/storage.tf             | azurerm_storage_account.example                     | Ensure storage for critical data are encrypted with Customer Managed Key                                     | https://docs.bridgecrew.io/docs/ensure-storage-for-critical-data-are-encrypted-with-customer-managed-key                                     |
| 175 | CKV_AZURE_23  | /azure/sql.tf                 | azurerm_sql_server.example                          | Ensure that 'Auditing' is set to 'On' for SQL servers                                                        | https://docs.bridgecrew.io/docs/bc_azr_logging_2                                                                                             |

---
### dockerfile scan results:

|    | check_id     | file                      | resource                   | check_name                                                               | guideline                                                                                                |
|----|--------------|---------------------------|----------------------------|--------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
|  0 | CKV_DOCKER_2 | /aws/resources/Dockerfile | /aws/resources/Dockerfile. | Ensure that HEALTHCHECK instructions have been added to container images | https://docs.bridgecrew.io/docs/ensure-that-healthcheck-instructions-have-been-added-to-container-images |
|  1 | CKV_DOCKER_3 | /aws/resources/Dockerfile | /aws/resources/Dockerfile. | Ensure that a user for the container has been created                    | https://docs.bridgecrew.io/docs/ensure-that-a-user-for-the-container-has-been-created                    |

---
### secrets scan results:

|    | check_id     | file              | resource                                 | check_name                 | guideline                                     |
|----|--------------|-------------------|------------------------------------------|----------------------------|-----------------------------------------------|
|  0 | CKV_SECRET_2 | /aws/ec2.tf       | fc3f784491eba6121c3bfcc1652a2c57d27b16cb | AWS Access Key             | https://docs.bridgecrew.io/docs/git_secrets_2 |
|  1 | CKV_SECRET_2 | /aws/lambda.tf    | 25910f981e85ca04baf359199dd0bd4a3ae738b6 | AWS Access Key             | https://docs.bridgecrew.io/docs/git_secrets_2 |
|  2 | CKV_SECRET_6 | /aws/lambda.tf    | d70eab08607a4d05faa2d0d6647206599e9abc65 | Base64 High Entropy String | https://docs.bridgecrew.io/docs/git_secrets_6 |
|  3 | CKV_SECRET_2 | /aws/providers.tf | 25910f981e85ca04baf359199dd0bd4a3ae738b6 | AWS Access Key             | https://docs.bridgecrew.io/docs/git_secrets_2 |
|  4 | CKV_SECRET_6 | /aws/providers.tf | d70eab08607a4d05faa2d0d6647206599e9abc65 | Base64 High Entropy String | https://docs.bridgecrew.io/docs/git_secrets_6 |
|  5 | CKV_SECRET_6 | /azure/sql.tf     | a57ae0fe47084bc8a05f69f3f8083896f8b437b0 | Base64 High Entropy String | https://docs.bridgecrew.io/docs/git_secrets_6 |

---

