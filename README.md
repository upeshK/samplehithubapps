# ebasa-mule-lanmaster-updateorderstatus

# Deployment Instructions

## Pre-requisites

1. Oracle JAVA7
2. maven 3
3. git (with ssh keypair installed to github and home folder)
4. python-pip
5. AWS-EB-CLI (http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)
6. AWS-CLI (http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

## Section 1 - Building the Artifact

1. Checkout the module from github using; `git@github.com:ebuilderCCO/ebasa-mule-lanmaster-updateorderstatus.git`
2. Using the maven commands you can build the artifact. 
3. From project home folder (ebasa-mule-lanmaster-updateorderstatus) type the following command.<br/>
   `mvn clean install -P [profle_name] -Denv=[profile]`

   Sample command for test profile:<br/> 
      `mvn clean install –P build-test –Denv=test`

   Valid profile names are `[build-dev, build-test, build-prod]` and valid profiles are `[dev, test, prod]`. This command will place a copy of the built artifact in the container directory. 

4. This module depends on 4 other eBuilder core modules.  
commons-int-muleexceptions  
commons-mule-file-archiver

Hence if you do not have the above jars in your local maven repository ({USER_HOME}/.m2 ) it is highly likely that you will get the following error, during the build.
[ERROR] Failed to execute goal on project ebasa-mule-lanmaster-updateorderstatus: Could not resolve dependencies for project com.ebuilder:ebasa-mule-l
anmaster-updateorderstatus:mule:1.1.2-SNAPSHOT: Failed to collect dependencies at com.ebuilder:commons-int-muleexceptions:jar:1.0.0: Failed to read ar
tifact descriptor for com.ebuilder:commons-int-muleexceptions:jar:1.0.0: Could not transfer artifact com.ebuilder:commons-int-muleexceptions:pom:1.0.0
from/to s3-mule-commons-jars-repo (s3://mule-commons-jars-repo): Unable to load AWS credentials from any provider in the chain <br/>

It's using a s3 bucket named 'mule-commons-jars-repo' as maven repository. So you need to configure your maven settings.xml file `{USER_HOE}/.m2/settings.xml` to download above 4 jars from s3 bucket. Add a new server configuration as below with particular AWS Credentials.

``
<server><br/>
<id>s3-mule-commons-jars-repo</id><br/>
<username>{aws access key}</username><br/>
<password>{aws secret key}</password><br/>
</server><br/>	
``

## Section 2 – Configuration changes

## Deployment Configuration

• `aws-deployment.properties` file in `$ebasa-mule-lanmaster-updateorderstatus/configuration/deploy/{env}` In this file, you will find the app-version, name, key-pair, deployment profile to be used, and tags

• `Dockerrun.aws.json` file in `$ebasa-mule-lanmaster-updateorderstatus/configuration/deploy/{env}` Docker base image, ports and authentication information can be found in this file. This file is almost static, and will be notified if there are any changes prior to the deployment.

• `options.config file` in `$ebasa-mule-lanmaster-updateorderstatus/configuration/deploy/{env}` This bundles the deployment information/ environment information towards AWS. 

• Create a standrd AWS `credentials` file in `$ebasa-mule-lanmaster-updateorderstatus/configuration/deploy/{env}`
The content of the credential file is given below 

[default]<br/>
aws_access_key_id =xxx<br/>
aws_secret_access_key =xxx<br/>
region=eu-west-1<br/>

[test]<br/>
aws_access_key_id =xxx<br/>
aws_secret_access_key =xxx<br/>
region=eu-west-1<br/>

• The above created `credentials` file should be available in `/root/.aws' as well`.

## Section 3 - Uploading into AWS EBS

1. After successfully completing section 1 you will end up with a zip file ebasa-mule-lanmaster-updateorderstatus-{version}.zip in ebasa-mule-lanmaster-updateorderstatus/container folder

2. Once you have the ebasa-mule-lanmaster-updateorderstatus-{version}.zip file from the maven build as in Section 1 and Section 2,Traverse to the container folder and execute eb-tool.sh to deploy the artifact in to AWS EBS.type the following command to execute eb-tool.sh <br/>
 `sh eb-tool.sh deploy|s-deploy|create auto [env]`

   Sample command for test profile:<br/> 
    `sh eb-tool.sh create auto test`

Switches
* deploy - deploy a new version to an existing application(Use this command for first time deployment)
* s-deploy - creates a new application if not existing & deploy
* create - create a new application

Valid environments are
* dev
* test
* prod
