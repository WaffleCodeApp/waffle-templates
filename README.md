# Waffle Templates

This repository is a set of infrastructure templates for AWS CloudFormation to help easily kick off high-security, compliant, full-stack web applicaitons.

## The intention of this project


The goal of this project is to enable businesses to start web application development projects with a small team that doesn't have devops engineering experience. The idea is to start with being compliant and audit-ready from the first-line of code using default industry standards and opinionated practices. And only hire devops personnel once the project requires customization to be cost efficient at large-scale.

If you want to use the Waffle templates but do not want to work with Cloudformation code, we now offer an affordable web-hosted version of Waffle that deploys these templates into your AWS account for you, that makes AWS as easy to use as Heroku/Vercel. See [wafflesoc2.com](https://wafflesoc2.com) for more information.

## What are these templates?

There are eight templates that can be used together to set up an infrastructure in AWS that serves the typical requirements of early-stage web application development projects.

While these templates rely on each other, they can be used independently as well. The templates produce output variables that are tracked by AWS CloudFormation. They use each-other's output vairables by default, which enables these tempates to work together and use each-other's resources without further configuration. But if one already has an alternative setup in place, these templates can be set up with custom parameters to use the custom resources instead.

The templates cover the following areas:
- Virtual private networking: private networks in mutliple availability zones with default access controls.
- API gateway: API key and user-authenticated HTTP REST endpoints with firewall, where the backend services can be accessed.
- SQL databases either from low to high performance.
- Github integration for CICD pipelines with secret handling.
- CICD pipelines for deploying frontend web-applications.
- CICD pipelines for deploying containerized backend services.
- CICD pipelines for deploying serverless functions.

The templates follow SOC II requirements:
- Implements least privilege approach for backend services’ communication over the internet.
- Implements least privilege approach secret management.
- Enables infrastructure level change management using infrastructure as code and related logging.
- Enables source code level change management and deployment approvals.
- Automated database backups
- Monitoring and alarms

The industry defaults and opinionated practices cover:
- Secret handling solutions for databases and backend services.
- Private networking security settings.
- API key plans.
- Default monitoring and alarms settings that notify the owners in email in case of suspicious activities.

## Requirements

These templates are designed for AWS only.

The templates only focus on the infrastructure, one can use any programming language for the backend that AWS has support for.

One can use these templates in their own AWS account. AWS charges the owner of the account for using the services. The authors of these templates don't take responsibility for the costs generated by the usage of these templates. It's recommended to set up billing alerts limits and the AWS accounts to prevent unexpected AWS costs.

## Need help setting up your infrastructure?

Let's chat! Happy to help. greg@wafflecode.com

# How to set up the infrastructure

Steps:
- Create an AWS account
- Create DNS settings
- Create generic a HTTPS certificate
- Deploy VPC
- Deploy Authentication
- Deploy the API Gateway
- Deploy Miscellaneous tools
- Setup secrets
- Deploy Databases
- Deploy backend services as serverless
- Deploy backend services as a containers
- Deploy frontends

## Creating an AWS account

For setting up your AWS account, refer to the [specific doc](setting_up_aws.md).

## Creating DNS Settings

Assuming that the root AWS account has the top level domain registered, let's say `example.com`. Then the idea is to have these templates deploy everything under a subdomain of it, like `dev.example.com`.

For this, in the newly created account, go to Route 53, and create a new hosted zone for `dev.example.com`, as a public hosted zone.

This new hosted zone will only be known and used by the public, when it's connected to the root DNS account. There will be an NS type record automatically created for the subdomain (`dev.example.com`). Then go to the AWS root account, Route 53, and select the hosted zone of the root DNS (`example.com`). Select `Create record`. For the subdomain use the new subdomain (`dev`), the record type has to be `NS`. 

## Creating generic a HTTPS certificate

We'll need a certificate that can be used for serving any backends or frontends using HTTPS.

Go to `AWS Certificate Manager` on the AWS Console, and proceed with `Request certificate`. Select requesting a public certificate.

For the domain names, specify the new domain (`dev.example.com`). Add another domain with a wildcard (`*.dev.example.com`). If the DNS was set up using Route 53, then you can choose DNS validation.

To accomplish DNS validation, you can view the details of the certificate, and in the Domains section select `Create records in Route 53`. Validation should be done in less than a minute.

The ARN of the newly created certificate will be used later. 

## Deployment ID

After this point a couple of templates are going to be deployed. They have references to each-other. In order for them to find each other, an identifier is used in all of them. It's the `DeploymentID` parameter in every template.

Choose an id for your deployment, and use that in all cases below. For example it can be `dev`.

## Deployment Type

Two different types of deployments are supported:
- `PROD` adds more monitoring features
- `DEV` is cheaper

Some of the stack require this setting.

## Deploying a VPC

Deploying the following template sets up a private network with private and public subnets, with the required routing tables, internet gateways, NAT settings and a set of security groups intended to be used for generic purposes.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `vpc-template.yml` from this repo.

Use the following parameters:
- Stack name: `vpc` (can be whatever)
- DeploymentId: `dev` (see the Deployment ID section above)
- DeploymentType: `DEV` (or `PROD`, see the Deployment Type section above)
- PrimaryPrivateCidr: `10.51.0.0/19` (feel free to use different CIDR settings)
- PrimaryPublicCidr: `10.51.128.0/20`
- SecondaryPrivateCidr: `10.51.32.0/19`
- SecondaryPublicCidr: `10.51.144.0/20`
- VPCCidr: `10.51.0.0/16`

It's OK to use the default stack option configuration.
Before submission, acknowledge that the templates are going to create IAM roles.

Expected to take 3-4 minutes to deploy.

## Deploying Authentication

Deploying the related template sets up AWS Cognito.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `authentication-template.yml` from this repo.

Use the following parameters:
- Stack name: `authentication` (can be whatever)
- AuthUserAuthenticationSMSText: `Hi {username}, your verification code is {####}`
- AuthUserInvitationSMSText: `Hi {username}, your invitation code is {####}`
- AuthUserVerificationSMSText: `Hi {username}, your verification code is {####}`
- DeploymentId: `dev` (see the Deployment ID section above)
- DeploymentType: `DEV` (or `PROD`, see the Deployment Type section above)

It's OK to use the default stack option configuration.
Before submission, acknowledge that the templates are going to create IAM roles.

Expected to take 2-3 minutes to deploy.

## Deploying the API Gateway

Deploying the next template sets up an API Gateway. The idea is that backend services with public endpoints register resources on this API Gateway. The gateway uses IAM based authentication, assuming that the connecting frontends are using AWS Cognito from the above template.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `authentication-template.yml` from this repo.

Use the following parameters:
- Stack name: `api` (can be whatever)
- BackendApiHostname: `api` (use anything you want, this will be the domain name of the api gateway under your subdomain)
- DeploymentId: `dev` (see the Deployment ID section above)
- FullDomainName: the domain created in the new AWS account (`dev.example.com`)
- GenericCertificateArn: the ARN of the certificate created above

It's OK to use the default stack option configuration.
Before submission, acknowledge that the templates are going to create IAM roles.

Expected to take 3-4 minutes to deploy.

## Deploying Miscellaneous tools

This template adds the following things:
- An AWS SNS topic that can be used from any backend services to send system-level alerts and notifications. The SNS messages are not delivered anywhere by these templates, but the idea is that a simple lambda function can take them and send them over to email, Slack or whatever.
- A generic secret for the deployment. It's supposed to store things like credentials for 3rd party services used by backend services, like for example Slack webhook settings for delivering the alerts. This secret will be accessible by any backend component.
- A secret specificly for GitHub credentials. It's used by CICD pipelines to get notified about changes and pull sources.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `misc-template.yml` from this repo.

Use the following parameters:
- Stack name: `miscellaneous` (can be whatever)
- DeploymentId: `dev` (see the Deployment ID section above)
- EmailNotificationList: provide an email address, or a comma-separated list, where CloudWatch alarms are sent (these alarms are not sent to the alerts SNS, as those are not delivered if there's no lambda to distribute them.)

It's OK to use the default stack option configuration.

Expected to take up to 1 minute to deploy.

Wohoo, at this point we're ready to deploy the actual app!

## Setting up secrets

In order to be able to deploy things from GitHub, credentials for GitHub has to be provided.

Go to `AWS Secrets Manager > Secrets` and select the one of the name starting with `GitHubSecret-`. The `token`` key-value pair is supposed be there, but with the value empty.

Get an access token from GitHub: `GitHub > Settings > Developer Settings > Personal Access Tokens > Tokens (classic)`, and generate one with enabled all `repo` and `admin:repo_hook` permissions. Copy the token value to the secret as the value of the key `token`.

If you're planning to use 3rd party services, like Slack for delivering alerts, then you can put them into the secret with the name starting with `Secret-`.

The ARN of this secret is shared with each backend service as an environmental variable called `SERVICE_SECRET_ARN`.

## Deploying Databases

This template deploys a PostgreSQL database. It can be either an RDS or a high performance Aurora.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `db-template.yml` from this repo.

Use the following parameters:
- Stack name: can be whatever, something that explains that this is a db and its purpose too
- AllocatedStorageSize: size of the DB in GB
- CreateReplica: specifying 'Yes' will create a read-only replica if the db is RDS (see DBType below).
- DBType: `rds` or `aurora`. Aurora is more robust, and has features for higher availability.
- DatabaseId: the name specified here will be used in backend services for accessing this db. Recommended to use a name that clearly identifies the db's purpose.
- DeploymentId: `dev` (see the Deployment ID section above)
- Family: typcally `postgres15` or `aurora-postgresql15`, has to be aligned with the DBType setting
- InstanceClass: the minimum is `db.t3.micro` for RDS and `db.t3.medium` for Aurora
- PostgresEngineVersion: feel free to use whatever exist and supported
- SnapshotId: optional: to be specified if you already have a db snapshot that you want to use as the initial db content

The optional parameters can be used to override values coming from the existing stacks.

Time to deploy depends a lot on the db type, it it's created from a snapshot and if there are replicas. Expected to take 15-60 minutes.

### Accessing DBs from backends

Databases have their master credentials auto-generated. Each database has its own secret that stores their master credentials.

The names of the secrets storing the application load balancers' host names are the following respectively:
`{DeploymentID}/db/{DatabaseID}/secretName`, where:
- DeploymentID is the the ID of this deployment in question
- DatabaseID is the ID of the database stack.

## Deploying backend services as serverless

This template deploys a CICD pipeline that can deploy a complete CloudFormation stack. The primary purpose is to have lambda function(s) defined in the stack template so that the CICD can deploy into AWS Lambda. But if needed other AWS resources can be added too, like CloudWatch Alarms, DBs, Queues, etc. More on the requirements on the deployed repo below.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `cfn-template.yml` from this repo.

Use the following parameters:
- Stack name: can be whatever. Recommended to have something that explains that this is a cicd deploying cloudformation and its purpose too (for example: `cicd-cfn-alerts`)
- BuildspecPath: The path to the `buildspec.yml` file. (like `alerts/buildspec.yml`)
- CICDManualApproval: If set to True, then there will be a manual step in the CICD process, that requires a human to approve the changes. Intended to be used on production environments.
- DeploymentId: `dev` (see the Deployment ID section above)
- DeploymentType: `DEV` (or `PROD`, see the Deployment Type section above)
- GithubBranch: the branch of the github repo to be deployed
- GithubOwner: the user or organization that the repository belongs to
- GithubRepoName: the name of the repository
- PipelineId: can be whatever. Recommended to use a name that can be used from code to access this service (for example? `alerts`)

The optional parameters can be used to override values coming from the existing stacks.

It's OK to use the default stack option configuration.
Before submission, acknowledge that the templates are going to create IAM roles.

Expected to take up to 2-3 minutes to deploy.

### CFN repository requirements

There are two required files to be included in a repo that's meant to be deployed as serverless.
- `cnf-template.yml` - this is a CloudFormation template that's meant to be deployed.
- `buildspec.yml` - this tells `AWS CodeBuild` what to do with the source files before deployment. In this specific case, it has to prepare a cfn template ready to be deployed with its parameters. 

### Using the CICD pipeline

You can find the CICD pipeline in `AWS Console > CodePipeline > Pipelines >` the name will be something similar to the stack name specified above.

Look for building issues in the related CodeBuild logs.

When the CICD processes the initial repo or a change in the repo successfully, then it creates a new or updated CloudFormation stack. The deployment of this stack can be tracked in `AWS Console > CloudFormation`. The name of the newly generated stack will be `{DeploymentID}-{PipelineID}`.

A deployment can take a longer time, depending on the build complexity and the number and types of AWS resources to be deployed or updated.

## Deploying backend services as a container

This template deploys a CICD pipeline that can deploy a docker container. After the CICD builds the contaner image, it is stored in the `AWS ECR` container repository. And new containers are deployed by `AWS CodeDeploy` as `AWS ECS Fargate` Clusters > Services > Tasks.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `ecs-template.yml` from this repo.

Use the following parameters:
- Stack name: can be whatever. Recommended to have something that explains that this is a cicd deploying cloudformation and its purpose too (for example: `cicd-ecs-engine`)
- BuildspecPath: The path to the `buildspec.yml` file. (like `engine/buildspec.yml`)
- CICDManualApproval: If set to True, then there will be a manual step in the CICD process, that requires a human to approve the changes. Intended to be used on production environments.
- DeploymentId: `dev` (see the Deployment ID section above)
- DeploymentType: `DEV` (or `PROD`, see the Deployment Type section above)
- GithubBranch: the branch of the github repo to be deployed
- GithubOwner: the user or organization that the repository belongs to
- GithubRepoName: the name of the repository
- InstanceCount: how many instances should be run in parallel. If you're unsure if your container builds without issues, it's helpful to start with 0, and increase the parameter later.
- PipelineId: can be whatever. Recommended to use a name that can be used from code to access this service (for example? `engine`)

The optional parameters can be used to override values coming from the existing stacks.

It's OK to use the default stack option configuration.
Before submission, acknowledge that the templates are going to create IAM roles.

The time required to deploy this stack depends on the number if instances to be deployed, as the deployment waits for the instances to be created. If it's 0 then it's about 2-3 minutes to deploy this stack.

### Accessing ECS backends from Lambdas

Backend services deployed to AWS ECS Fargate have application load-balancers. These load balancers' host names are stored in the AWS SSM parameter store so that lambdas (or other services) can look them up easily.

The names of the parameters storing the application load balancers' host names are the following respectively:
`{DeploymentID}/ecs/{PipelineID}/albDnsName`, where:
- DeploymentID is the the ID of this deployment in question
- PipelineID is the ID of the CICD stack that deploys to ECS Fargate.

## Deploying frontends

This template deploys a CICD pipeline that can deploy static files into an AWS S3 bucket. It also sets up an AWS CloudFront distribution for serving websites that uses the S3 bucket as its file storage.

Go to `AWS Console > CloudFormation` and select `Create stack`. Select that the template is ready, and choose `Upload a template file`. Select `cdn-template.yml` from this repo.

Use the following parameters:
- Stack name: can be whatever. Recommended to have something that explains that this is a cicd deploying cloudformation and its purpose too (for example: `cicd-cdn-client-ui`)
- BuildspecPath: The path to the `buildspec.yml` file. (like `client-ui/buildspec.yml`)
- CICDManualApproval: if deploying to production, select True, else False
- DeploymentId: `dev`, `staging` or `prod`
- DeploymentType: `DEV` or `PROD`
- FullDomainName: the domain created in the new AWS account (`dev.example.com`)
- GenericCertificateArn: the ARN of the certificate created above
- GithubBranch: the branch of the github repo to be deployed
- GithubOwner: the user or organization that the repository belongs to
- GithubRepoName: the name of the repository
- PipelineId: can be whatever. Recommended to use a name that can be used from code to access this service (for example? `client-ui`)
- WebHostname: `www` by default (for `www.dev.example.com`), but can be whatever

The optional parameters can be used to override values coming from the existing stacks.

It's OK to use the default stack option configuration.
Before submission, acknowledge that the templates are going to create IAM roles.

The time required to deploy the cicd and related infrastrucre, is expected to take 5-ish minutes. To run the CICD and wait for the CDN distribution, depends on the buildtime of the frontend.
