# Terraform & Serverless 1:  Overview

## Example code

* [terraform-and-serverless-demo](https://github.com/stablecaps/terraform_and_serverless_demo/tree/master)

## Overview

**1. Step1:**
A company allows their users to upload pictures to an S3 bucket. These pictures are always in the .jpg format. The company wants these files to be stripped from any exif metadata before being shown on their website. Pictures are uploaded to an S3 bucket A. Create a system that retrieves .jpg files when they are uploaded to the S3 bucket A, removes any exif metadata, and save them to another S3 bucket B. The path of the files should be the same in buckets A and B.

![Brief](https://github.com/stablecaps/terraform_and_serverless_demo/blob/master/docs/image_structure.png)

**2. Step2:**
To extend this further, we have two users User A and User B. Create IAM users with the following access:
* User A can Read/Write to Bucket A
* User B can Read from Bucket B

## Solution Overview

![Exif-ripper architecture](https://github.com/stablecaps/terraform_and_serverless_demo/blob/master/docs/exif_ripper.drawio.png)

A natural solution for this problem is to use AWS lambda because this service provides the ability to monitor an S3 bucket and trigger event based messages that can be sent to any arbitrary downstream image processor. Indeed, a whole pipeline of lambda functions can used in the "chain of responsibility pattern" if desired.

![Chain of responsibility](https://github.com/stablecaps/terraform_and_serverless_demo/blob/master/docs/Chained-Microservices-Design-Pattern.png)

Given the limited time to accomplish this task, a benefit of using Serverless is that it is trivial to set up monitoring on a bucket for pushed images because the framework creates the monitoring lambda for us in a few lines of code. When the following code is added to the Serverless.yml file, the monitoring lambda pushes an [event](https://www.Serverless.com/framework/docs/providers/aws/events/s3) to the custom exif-ripper lambda when a file with the suffix of `.jpg` and a S3 key prefix of `incoming/` is created in the bucket called `mysource-bucket`.

```yaml
functions:
  exif:
    handler: exif.handler
    events:
      - s3:
          bucket: mysource-bucket
          event: s3:ObjectCreated:*
          rules:
            - prefix: uploads/
            - suffix: .jpg
```

The lambda Python3 code for exif-ripper is located in `Serverless/exif-ripper/` and it leverages the following libraries to execute the following workflow:
1. [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#s3): Read binary image file from S3 into RAM.
2. [exif](https://pypi.org/project/exif/): Strips any exif data from image
3. Use Boto3 again to write the sanitised file to the destination bucket.

### Deployment tool overview
The solution for this problem was solved using the following deployment tools:
1. Terraform - To create common shared resources such as IAM entities & policies, security groups, sqs queues, S3 buckets, databases, etc
2. Serverless (sls) - To deploy lambda functions - lambda functions, API Gateways, step functions, etc

Sls is a better choice to deploy app-specific infrastructure because the Serverless.yml is tightly coupled to the app code and thus changes can be easily made in a few lines of code compared to the several pages required for Terraform. Additionally, it allows developers to easily modify the infrastructure without asking for DevOps help (as not everyone knows the Terraform DSL). However, It can be a bad idea to allow anyone with access to a Serverless.yml file to deploy whatever they like. Thus, shared-infrastructure that is more stateful is deployed solely by Terraform and ideally this paradigm would be enforced by appropriate IAM permissions. Methods to pass & share data between the two frameworks include the following:

1. Using Terraform data
2. Reading remote terraform state
3. Writing and consuming variables from SSM
4. Using `terraform out` to write out output values to file
5. Using AWS CLI to retrieve data

Note that options 3 & 4 are used in this example repo.


Further reading:
1. [The definitive guide to using Terraform with the Serverless Framework](https://www.serverless.com/blog/definitive-guide-terraform-serverless)
2. [Terraform & Serverless framework, a match made in heaven?](https://medium.com/contino-engineering/terraform-Serverless-framework-a-match-made-in-heaven-part-i-69af51155e00)
3. [A Beginner's Guide to Terraform and Serverless](https://blog.scottlogic.com/2020/01/21/beginners-terraform-serverless.html)


### Serverless Function Overview
Exif-Ripper is a Serverless application that creates an event triggering lambda that monitors a source S3 bucket for the upload of jpg files. When this occurs, an AWS event invokes another (Python3) lambda function that strips the exif data from the jpg and writes the "sanitised" jpg to a destination bucket. This lambda function also reads & processes the image directly in memory, and thus does not incur write time-penalties by writing the file to scratch.
