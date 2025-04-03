# AWS-Resources-Duplication-Using-Terraform
AWS Account Resources Duplication Using Terraform with Non-Persistent State

There are different approaches to replicate resources from one AWS account to another account.

We need to replicate both infrastructure/configuration as well as data and there are different ways to achieve it.

First, we will see how we can replicate infra/config with non-persistent state.

**Using Import:
**
We use "terraform import" for importing existing infrastructure resources under terraform. In this approach terraform can import all the resources into the terraform state.

For example, if we have 10 ec2 instances, 1 vpc and 2 subnets we must write 13 different import       statements one for each resource to import these resources into terraform state using "terraform import".

To import configuration, we can use the "import block" to import existing infrastructure resources into Terraform.

Code:
   import CLI example:
   terraform import aws_instance.firstinstance i-0ecfs4162gamr2753 --> this will import ec2 instance with the       id "i-0ecfs4162gamr2753" to the terraform state.

   import block example:

   import {
    to = aws_instance.firstinstance
    id = "i-abcd1234"
   }

   resource "aws_instance" "firstinstance" {
    name = "hashi"
    # (other resource arguments...)
  }

Once we have state and configuration in the source account we need to modify the provider for each resources we imported to apply it to the target account.

  resource "aws_instance" "firstinstance" {
  provider = aws.secondinstance
  ami           = "ami-0ecfs4162gamr2753"
  instance_type = "t2.micro"
}

 Then we will change all the configuration related to target account like account number, vpc name, I am role etc. Once changes are done we will apply to the target account.

**Using Former2 :
**
Former2 allows us to generate Infrastructure-as-Code outputs from our existing resources within our AWS account. We can generate terraform code from Former2 using either CLI or web browser.

Using Web Browser Steps:

        a) First go to Former2 website and we need to connect to our AWS account.
        b) During setup add AWS access key id and secret key id then scan our account, it will scan as per entered credentials.
        c) After all the setup, we will be able to see a list of all the AWS resources. We can choose which resources we want to include in our Terraform configuration like EC2, RDS, Lambda function, dynamodb etc.
        d) After selecting the resources we want to copy from one account to another, we can choose the output format as Terraform.
        e) Former2 will then generate the necessary Terraform code.

       EC2 Instance Example:
                      Suppose you have an EC2 instance in your AWS environment. The Terraform code that Former2    generates might look like this:

                       resource "aws_instance" "my_ec2_instance" {
                         ami           = "ami-2f67f774jvtiwu4j"  
                         instance_type = "t2.micro"
                         key_name      = "serlytoken"
 
                         tags = {
                           Name = "MyFirstAccount"
                         }
                       } 

          f) Once we have our Terraform configuration file do the following steps:
                i) Save the code to .tf file
                ii) Change values specific to target account, for example in above example we need to change the ec2 instance's ami, key name and tags name. (needs to be done for all resources).
                iii) Run terraform init
                iv) terraform plan
                v) terraform apply
               
  Using CLI to generate the terraform code:

a) Install former2 in CLI:

      npm install -g former2

b) To Authenticate: 
      aws configure

c) Generate the Terraform configuration

      former2 generate --profile <your-aws-profile> --region <your-region> --output terraform

d) Perform above steps.



**Using Cloudendure: (Acquired by AWS)
**
We also can use cloudendure to copy resource from one account to another. 

   Steps:
    a) We need to install and configure CloudEndure in both the source and destination AWS accounts.

    b) Create a New Project in CloudEndure and select "AWS" environment for both the source and destination accounts.
    c) On the source AWS account, we need to install the CloudEndure agent on the instances that we want to replicate. This can be done via the CloudEndure console where it will guide us through the installation.
    d) Grant all necessary permission.
    e) We need to set up networking set up like vpc in destination accounts along with IAM.
    f) We start the copying of resources.



**Using Sharing of Resource:
**
We can use AWS Resource Access Manager (RAM) for Sharing Resources.

   Steps:
   a) In the AWS RAM Console (in source account) create a resource share.
   b) Choose the resource and confirms.
   c) Create IAM roles and policies in target account so that it can access source account.
   d) In the target account,  accept the resource share request.

I also looked at AWS Migration Hub, AWS Server Migration Service but they are not suitable for the account level replication.
