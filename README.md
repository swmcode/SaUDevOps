# SaUDevOps

DevOps code, templates and config for infrastructure management of Submit an Update 2 (SaU2)

## AWS Application Deployment

We aimed to use AWS cloud native (serverless) infrastructure for SaU application stack (client and server). The infrastructure is being managed as code (IaC) via AWS CloudFormation templates.

As of this writing, 09/07/2019, the backing store, MongoDB, is not provisioned, but is a separate third party managed service.

A robust approach has been taken with the individual SaUClient and SaUServer repos, with both having their own CI/CD CodePipelines for integrating with source repo (GitHub), automating build and deploying when source changes are pushed to source repo.

A master template is used to create the entire SaU application stack, this master template calls the other templates which create base resources (accounts, network, S3 bucket, other...), client and server CI/CD CodePipelines, CloudFront Distribution and DNS alias records.

The master template is parameterized allowing for domain, subdomain (environment), github info (including target branch), environment vars (as AWS Managed Secret), SSL Cert and S3 Bucket URI (where CF templates can be found) to be passed when setting-up stack creation. Use of parameters, nested templates and CI/CD infrastructure allows for whole application environments to be created and managed with CI/CD, with this you can do the following:

-   Create Production environment (www.submitanupdate.com) targeting GitHub Master branches for SaUClient and SaUServer, so as code is merged to Master branch it is deployed to production environment.

-   Create QA environment (qa.submitanupdate.com) targeting GitHub develop branches for SaUClient and SaUServer, so as code is merged to develop branch it is deployed to qa environment.

-   Environments can be created as desired/needed for development (e.g., myfeature.submitanupdate.com) and target development branches on SaUClient and SaUServer repos --this process of creating environments for dev could be automated.

As of 09/07/2019, tickets exists for building out automated integration and functional tests which could then be added to CodePipelines and possibly Stack Creation for (smoke) testing and validating application deployments/updates.

### CloudFormation (CF) Template CI

A CodePipeline should/could be built out using CF that will integrate with GitHub using Webhook, and would pull updated CF templates on GitHub Push and execute fullstack template and then run integration and functional tests and then delete/remove stack.

As of 09/07/2019 there is a ticket for this work.
