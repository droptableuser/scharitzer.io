---
title: "Footstock"
date: 2020-01-08
client: true
logo: "/images/clients/footstock.png"
tags: ["Github Actions", "AWS"]
---

At Footstock I moved from manual deployments to automated deployments on AWS. Footstock hosted all its source code on GitHub. We leveraged Github Actions as the continuous integration and delivery platform. 

At first, I created separate VPCs for production, testing, and development. Regulatory requirements made this separation necessary.

I built infrastructure as code playbooks. They provided an easy way to create and tear down different setups fast. Technologies used were Fargate, RDS, KMS, IAM, and ALB. 

I also automated the backup mechanisms for local development purposes. Those backup mechanisms also performed data cleaning for privacy.