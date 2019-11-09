# SaU2 Application Deployment with AWS Cloud Formation

The guide walks you through the process of deploying a production grade instance of SaU2 application software on a AWS account. The deployed application will have:

-   [Single CloudFront distribution](https://aws.amazon.com/blogs/networking-and-content-delivery/dynamic-whole-site-delivery-with-amazon-cloudfront/) for serving both React Client (static content) and REST API (dynamic content)
    -   Scalability and performance
    -   Negate CORS
    -   SSL Termination
    -   Caching
    -   Content Compression
    -   Rate Limiting (DDoS protection)
    -   More...
-   S3 (serverless) hosting of React Client (static content)
-   A custom Route 53 powered domain name, subdomain and SSL certificate for HTTPS
-   High availability and autoscaling of Server API via AWS ECS Fargate (serverless) deployment
-   A Code CI/CD pipeline, using AWS CodePipeline, CodeBuild and CodeDeploy so that you can deploy or redeploy changes to both the SaUClient and SaUServer applications with a simple `git commit` + `git push`

## 1. Create a domain name to host the application

Go to [Route 53](https://console.aws.amazon.com/route53/home?region=us-east-1#DomainListing:) and you'll see a box to "Register a domain".

Once your domain is purchased and processed you'll see it show up on [your list of domains](https://console.aws.amazon.com/route53/home?#DomainListing:):

## 2. Register an SSL certificate for the domain

SSL Certificate(s) are used to support https protocl for site and backing API. If using a wildcard (e.g., \*.submitanupdate.com) for cert you can use the same cert for both site and API, as described in following documentation. If using named cert for site (e.g., mydev.submitanupdate.com) you will need to create a second cert for API (e.g., mydev-api.submitanupdate.com).

The pattern the templates use with stack creation is to create a subdomain using the value provided for the "environment name" and "host name" parameters, so if you passed to the template "sau2-fullstack-cd-aws-cf.yml", the following parameter values -

EnvironmentName="foo"
HostZoneName="bar.com"

The templates would constructe the following subdomain "foo.bar.com"

You would need to create either a single wildcard cert "\*.bar.com" or two certs, one for site "foo.bar.com" and one for API "foo-api.bar.com"

Please besure you under stand the following requirements for [importing into AWS Certificate Manager (ACM)](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate-prerequisites.html), or for [association with a distribution](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html):

-   The certificate must be imported in the US East (N. Virginia) Region.
-   The certificate must be 2048 bits or smaller.
-   The certificate must not be password protected.
-   The certificate must be PEM encoded.

To associate the imported certificate and certificate chain to your CloudFront distribution, you must be sure that they meet these requirements. Or, you can [request a public certificate from ACM](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html) in the US East (N. Virginia) Region to meet the requirements. Then, you can associate the newly requested certificate with your distribution. This information was found in [AWS Support Article](https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-invalid-viewer-certificate/)

Go to [Amazon Certificate Manager](https://console.aws.amazon.com/acm/home) to get a free SSL certificate for the domain. Click the "Request a certificate" button.

Then select a "public certificate" because this is the certificate public web browsers will use for communicating securely with SaU application.

Then add the list of domains that should be covered by the SSL certificate.

> Note: Use of wildcard (e.g., '\*.submitanupdate.com') for cert provides support for any subdomain, requiring you to only create a single cert, once, for both site and API across all environments for hosted zone name (domain).
>
> You may also want to specify the "bare domain" in the same certificate:

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/cert-domain-list.jpg' width='50%' />

This allows for serving HTTPS traffic on https://submitanupdate2.com and https://www.submitanupdate2.com as well as on any other subdomains (e.g., https://dev1.submitanupdate2.com )
ACM can take care of automatically validating the certificate for you if you are also hosting the domain on Route 53. After a few mins I am able to view the details of the created certificate and get the certificate ARN (Amazon Resource Name):

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/ssl-cert-manager-cert-arn.jpg' width='50%' />

Copy that ARN value for usage later.

## 3. Setup Github repo for CI/CD

Go to your [Github settings to generate a new token](https://github.com/settings/tokens). Click on "Generate new token" and create a token which has the following access:

-   `admin:repo_hook`
-   `repo`

These two permissions will allow AWS CodePipeline to monitor the Github repo for changes, and react to updates by redeploying the application.

## 4. Create Secret for JSON Object (String) containing Environment Variables

These instructions are for the fullstack deployment using template
[sau2-fullstack-cd-aws-cf.yml](https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/templates/sau2-fullstack-cd-aws-cf.yml) this template takes a parameter `EnvSecretArn` which is the ARN for a AWS Secret Manager Key whose value is a JSON object String containing key:Value pairs for all required and optional environment vars, including sensitive data.

Further documentation about SaU2Server Environment Variables can be found in [SaU2Server repo README.md](https://github.com/swmcode/SaUServer/blob/develop/README.md)

Go to [AWS Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager/home?region=us-east-1#/listSecrets) for selected/desired region. Click "Store a new secret"

Select "Other type of secrets" for secret type and under plaintext view enter/paste environment var JSON object string

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-secret-manager-env-vars.jpg' width='50%' />

Complete the entry steps to create key (see following screen shot for meaningful suggestion on name, description and tags), then copy ARN for use when running fullstack formation.

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-secret-manager-env-vars-key.jpg' width='50%' />

## 5. Deploy SuA2 Application Fullstack on your account

The following instructions require you to have CF templates available in a S3 bucket. This can be manually performed or managed via CI/CD --see [SaU2DevOps README.md](https://github.com/swmcode/SaUDevOps/blob/master/README.md)

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-s3-bucket-cf-templates.jpg' width='50%' />

Go to [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/home) and click the "Create Stack" button. In the following dialog select "Template is ready" and "Amazon S3 URL" then enter AWS S3 URL for template `sau2-fullstack-cd-aws-cf.yml`

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-fullstack-template-s3-url.jpg' width='50%' />

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack.jpg' width='50%' />

then click "Next". You will see a list of input parameters. Fill them in with the appropriate values, including the ARNs for cert and secret key. Click "Next" for remaining steps, check acknowledgements and run stack creation.

You can watch progress of stack creation and completion --hopefully you don't see any errors (which should result in rollback) .

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack-progress.jpg' width='50%' />

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack-complete.jpg' width='50%' />

## 7. Validate Stack Creation

After successful completion of stack creation you will be able to browse to running application using following pattern https://[EnvironmentName].[HostZoneName] so if you passed the following `EnvironmentName=qa` and `HostZoneName=submitanupdate.com` then your URL would be `https://qa.submitanupdate.com`

You can also validate CI/CD by modifying application in SaUClient and SaUServer repositories' for targeted branches (branch names passed as parameters for stack creation). Do a `git commit` and `git push` to push your changes up to Github. CodePipeline will pick up the changes and rereun the pipeline to rebuild the application and roll out your updates with zero downtime.

## Troubleshooting

When troubleshooting, remember to make sure correct region is selected.

If Stack creation fails, start with reviewing creation events and review any error/failed statuses.

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack-error.jpg' width='50%' />

Besure to review failures and events for individual nested stacks as well as master stack.

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack-error-resources.jpg' width='50%' />

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack-error-resources-events.jpg' width='50%' />

If Stack creation succeeded but application can't be browsed to or your can't log-in or perform other actions you will want to start by reviewing [code pipelines](https://console.aws.amazon.com/codesuite/codepipeline/pipelines?region=us-east-1) for any failures/errors, if none found you will want to look at individual services and their associated logs -

[S3](https://s3.console.aws.amazon.com/s3/home?region=us-east-1) for React Client hosting

[ECS](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters) for Rest API
