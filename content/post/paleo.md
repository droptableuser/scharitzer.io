---
title: "PaleoAncestry"
date: 2021-05-17
client: true
logo: "images/clients/paleoancestry.png"
tags: ["Python","AWS","Cloud Architecture"]
---

The scope of this project was to optimize and move an application to the cloud. The source code for the analysis pipeline was created for the use of university super-computer equipment.

Another freelancer developed the frontend for the B2C offering. Any performance, UI/UX and scaling issues were not part of my tasks.

I developed several different architecture patterns. Together with the customer we decided on the most suitable solution. Cost, scalability, and maintainability were the main factors to consider in this situation. 

We finally settled on a decoupled 3 tier application stack. As the communication layer between the different application parts we chose SQS. For the frontend hosted on Heroku, I created an IAM user limited to read and write to the relevant SQS queues. Everything else used IAM roles. 

SQS also enabled us to scale horizontally with ease. Cloudwatch allowed the autoscaling group to take action on the age and amount of messages in the queues.

As a persistent store for eventual reanalysis of customer data S3 was used. This also enabled simple lifecycle management of objects to help with data minimization.  

The source code that this project was based on, were several bash and a few python 2 scripts. I migrated everything to Python 3 and minimized interprocess communication. 

After the rewrite an analysis took only 30 minutes, down from 24 hours. I also reduced the hardware requirements by having the most intensive task precomputed. The whole pipeline was running on a t2.medium EC2 instance, down from a supercomputer.