# Editing Terraform & Serverless 3: TF Patterns

### Terraform code structure Overview
A few patterns of organising and deploying Terraform code are illustrated in this repo's example code. This is a large topic and there is no "one" right answer as it depends on the needs and scale of your organisation.

1. Monolith or multi-repo pattern?
2. Local or remote state?
3. Local and/or remote modules?
4. Organisational IaC architecture that addresses separation of concerns & scalability. DRY code and flexibility invariably results in an increase in complexity; the balance of which depends on the company's needs.


##### [5 Common Terraform Patterns (Click embedded video below)](https://www.youtube.com/watch?v=wgzgVm7Sqlk)
[![Evolving Your Infrastructure with Terraform" TEXT](https://github.com/stablecaps/terraform_and_serverless_demo/blob/master/docs/evolving_terraform_thumb.png)](https://www.youtube.com/watch?v=wgzgVm7Sqlk "5 Common Terraform Patterns")

Some of the pertinent questions with regards to how terraform code is structured are listed below, but a detailed discussion is beyond the scope of this document.

1. `terraform_v1` - [The simplest method](https://github.com/stablecaps/terraform_and_serverless_demo/blob/master/xxx_pipeline_create.sh#L44-L47)
    - Uses a local state file so the terraform.tfstate file is saved to the local disk. In order to facilitate shared team editing, the state file is typically stored in git. This is a potential security concern as sensitive values can be exposed.
    - Once the DEV environment is created, it can be copied and pasted to create UAT & DEV environments. Only a few values such as env value (e.g. `dev --> uat`) will have to be changed in the new env. However, the resulting code duplication can result in env-variant configuration drift and uncaught errors.
    - Uses publicly available remote modules from the [Terraform registry](https://registry.terraform.io/) for resources such as S3 to avoid reinventing the wheel.
    - Uses local modules that are nested in the root of `terraform_v1`. This is a step in the right direction, but any modules defined here cannot be reused for other Terraform consumers. Furthermore, there is no module versioning and changes to these modules will be applicable to DEV, UAT & PROD. We can work around this by checking out specific branches in CI/CD in an env-specific manner, but this is a clunky solution that has suboptimal visibility.
2. `terraform_v2` - [A DRY method](https://github.com/stablecaps/terraform_and_serverless_demo/blob/master/xxx_tfver2_pipeline_create.sh#L73-L76)
    - Uses a remote s3/dynamodb backend with remote state locking. Facilitates multi-user collaboration
    - DRY: Leverages passing in tfvar variables (stored in the envs folder) via the `-var-file` CLI argument. e.g. `terraform init -backend-config=../../envs/${myenv}/${myenv}.backend.hcl`,  followed by `terraform apply -var-file=../../envs/${myenv}/${myenv}.tfvars` A disadvantage is complexity increase and potential accidental deployment to the wrong environment if deploying from the CLI. Usually not such a big problem because CI/CD is used to deploy. However, something to watch out for.
    - Uses custom remote module written by yours truly to provision an IAM role with custom or managed policies. The remote module is versioned with release  tags and can be found here: https://github.com/stablecaps/terraform-aws-iam-policies-stablecaps.


#### Terraform_v1 components & workflow
See `xxx_pipeline_create.sh`

1. Creates a global Serverless deployment bucket which can be used by multiple apps. Multiple Serverless projects can be nested in this bucket. This is to avoid multiple random Serverless buckets being scattered around the root of S3.
2. Creates source & destination S3 buckets for exif image processing
3. Pushes the names of these buckets to SSM
4. Creates a lambda role and policy using a custom remote module pinned to a specific tag
5. Creates two users with RO and RW permissions to the buckets as specified in the brief
6. Uses `Terraform output` to write the role arn & the deployment bucket name to the Serverless folder. Both these variables are used to bootstrap Serverless and thus cannot be retrieved from SSM.

```
.
├── 01_sls_deployment_bucket
├── 02_DEV
├── 03_PROD
└── modules
    ├── exif_ripper_buckets
    ├── iam_exif_users
    └── lambda_iam_role_and_policies

```

#### Terraform_v2 does the following:
**(Please ensure any infra created by v1 is destroyed before deploying v2!)**
This version is included to illustrate a method that is more DRY than v1. See `xxx_tfver2_pipeline_create.sh`
1. Creates a global S3/dynamodb backend and writes the backend config files to envs folder (`00_setup_remote_s3_backend_{dev,prod}`)
2. Creates Serverless deployment bucket. Multiple Serverless projects can be nested in this bucket. This is to avoid the mess of multiple random Serverless buckets being scattered around the root of S3.
3. Creates source & destination S3 buckets for exif image processing
4. Pushes the names of these buckets to SSM
5. Creates a lambda role and policy using a [remote module](https://github.com/stablecaps/terraform-aws-iam-policies-stablecaps).
    - Uses tags so that consumers pin to a specific version of the upstream code
    - Has [scripts](https://github.com/stablecaps/terraform-aws-iam-policies-stablecaps/blob/master/xxx_terraform-docs.sh) to automate README.md creation
    - Has live examples that can be [created](https://github.com/stablecaps/terraform-aws-iam-policies-stablecaps/blob/master/xxx_tests_run_examples.sh) and [destroyed](https://github.com/stablecaps/terraform-aws-iam-policies-stablecaps/blob/master/xxx_tests_destroy_examples.sh) to test any new code that might be merged into master


```
.
├── 00_setup_remote_s3_backend_dev
├── 00_setup_remote_s3_backend_prod
├── entrypoints
│   ├── exifripper_buckets_and_iam_role
│   └── sls_deployment_bucket
├── envs
│   ├── dev
│   └── prod
└── modules
    └── exif_ripper_buckets
```
