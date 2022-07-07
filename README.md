# Continuous Delivery (Continuous Integration & Continuous Deployment) Infrastructure with Continuous Monitoring

[![Nolwac](https://circleci.com/gh/Nolwac/continuous-delivery-infra.svg?style=svg)](https://circleci.com/gh/Nolwac/continuous-delivery-infra)

## Project Overview

This project is a demonstration of the elegance of Continuous Delivery and Continuous Monitoring. The project is an automated deployment of a frontend and backend infrastructure.
**This is the HEART of DevOps**

You can take a look at the *presentation.pdf* file to know why CI/CD and monitoring are very important concepts you can incorperate into your project workflow.

You can as well take a look at the screenshots in the screenshots folder to see what was done.

## Seeing the project in action

To run the project, you have to do the following:
1. Signup on [CircleCi](https://circleci.com)
2. Signup on Amazon Web Services (AWS)
3. Create use the *cloudfront.yml* file at *.circleci/files* to create a stack with name *InitialStack* with CloudFormation.
4. Create a PostgreSQL database on AWS RDS.
5. Create an IAM user with programmatic access on AWS.
7. Fork this repo and create a project on CircleCi with the forked repo.
8. Terminate the initial pipeline workflow triggered when the project is created.
9. Use the information from step 4 and 5 to create the following environment variable on the circlci project.
   * AWS_ACCESS_KEY_ID=(from IAM user with programmatic access)
   * AWS_SECRET_ACCESS_KEY= (from IAM user with programmatic access)
   * AWS_DEFAULT_REGION=(your default region in aws)
   * TYPEORM_CONNECTION=postgres
   * TYPEORM_MIGRATIONS_DIR=./src/migrations
   * TYPEORM_ENTITIES=./src/modules/domain/**/*.entity.ts
   * TYPEORM_MIGRATIONS=./src/migrations/*.ts
   * TYPEORM_HOST={your postgres database hostname in RDS}
   * TYPEORM_PORT=5432 (or the port from RDS if it’s different)
   * TYPEORM_USERNAME={your postgres database username in RDS}
   * TYPEORM_PASSWORD={your postgres database password in RDS}
   * TYPEORM_DATABASE=postgres {or your postgres database name in RDS}
10. Create an EC2 instance (Ubuntu) on AWS following these guides to setup the instance as a Prometheus Server for monitoring.
   * [Setup Prometheus on the EC2 Instance](https://codewizardly.com/prometheus-on-aws-ec2-part1/)
   * [Setup the EC2 instance to allow monitoring of instance on AWS](https://codewizardly.com/prometheus-on-aws-ec2-part3/)
   * [Setup Alert Manager on the EC2 Instance](https://codewizardly.com/prometheus-on-aws-ec2-part4/)

Once you have done all this, go ahead and make any commit on the forked project to see the Pipeline in action.
*An example change you could make would be to edit this README, or to add a line of code that would cause the code to break*

You can observe on your AWS console as infrastructures are being created and destroyed for every commit you make to the master branch of the forked repo.

## Prometheus Monitoring interface.
To see the Monitoring result navigate to *http://<EC2 Instance Public IP or Public DNS Name>:9090* to see the monitoring interface provided by Prometheus. Example is **http://ec2-3-95-154-68.compute-1.amazonaws.com:9090**.
Explore the metrics.

You can try stoping the EC2 instance created by one of the stacks from the cloudformation yml script for some minutes to see Prometheus send you an alert email that your server is down. That's the beauty of Monitoring, but there is a lot more you can do with it.

## How it works
To understand how it works, you need to first of all recognize these services.
* AWS cloudformation: is an Infrastructure as Code (IaC) service offered by AWS that allows you create infrastructure to run almost any kind of software/service without visiting the AWS console. You do that completely with code (YAML or JSON).
* Ansible: allows you to automatically configure servers remotely with code.
* CircleCi: offers a CI/CD pipeline service that can get triggered when commit, pullrequest and other operations are made on your remote repository.
* Prometheus: as earlier mentioned is an open source monitoring software.
* AlertManager: is an open source alert solution for Prometheus.

Haven known about these tools/services, let's dive into how they all come together to give you a completely automated workflow from code to production.

Simple put, Once your codebase (remote repository) is integrated with CircleCi, the following is done on the pipeline:
* Unit testing and integration testing, code linting and dependency vulnerablility checks are done on the codebase.
* *awscli* tool is installed and used to execute the cloudformation script in the project to create the infrastructures for the system.
* Ansible is installed and used to execute the ansible configuration codes (Playbooks) to configure the instances created by the cloudformation script on AWS. This configuration, includes, setting up the environment to run the codebase on and setting up Prometheus *Node-exporter*, a software that monitors the health of the bare metal of the EC2 instance and serves it as endpoint so that prometheus can make calls to the endpoint to collect metrics.
* A smoke test is done to make sure that the newly deployed system is functional and healthy.
* At any point if, any of the deployment processes (jobs) fail, then a rollback process is done to destroy the infrastructure that was created.
* If all the tests/process pass succefully, then the deployment is promoted to be the main production deployment and the previous deployment destroyed. This is what is known as *Blue-Green* deployment strategy.

And that is it, at point that a server is done, the alert manager sends you an alert, how awesome!

As you may have notice, at the topmost level of this docs, just after the title is a CircleCI badge, which displays the current status of the Pipeline build. Here it is again.

[![Nolwac](https://circleci.com/gh/Nolwac/kubernetes-ml-microservice.svg?style=svg)](https://circleci.com/gh/Nolwac/kubernetes-ml-microservice)