---
title: Introduction to DevSecOps - Part 1 
date: "2023-02-10T01:10:01.123Z"
description: "For me, this is the future."
---

I've been a DevOps Engineer since *roughly* 2011, or some time around that when the fashion for pure Systems Administration became a lot more automated, and the start of the 'shift left' movement started, with integrating the tooling that we know and love now into deployment architectures. 

I've also spent a significant portion of my working career in security-focussed roles, either from a purely application security perspective, or a more holistic standpoint on infrastructure security. 

--

In today’s rapidly evolving technological landscape, software development and deployment have become increasingly complex. As a result, organisations are now placing a higher priority on security in their development and deployment processes, leading to the emergence of DevSecOps.

Much like how we went from the Sysadmin era to the DevOps era, a similar shift is happening now with the push towards 'shifted-left' security. 

DevSecOps shifts security "left" by incorporating security considerations and practices into the early stages of the software development lifecycle, rather than treating security as an afterthought that is only considered later in the process.

In more traditional software development models, security teams are often not involved until later in the development process, during testing or even after deployment. By that time, the code may have already been written and deployed, making it more difficult and expensive to address any security vulnerabilities that are discovered.

With a move towards implementing DevSecOps, security teams are involved from the beginning of the development process, working closely with developers to ensure that security is considered and integrated at every stage of the software development lifecycle. 

By shifting security left, organisations are better equipped to identify and address security risks early on, reducing the likelihood of data breaches and cyber-attacks. 

It also helps to ensure that security is built into the software from the start, rather than added as an afterthought, leading to more secure software that is easier and less expensive to maintain over time.

One of the key principles of DevSecOps is the use of automated tools and processes to ensure that security is integrated into the development and operations processes. This includes the use of security testing tools, security orchestration tools, and continuous integration/continuous deployment (CI/CD) pipelines.

From an AWS Perspective, I'm looking at the integration of Infrastructure as Code (IaC) scanning tools into the build pipeline for early detection of security misconfigurations, which if they were deployed, could lead to operational vulnerabilities. 

These are a couple of tools that I've used for IaC security scanning, [tfsec](https://github.com/aquasecurity/tfsec) and [checkov](https://www.checkov.io/).  

I also like to have [Prowler](https://github.com/prowler-cloud/prowler) running in a scheduled task (or triggered by a CI pipeline execution) to catch misconfigurations that aren't picked up by IaC scanning. 

Within AWS, I'm looking for teams to be deploying GuardDuty as a bare minimum, ideally with VPC Flow Logging enabled. 

AWS Security Hub is a centralised security management platform that provides a comprehensive view of the security and compliance posture of an organisation's AWS accounts. Security Hub aggregates and prioritises security findings from multiple AWS services, such as GuardDuty, as well as from third-party security solutions such as Prowler and so on.

It's important to note that integrating AWS GuardDuty with AWS Security Hub is optional, and you can choose to use either service on its own if it meets your specific security needs. 

However, by integrating the two services, you can take advantage of their combined capabilities to gain a more comprehensive view of your security posture and to respond to potential security incidents more effectively.

In conclusion, DevSecOps tooling plays a critical role in ensuring the security of modern software development and delivery processes. 

By automating security tasks and integrating security into the development lifecycle, organisations can improve the security of their applications and systems while also reducing the time and effort required to perform manual security tasks.

