---
layout: post
title: DevSecOps Pipeline - Introduction
date: 2024-04-12 19:00:00 +0000
category: [DevSecOps]
tags: [devsecops, aws, git]
---

--- 
Its been a while since I have written any posts and a lot has happened over the past year. I learnt a lot of things and did a few projects. One of the major thing I learnt was AWS that is **Amazon Web Services**. There was a requirement for a project where one of the primary focus was security integrated within the CI/CD pipeline on AWS (service named AWS Pipeline). 
<br>
So, in this post I'll be giving a introduction of the DevSecOps pipeline we developed as a team and in the following posts, I'll be walking you guys through each open-source tool that we have used to create this DevSecOps pipeline.

---
## <ins> Introduction </ins>
---

![Cover](/assets/img/devsecops/cover.png)
First and foremost, lets answer the question **What is a DevSecOps pipeline?**
- DevSecOps Pipeline are automated workflows in a CI/CD pipeline which are integrated with security tools that scan, test and monitor at every stage throughout the software developement lifecycle.

---
## <ins>Tools</ins>
---
Here is the list of tools that are used in the pipeline:
- **CI/CD:** AWS CodePipeline
- **Version Control:** AWS CodeCommit
- **Secrets Scanning:** GitLeaks
- **SAST(Static Application Security Testing):** SonarQube
- **Container Scanning:** Clair
- **DAST(Dynamic Application Security Testing):** OWASP ZAP
- **IAC Scanning:** Checkov
- **Infra Scanning:** Nuclei
- **Centralized Vulnerability Management:** DefectDojo Community edition


---
## <ins> FlowChart</ins>
---
The Flowchart of our pipeline goes like this (referred from OWASP website):
![FlowChart](/assets/img/devsecops/flow.png)

That's all for the introduction of the pipeline. See you guys on the next post!!