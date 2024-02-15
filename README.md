# Deploying CloudFormation via CodePipelines Custom Workshop - BLS

This workshop is a custom workshop designed to walk through the process of building a pipeline to deploy CloudFormation templates in nested stacks and build the infrastructure for a simple web application.  This workshop will walk through the process of creating a CodePipeline Pipeline and adding nested stacks to a master stack to deploy the infrastructure.

Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Requirements

* [Git Installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

## Workshop Instructions

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
    ``` 
    git clone https://github.com/awsjrdwyer/cfn-pipeline-workshop.git
    ```
1. Follow along with instructor for creating and cloning a CodeCommit repository.
1. Copy files from GitHub to CodeCommit and commit changes
   ```
   cp cfn-pipeline-workshop/* cfn-pipeline-workshop-codecommit/
   ```
1. Follow along with instructor to create pipeline and run initial deployment.
1. The application team has now provided their application stack with app-stack.yml and we need to add it to the master stack.  The following will need to be added to the master stack and config files.
   master-stack.yml Parameters:
   ```
     EnvironmentName:
       Description: A custom alphanumeric name for this Environment
       Type: String
       MinLength: 1
       MaxLength: 20
       ConstraintDescription: Must be alphanumeric with no spaces
       AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
     WebInstanceType:
       Description: Instance Type for Web/App Servers
       Type: String
       AllowedValues:
         - "t3.small"
         - "t3.medium"
         - "t3.large"
         - "t3.xlarge"
         - "m5.large"
         - "m5.xlarge"
         - "c5.large"
         - "c5.xlarge"
         - "r5.large"
         - "r5.xlarge"
     DbInstanceType:
       Description: Instance Type for Database Instance
       Type: String
       AllowedValues:
         - "db.t3.small"
         - "db.t3.medium"
         - "db.t3.large"
         - "db.t3.xlarge"
         - "db.m5.large"
         - "db.m5.xlarge"
         - "db.r5.large"
         - "db.r5.xlarge"
     DbVersion:
       Description: MySQL Version for Database Instance
       Type: String
     DbUser:
       Description: Username for Database Instance
       Type: String
     DbPassword:
       Description: Password for Database Instance
       Type: String
       NoEcho: true
       MinLength: 1
       MaxLength: 41
       AllowedPattern: ^[a-zA-Z0-9]*$
     KeyPair:
       Type: "AWS::EC2::KeyPair::KeyName"
     AMIId:
       Type: "AWS::EC2::Image::Id"
   ```
   master-stack.yml Resources:
   ```
     AppStack:
       Type: "AWS::CloudFormation::Stack"
       Properties:
         TemplateURL:
           Fn::Sub: "https://s3.amazonaws.com/${TemplatePath}/app-stack.yml"
         Parameters:
           EnvironmentName:
             Ref: EnvironmentName
           WebInstanceType:
             Ref: WebInstanceType
           DbInstanceType:
             Ref: DbInstanceType
           DbVersion:
             Ref: DbVersion
           DbUser:
             Ref: DbUser
           DbPassword:
             Ref: DbPassword
           VPCId:
             Fn::GetAtt: NetworkStack.Outputs.VPCId
           PublicSubnet1:
             Fn::GetAtt: NetworkStack.Outputs.PublicSubnet1Id
           PublicSubnet2:
             Fn::GetAtt: NetworkStack.Outputs.PublicSubnet2Id
           AppSubnet1:
             Fn::GetAtt: NetworkStack.Outputs.AppSubnet1Id
           AppSubnet2:
             Fn::GetAtt: NetworkStack.Outputs.AppSubnet2Id
           DataSubnet1:
             Fn::GetAtt: NetworkStack.Outputs.DataSubnet1Id
           DataSubnet2:
             Fn::GetAtt: NetworkStack.Outputs.DataSubnet2Id
           KeyPair:
             Ref: KeyPair
           AMIId:
             Ref: AMIId
         Tags:
           - Key: Name
             Value: AppStack
   ```
   master-stack.yml Outputs:
   ```
   Outputs:
     ALBURL:
       Description: "URL endpoint of ALB"
       Value:
         Fn::GetAtt: AppStack.Outputs.ALBURL
   ```
   config.json:
   ```
       "EnvironmentName": "Workshop",
       "WebInstanceType": "t3.small",
       "DbInstanceType": "db.t3.small",
       "DbVersion": "8.0.36",
       "DbUser": "admin",
       "DbPassword": "dbpassword",
       "KeyPair": "demo-keypair",
       "AMIId": "ami-0e731c8a588258d0d"
   ```
1. Follow along with instructor to commit your changes to a branch and create a pull request to merge the app changes to the main branch for deployment.
1. Now that we have the app deployed, the security team has noticed that the password for the database is in plain text in the config.  Let's create a hotfix branch to move the database password to Secrets Manager.
   app-stack.yml Resources:
   ```
     RDSSecrets:
       Type: AWS::SecretsManager::Secret
       Properties:
         Description: 'This is the secret for my RDS instance'
         GenerateSecretString:
           SecretStringTemplate: '{"username": "admin"}'
           GenerateStringKey: 'password'
           PasswordLength: 16
           ExcludeCharacters: '"@/\'
      RDSInstance:
         Properties:
           MasterUsername: !Join ['', ['{{resolve:secretsmanager:', !Ref RDSSecrets, ':SecretString:username}}' ]]
           MasterUserPassword: !Join ['', ['{{resolve:secretsmanager:', !Ref RDSSecrets, ':SecretString:password}}' ]]
   ```
1. Follow along with the instructor to clean up the code and commit to the new hotfix branch.
