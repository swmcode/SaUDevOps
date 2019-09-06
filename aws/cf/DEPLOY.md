# SaU2 Application Deployment with AWS Cloud Formation

The guide walks you through the process of deploying a production grade instance of SaU2 application software on a AWS account. The deployed application will have:

-   [Single CloudFront distribution](https://aws.amazon.com/blogs/networking-and-content-delivery/dynamic-whole-site-delivery-with-amazon-cloudfront/) for hosting/servering both React Client (static content) and REST API (dynamic content)
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

Go to [Route 53](https://console.aws.amazon.com/route53/home?region=us-east-1#DomainListing:) and you'll see a box to "Register a domain". Go ahead and purchase a domain name of your choice.

Once your domain is purchased and processed you'll see it show up on [your list of domains](https://console.aws.amazon.com/route53/home?#DomainListing:):

## 2. Register an SSL certificate for the domain

Now go to [Amazon Certificate Manager](https://console.aws.amazon.com/acm/home) to get a free SSL certificate for the domain. Click the "Request a certificate" button.

Then we need to select that we want a "public certificate" because this is a certificate public web browsers will use for communicating securely with the chat app.

Then add the list of domains that should be covered by the SSL certificate. For maximum flexibility I prefer to have both the bare domain and a wildcard domain in the same certificate:

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/cert-domain-list.jpg' width='50%' />

This allows for serving HTTPS traffic on https://submitanupdate2.com and https://www.submitanupdate2.comas as well as on any other subdomains (e.g., https://dev1.submitanupdate2.com )
ACM can take care of automatically validating the certificate for you if you are also hosting the domain on Route 53. After a few mins I am able to view the details of the created certificate and get the certificate ARN (Amazon Resource Name):

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/ssl-cert-manager-cert-arn.jpg' width='50%' />

Copy that ARN value for usage later.

## 3. Setup Github repo for CI/CD

Go to your [Github settings to generate a new token](https://github.com/settings/tokens). Click on "Generate new token" and create a token which has the following access:

-   `admin:repo_hook`
-   `repo`

These two permissions will allow AWS CodePipeline to monitor the Github repo for changes, and react to updates by redeploying the application.

## 4. Create Secert for Environment Var JSON Object (String)

These instructions are for the fullstack deployment using template
[sau2-fullstack-cd-aws-cf.yml](https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/templates/sau2-fullstack-cd-aws-cf.yml) these templates takes a parameter `EnvSecretArn` which is the ARN for a AWS Secret Manager Key whose value is a JSON object String containing key:Value pairs for all required and optional environment vars, including sensitive data.

Futher documentation about SaU2Server Environment Variables can be found in [SaU2Server repo README.md](https://github.com/swmcode/SaUServer/blob/develop/README.md)

Got to [AWS Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager/home?region=us-east-1#/listSecrets) for selected/desired region. Click "Store a new secret"

Select "Other type of secrets" for secret type and under plaintext view enter/paste environment var JSON object string

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-secret-manager-env-vars.jpg' width='50%' />

Complete the entry steps to create key (see following screen shot for meaningful suggestion on name, description and tags), then copy ARN for use when running fullstack formation.

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-secret-manager-env-vars-key.jpg' width='50%' />

## 5. Deploy SuA2 Application Fullstack on your account

The following instructions require you to have CF templates available in a S3 bucket. This can be manually performed or mananged via CI/CD --see [SaU2DevOps README.md](https://github.com/swmcode/SaUDevOps/blob/master/README.md)

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-s3-bucket-cf-templates.jpg' width='50%' />

Go to [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/home) and click the "Create Stack" button. In the following dialog select "Template is ready" and "Amazon S3 URL" then enter AWS S3 URL for template `sau2-fullstack-cd-aws-cf.yml`

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-fullstack-template-s3-url.jpg' width='50%' />

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack.jpg' width='50%' />

then click "Next". You will see a list of input parameters. Fill them in with the appropriate values, including the ARNs for cert and secret key. Click "Next" for remaining steps, check acknowledgements and run stack creation. You Can watch progress of stack creation, watch for errors (should result in rollback) and completion.

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack-progress.jpg' width='50%' />

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/aws-cf-createstack-complete.jpg' width='50%' />

## 7. Validate CI/CD Process

Feel free to modify your local copy of the application in SaUClient and SaUServer repositories' targeted branches. Do a `git commit` and `git push` to push your changes up to Github. CodePipeline will pick up the changes and rereun the pipeline to rebuild the application and roll out your updates with zero downtime.