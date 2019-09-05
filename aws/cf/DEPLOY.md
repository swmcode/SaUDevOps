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

Go to [Route 53](https://console.aws.amazon.com/route53/home) and you'll see a box to "Register a domain". Go ahead and purchase a domain name of your choice.

<img src='' width='50%' />

Once your domain is purchased and processed you'll see it show up on [your list of domains](https://console.aws.amazon.com/route53/home?#DomainListing:):

<img src='' width='50%' />

## 2. Register an SSL certificate for the domain

Now go to [Amazon Certificate Manager](https://console.aws.amazon.com/acm/home) to get a free SSL certificate for the domain. Click the "Request a certificate" button.

<img src='' width='50%' />

Then we need to select that we want a "public certificate" because this is a certificate public web browsers will use for communicating securely with the chat app.

<img src='' width='50%' />

Then add the list of domains that should be covered by the SSL certificate. For maximum flexibility I prefer to have both the bare domain and a wildcard domain in the same certificate:

<img src='https://github.com/swmcode/SaUDevOps/blob/master/aws/cf/docs/images/cert-domain-list.jpg' width='50%' />

This allows me to serve HTTPS traffic on https://submitanupdate2.com as well as on any subdomains if for example I want to have https://dev1.submitanupdate2.com

ACM can take care of automatically validating the certificate for you if you are also hosting the domain on Route 53. After a few mins I am able to view the details of the created certificate and get the certificate ARN (Amazon Resource Name):

<img src='' width='50%' />

Copy that ARN value for usage later.

## 3. Setup Github repo for CI/CD

Go to your [Github settings to generate a new token](https://github.com/settings/tokens). Click on "Generate new token" and create a token which has the following access:

-   `admin:repo_hook`
-   `repo`

These two permissions will allow AWS CodePipeline to monitor the Github repo for changes, and react to updates by redeploying the application.

## 4. Deploy SuA2 Application Fullstack on your account

Go to [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/home) and click the "Create Stack" button. In the following dialog click "Specify template" and select the `sau2-fullstack-cd-aws-cf.yml` file you located, then click "Next". You will see a list of input parameters. Fill them in with the appropriate values:
