# Editing Terraform & Serverless 2: Colocation

### Serverless Function Overview
Exif-Ripper is a Serverless application that creates an event triggering lambda that monitors a source S3 bucket for the upload of jpg files. When this occurs, an AWS event invokes another (Python3) lambda function that strips the exif data from the jpg and writes the "sanitised" jpg to a destination bucket. This lambda function also reads & processes the image directly in memory, and thus does not incur write time-penalties by writing the file to scratch.

#### The Serverless.yml does the following:
See `Serverless/exif-ripper/Serverless.yml`
1. Uses the Serverless deployment bucket created by Terraform
2. Fetches ssm variables that have previously been pushed by the Terraform code: (source and destination buckets)
3. Creates the trigger on the source bucket
4. Creates the lambda function (using buckets created by Terraform)

```
.
└── Serverless
    └── exif-ripper
        ├── config
        └── test_images
```


#### Co-located Monorepo Directory Structure
```
.
├── Serverless
│   └── exif-ripper
│       ├── config
│       └── test_images
├── Terraform_v1
│   ├── 01_sls_deployment_bucket
│   ├── 02_DEV
│   ├── 03_PROD
│   └── modules
│       ├── exif_ripper_buckets
│       ├── iam_exif_users
│       └── lambda_iam_role_and_policies
└── Terraform_v2
    ├── 00_setup_remote_s3_backend_dev
    ├── 00_setup_remote_s3_backend_prod
    ├── entrypoints
    │   ├── exifripper_buckets_and_iam_role
    │   └── sls_deployment_bucket
    ├── envs
    │   ├── dev
    │   └── prod
    └── modules
        └──exif_ripper_buckets
```

The directory structure in this project co-locates the infrastructure code with the dev code. An alternative method is accomplished via separation of the infrastructure code from the dev code into 2 repos:

1. myCompany-test (conatins dev Serverless code)
2. myCompany-test-infra (contains only terraform code)

**Pros and cons of co-location method:**
The primary benefit of co-location of the terraform code within a Serverless project is the ostensible ease of deploying the compressed Serverless zip file from a single directory. [ See ./xxx_pipeline_create.sh](https://github.com/stablecaps/terraform_and_serverless_demo/blob/master/xxx_pipeline_create.sh). This makes sense in the context of this example project because there is a requirement to share an uncomplicated code base.

```
.
└── myCompany-test
    ├── Serverless (code repo)
    ├── Terraform_v1 (terraform repo)
    └── Terraform_v2 (terraform repo)
```

However, if a build server was available, we can escape monorepo-centric notions imposed by the co-location method because commands can be run outside of the context/restrictions of a single monorepo/folder. e.g:

```bash
.
└── build_agent_dir
    ├── myCompany-test  (code repo)
        ├── Serverless
        └── exif-ripper
            ├── config
            └── test_images


### Note infra repo is accessible at another location on the same build server
.
└── /opt/all_terraform_consumers
    └── myCompany-test-infra (terraform repo)
         └── terraform_v1

```


There are several benefits in maintaining the infrastructure code in a separate repo:
1. Increased DevOps agility: Application code is subject to a lengthy build & test process during which an artifact is typically created before it can be deployed. If the Terraform code is tightly coupled to the app code via co-location, then even trivial IaC changes such as changing a tag will result in a long delay before (re)deployment can occur. This is almost always unacceptable.
2. Dev code repos generally have a more complicated git [branching strategy/structure](https://www.flagship.io/git-branching-strategies/). e.g. GitFlow typically has master, develop, feature, release and hotfix branches. Such complexity is usually unsuitable for terraform IaC which typically only requires master and feature branches because terraform IaC can be designed to consume remote modules. As each remote module inhabits it's own git repo, terraform consumers can be [pinned](https://www.terraform.io/language/modules/sources#selecting-a-revision) against any tagged commit in the modules's master branch; or even be pinned against another branch or indeed, any arbitrary commit hash.
