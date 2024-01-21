# Terraform & Serverless 4: Deploy notes

## Deployment notes
As S3 buckets must be globally unique, a random string is used so that multiple people can run the deployment in their own environments at any given time without error.


## Practical Usage
**All instructions are for Ubuntu 20.04 using BASH in a terminal, so your mileage may vary if using a different system.
Several scripts have been included to assist getting this solution deployed. Please treat these scripts as additional documentation and give them a read.**

#### Install NVM, Node & Serverless

```bash
cd ~/Downloads
sudo apt install curl
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile
nvm install 14
npm install -g Serverless
cd -
```

#### Please ensure you have exported your aws credentials into your shell
This has been test-deployed into an R&D account using Admin credentials. Try to do the same or use an account with the perms to use lambda, s3, iam, dynamodb, and SSM (systems manager) at the least.

An optional method to get a great bash experience via https://github.com/stablecaps/stablecaps_bashrc

```bash
cd
git clone https://github.com/stablecaps/stablecaps_bashrc.git
mv .bashrc .your_old_bashrc
ln -fs ~/sys_bashrc/_bashrc ~/.bashrc
source ~/.bashrc
```

#### use awskeys command to easily export aws key as env variables with sys_bashrc

```bash
csp1
colsw 72
awskeys help
awskeys list
awskeys export $YOUR_AWS_PROFILE
```

#### Running Deploy Scripts

```bash
### Install packages
sudo apt install eog jq unzip curl wget

### Install latest aws cli
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

### Install terraform_1.0.6
wget https://releases.hashicorp.com/terraform/1.0.6/terraform_1.0.6_linux_amd64.zip
unzip terraform_1.0.6_linux_amd64.zip
sudo mv terraform /usr/local/bin/terraform_v1.0.6
rm -f terraform_1.0.6_linux_amd64.zip
```

#### Terraform_v1 with Serverless application and users (just dev)
```bash
### Create stack From repo root
./xxx_pipeline_create.sh terraform_v1.0.6 $YOUR_TERRAFORM_EXEC $RANDOM_STRING

### Test Serverless function
cd Serverless/exif-ripper
    ./00_test_upload_image_2_s3_source.sh default

    ### Note you can tail serverless logs using
    serverless logs -f exif -t
cd -

### Test user permissions
cd terraform_v1/02_DEV/
    ./000_extract_user_secrets_from_tfstate.sh
    cat ./append_these_perms_to_aws_credentials_file.secrets # <<! take contents of this and paste into ~/.aws/credentials file

    ### run user perm tests & check output
    ./001_test_user_bucket_access.sh
cd -

### DESTROY STACK ONCE FINISHED
./xxx_pipeline_destroy.sh $YOUR_TERRAFORM_EXEC
```

#### Terraform_v2 (NO Serverless or Users) - dev & prod
```bash
### Create stack From repo root
./xxx_tfver2_pipeline_create.sh $YOUR_TERRAFORM_EXEC $RANDOM_STRING

### Look around and check code!!

### DESTROY STACK ONCE FINISHED
./xxx_tfver2_pipeline_destroy.sh $YOUR_TERRAFORM_EXEC $RANDOM_STRING

### Note you will have to manually delete the remote state buckets!
```