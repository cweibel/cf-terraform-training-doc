# Deploying Cloud Foundry on AWS using Terraform

# Training Manual

Welcome!

This manual will guide you through the steps necessary to configure your computer and the deployment of Cloud Foundry to Amazon Web Services using Terraform.  A tremendous amount of automation has been put in place to allow you to quickly deploy Cloud Foundry in an easy and repeatable way.

# Overview

For this demo we will be using a Stark & Wayne student lab environment on AWS, we will explore each step in the subsequent sections
 - Prerequisites
 - Configure Install
 - Connect to Cloud Foundry
 - Deploy Services
 - Destroy the Deployment

# Exercise 1 - Prerequisites

## AWS Configuration

For the demo we have already created individual AWS accounts and a "jumpbox" on AWS so you do not have to install any tools on your laptop or worry about proxy filtering.

## Log onto the Jumpbox

We have created a server for you to connect to but you will need an ssh key to connect.  Download the key located here: https://gist.github.com/cweibel/ce13bd44b42e700d63a0

Copy the file to your ~/.ssh folder and name the file "demo"

Change the permissions on the file
```
chmod 400 ~/.ssh/demo
```

Now you can log onto the jumpbox
```
ssh -i ~/.ssh/demo ubuntu@training.gotapaas.eu
```

## Pick a student

You will be assigned a student number, once this has been assigned you can navigate to `~/students/student#` substituting # for your student number.

Once you are in this folder, you will see `terraform-aws-cf-install' folder.  Navigate to this folder.
```
cd terraform-aws-cf-install
```

A quick note: we've already downloaded this repo for you but it would have the same results if you were to clone  https://github.com/cloudfoundry-community/terraform-openstack-cf-install

## Add Docker Services

Once in this folder, we will need to make some changes to `terraform.tfvars` so go ahead and open this file with your favorite editor
```
vi terraform.tfvars
```

To the bottom of the file add the following line which will also install the Docker Services
```
install_docker_services = "true"
```

A quick note: The AWS access and secret keys have already been assigned in this file, when you clone the project on your own these will need to be provided.  There is a step-by-step manual for doing this on the DP2 wiki but isn't needed for this demo.

## Verify Terraform

Terraform 0.4.0 has already been installed on the Jumpbox.  To verify this you can run:
```
terraform -v
```

You should get the following output:
```
Terraform v0.4.0
```

For this demo we require v0.4.0, v0.4.2 breaks the install at this time.

The DP2 Wiki has instructions for installing Terraform but here is a short summary:
- Download terraform here: https://dl.bintray.com/mitchellh/terraform/terraform_0.4.0_linux_amd64.zip
- Unzip and place the files somewhere in your $PATH



# Exercise 2 - Deploy Terraform

Now you are ready to deploy, run:
```bash
make clean
make plan
make apply
```

Go get something to drink. It will take about an hour to deploy everything to AWS. Don’t panic if an error occurs during “make apply”, run the command again as AWS resources aren’t always available when requested.

When the installation has completed, your screen should output a series of values you will need to connect to the Cloud Foundry deployment, in our example we see:
```bash
aws_instance.bastion (remote-exec): Deployed 'cf-aws-tiny.yml' to 'bosh-vpc-885f0bed'
aws_instance.bastion: Creation complete

Apply complete! Resources: 9 added, 2 changed, 0 destroyed.

The state of your infrastructure has been saved to the path
below. This state is required to modify and destroy your
infrastructure, so keep it safe. To inspect the complete state
use the 'terraform show' command.

State path: terraform.tfstate

Outputs:

  aws_internet_gateway_id              = igw-23b12546
  aws_key_path                         = ~/.ssh/bosh.pem
  aws_route_table_private_id           = rtb-5d8cad38
  aws_route_table_public_id            = rtb-558cad30
  aws_subnet_bastion                   = subnet-17a6204e
  aws_subnet_bastion_availability_zone = us-west-1a
  aws_vpc_id                           = vpc-885f0bed
  bastion_ip                           = 54.175.138.250
  cf_admin_pass                        = c1oudc0wc1oudc0w
  cf_api                               = api.run.52.0.125.51.xip.io
  cf_domain                            = XIP

```
The three fields which we will need to connect to Cloud Foundry and the Bastion server have been noted above.
# Exercise 3 - Connect to Cloud Foundry

In order to connect to Cloud Foundry the CF CLI has already been installed on the Bastion server:
 
### Connect
Now that the CF CLI tool has been installed on the Bastion server, ssh to the server and connect to Cloud Foundry using the 2 noted values outputted from Step 2:
```bash
cf login --skip-ssl-validation -a api.run.52.0.125.51.xip.io -u admin -p c1oudc0wc1oudc0w
```

Thats it!  You can now write your application locally and then push it to Cloud Foundry.


### Create Service Broker

Execute the following on the bastion server:
```bash
cf create-service-broker docker containers containers http://cf-containers-broker.run.52.0.125.51.xip.io
```

###Note 1
If you get an error indicating “cf” is not installed, run the following:
```bash
curl -s https://raw.githubusercontent.com/cloudfoundry-community/traveling-cf-admin/master/scripts/installer | bash
```

### Note 2
If you forgot to login in Exercise 3 you will get the message ”No API endpoint set. Use 'cf login' or 'cf api' to target an endpoint.”  If so, execute the following:
```bash
cf login --skip-ssl-validation -a api.run.52.0.125.51.xip.io -u admin -p c1oudc0wc1oudc0w
```

Once the service broker is created you can list the services available:
```bash
cf service-access
cf enable-service-access mysql56  # Enables a mysql 5.6 instance
```

# Exercise 4 - Tear Down

Terraform does not yet quite cleanup after itself. You can run make destroy to get quite a few of the resources you created, but you will probably have to manually track down some of the bits and manually remove them. Once you have done so, run make clean to remove the local cache and status files, allowing you to run everything again without errors. Locally run:
```bash
make destroy
make clean
```

Now log back into AWS Console and delete any of the following which may have been left behind:
 - Instances
 - VPC
 - Volumes
 - Elastic IPs
 - Key Pairs 
 
There is a step by step manual for doing the teardown on the DP2 wiki. Don't worry about doing this as we will tear down the AWS accounts at the end of the day.

That’s it!  

# Resources
The primary repository is located here:
https://github.com/cloudfoundry-community/terraform-aws-cf-install

Terraform AWS VPC
https://github.com/cloudfoundry-community/terraform-aws-vpc

Terraform AWS CF Net
https://github.com/cloudfoundry-community/terraform-aws-cf-net

Installing RVM
https://rvm.io/rvm/install

Installing Git
http://git-scm.com/book/en/v2/Getting-Started-Installing-Git

CF CLI Releases
https://github.com/cloudfoundry/cli/releases

Traveling CF CLI Install
https://github.com/cloudfoundry-community/traveling-cf-admin
