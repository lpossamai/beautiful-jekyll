---
layout: post
title: From bare metal to AWS Cloud in 90 minutes.
---

The job management platform ran a collection of bare metal servers in a leased data centre. Over the last seven years, the application requirements had outgrown the capacity of the hosting platform. The company decided to migrate the application to Amazon Web Services (AWS) rather than invest in additional bare metal hardware.

## The legacy environment

The job management platform ran on a collection of bare metal servers in a leased data centre. Over the last seven years, the application requirements had outgrown the capacity of the hosting platform. The company decided to migrate the application to Amazon Web Services (AWS) rather than invest in additional bare metal hardware. The staff who originally set up the environment were no longer working for the company. A full-time team of three developers, as well as a sysadmin and DBA, were maintaining the legacy platform.

While the PHP code was git repository controlled, the actual sysadmin configuration was hand edited over several years; items such as Apache configuration, log file management and shared libraries. For the web and API tiers, there was a server running NGINX acting as a load balancer, distributing traffic on a round robin basis to two back-end web servers, each of which was attached to an NFS mounted disk on which the PHP code was installed.

PostgreSQL 9.2 was the database tier along with Redis for credential storage. PostgreSQL was 4TB and growing. Disk I/O was 100% on a normal day. The original designers had chosen to store media files such as images and video in the database and had created a virtual file system mapped in relational database tables within PostgreSQL. The database tier simply would not scale.



## The challenges:

1. With over 60,000 active international users (and growing) the system was in use around the clock and the migration required minimal downtime.

2. Perhaps the largest problem was the old version of PostgreSQL and its +4TB of active data and growing. AWS Database Migration Service was a clear choice. It is a web service you can use to migrate data from your database that is on-premises, onto an Amazon Relational Database Service (Amazon RDS) DB instance in real time. This was not available to us, as it requires a PostgreSQL database that is version 9.3.x or later. We opted to use native PostgreSQL tools to create a live streaming replica from the legacy data centre to PostgreSQL on EC2. We leveraged the flexibility of EC2 to create three tiers of cascading PostgreSQL slave databases such that when we performed the actual cutover we promoted the primary replica to become the master and the two tiers of remaining slaves became the read replica and the failover databases. Using AWS snapshots and CloudFormation we can continue to build chains of cascading slaves in the event of a failure in any node, giving us an HA database solution in AWS. Due to the legacy 100% I/O issues, we provisioned the PostgreSQL EBS volumes using 10,000 IOPS and r4.4xlarge EBS–optimized instances.

3. The API and web tiers depended on an NFS mounted volume to share PHP code and sessions. We re-factored the code across 6 git repositories on a migration branch. The new code stored sessions in Redis and managed file uploads via S3 using an AWS SDK. Using immutable infrastructure, we were able to deploy a built AWS ami to an auto-scaling group of m4.xlarge application servers, fronted with an AWS classic Load Balancer. We chose CloudFormation wrapped in a parameter driven script framework to build new amis reflecting the git commit being deployed. Part of this infrastructure as code delivered a new sysadmin environment using Ubuntu Xenial and Apache HTTP Server 2.4. This resolved the issue of the legacy sysadmin environment being unknown and hand-coded over many years.

4. For Orchestration, we chose Bamboo, which is a continuous integration and deployment server from Atlassian, the makers of JIRA and Confluence which were already in use by the company. We used Bamboo as a task runner/task logger to execute our parameter driven script framework which we installed on a deployment server, also built via the same CloudFormation framework.

5. The company had an existing skilled team with no cloud experience. AWS provided training credits and the AWS consulting partner provided skills transfer to ensure that the existing team were able to manage the migrated solution to deploy to development, UAT and production environments for each application tier.

## Project results:

The main migration was completed with 90 minutes of downtime which had been communicated to the users well in advance. The migration window was over the low traffic Christmas / New Year period.

From the beginning, the job management platform ran well on AWS. For the first week or so the team was able to tune various AWS settings to achieve optimal performance.

There were problems. It was impossible to test the production workload in spite of having a full copy of the production database for extensive pre-migration testing. As issues surfaced in the first hours and days, the developers and DBA were able to make and test changes quickly and deploy those changes using the Bamboo driven pipeline. We encountered no sysadmin specific issues.

Architecture can be found below:
![aws architecture](/assets/img/aws-architecture.png)

**Factors Critical to Project Success**
The critical success factors were obvious in hindsight and key learnings for any company attempting a migration project of this size. A deficit in any of the following may well have lead to an orphan project.

**Senior Company Executive Backing & Strong CTO Leadership**
The company CTO directed the project and provided strong leadership to ensure each participant was aligned with the same goals. He was able to communicate effectively to the board as well as dive into the detail by asking the right questions and offering the right advice at each step. He also played a vital role in communicating the company vision to AWS to achieve maximum vendor support.

**Strong AWS Consulting Partner Support**
The AWS Consulting Partner leads the project, using a technology stack which has undergone the test of time; there were no surprises.

**Skilled and Dedicated Developers and DBA**
The company employees were skilled and professional and they knew their application code well. With zero previous AWS experience, they received a one-day AWS Essentials training course and in-project training by the AWS Consulting Partner. They learned quickly and were motivated put in their best efforts. Post-migration they are successfully managing the AWS environments.

**Significant Support from AWS**
AWS was engaged from the beginning to help the customer directly and in partnership with the AWS Consulting Partner. The first AWS leadership principle is customer obsession. They start with the customer and work backwards. They work vigorously to earn and keep customer trust.
