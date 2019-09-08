# SaUDevOps

DevOps code, templates and config for infrastructure management of Submit an Update 2 (SaU2)

## AWS Application Deployment

We attempted to use AWS cloud native (serverless) infastructure for SaU application stack (client and server). The infastructure is being managed as code (IaC) via AWS CloudFormation templates.

As of this writing, 09/07/2019, the backing store, MongoDB, is not provisioned, but is a seperate third party managed service.

A robust approach has been taken with the individual SaUClient and SaUServer repos, with both having their own CI/CD CodePipelines for integrating with source repo (GitHub), automating build and deploying when source changes are pushed to source repo.

A master template is used to create the entire SaU application stack, this master template calls the other templates which create base resources (accounts, network, S3 bucket, other...), client and server CI/CD CodePipelines, CloudFormation Distribution and DNS alias records.

The
