---

layout: col-sidebar
title: OWASP Faction
tags: reporting, vulnerability management, pentesting
level: 2
type: code
pitch: Faction is a pentesting assessment collaboration framework that helps automate pentesting reports, schedule larege scale assessments, manage vulnerability tracking, and increase collaboration in your teams. 

---

# OWASP FACTION PenTesting Report Generation and Collaboration Framework

 ![GitHub last commit](https://img.shields.io/github/last-commit/factionsecurity/faction) ![GitHub Release Date - Published_At](https://img.shields.io/github/release-date/factionsecurity/faction)

[![Bluesky](https://img.shields.io/badge/Bluesky-0285FF?logo=bluesky&logoColor=fff)](https://bsky.app/profile/factionsecurity.com)


![image](/www-project-faction/assets/images/290761747-d9237bed-302f-4e6a-9716-22ae88d0dc36.png)


### Become a Sponsor ❤️
If you like the project and would like to see it advance then consider being a sponser. All sponsers get access to the Faction discord server and will have bug reports priotirized. Just click the sponsor links at the top of this repo. 

[Sponsor via GitHub Sponosrs](https://github.com/sponsors/factionsecurity)

## Introduction

FACTION is your entire assessment workflow in a box. With FACTION you can:
1. Automate pen testing and security assessment Reports
1. Peer review and track changes for reports
1. Create customized DOCX templates for different assessment types and retests
3. Real-time collaboration with assessors via the web app and [Burp Suite Extensions](https://github.com/factionsecurity/Faction-Burp)
4. Customizable vulnerability templates with over 75 prepopulated
5. Easily manage assessment teams and track progress across your organization
6. Track vulnerability remediation efforts with custom SLA warnings and alerts  
7. Full Rest API to integrate with other tools                     

Other Features:           
1. LDAP Integration       
1. OAUTH2.0 Integration
1. SMTP integration 
1. Extendable with Custom Plugins similar to Burp Extender.
2. Custom Report Variables

__Want to see it in action?__ -> [Faction YouTube Channel](https://www.youtube.com/@factionsecurity/videos)

## Quick Setup
__Requirements__
- Java JDK11 
- Maven (for building the project)
- (Optional for VM). Mongo DB requires a CPU with AVX support. You may run into this issue if using [Oracle Virtual](https://www.mongodb.com/community/forums/t/could-not-start-mongodb-5-0-running-oracle-linux-on-virtualbox/120524/10) Box or [Kubernetes](https://stackoverflow.com/questions/70818543/mongo-db-deployment-not-working-in-kubernetes-because-processor-doesnt-have-avx)

Run the following commands to build the war file and deploy it to the docker container. 
```
git clone git@github.com:factionsecurity/faction.git
cd faction
docker-compose up --build
```

Once the containers are up you can navigate to http://127.0.0.1:8080 to access your FACTION instance. 
On the first boot, it will ask you to create an admin account. 

## Import the Vulnerability Templates
1. Navigate to Templates -> Default Vulnerabilities
2. Click Update from Faction. 

## Customize reports
You can find out more information about creating your own custom report templates here:
[Custom Security Report Templates - Faction Security](https://docs.factionsecurity.com/Custom%20Security%20Report%20Templates/)

## Burp Suite Extension
Faction is in the Burp BApp store but you can also get it from our github:
[Burp Suite Extensions](https://github.com/factionsecurity/Faction-Burp)

## Manuals and Tutorials
[Manual](https://docs.factionsecurity.com/)


## Screenshots
__Vulnerability Templates__
![image](/www-project-faction/assets/images/vuln-templates.png)

__Assessment Scheduling__
![image](/www-project-faction/assets/images/scheduling.png)


__Peer Review and Track Changes__
![image](/www-project-faction/assets/images/peerreview.png)


__Remediation/Retest Queue__
![image](/www-project-faction/assets/images/retests.png)

__Schedule Retests__
![image](/www-project-faction/assets/images/scheduleretests.png)

__Assessor Retest Interface__
![image](/www-project-faction/assets/images/assessorretest.png)

__Vulnerability Status Tracking__
![image](/www-project-faction/assets/images/vulnstatus.png)

# Faction App Store

Faction 1.2 introduces the App Store! The Faction App Store will make it easier for developers to extend faction. Faction Extensions can be used to trigger custom code when certain events happen in your workflow like sending all vulnerbilities to Jira when the assessment is complete or update a tracking system when retests pass or fail. More information can be found in the [documentation site](https://docs.factionsecurity.com). 

### ⭐️ Jira Integration and AppStore Dashboard
![image](/www-project-faction/assets/images/appstore.png)

Note you can reorder extensions so that updates for one can affect updates to the next. 

### ⭐️ Extensions for Custom Graphics
Extensions will also allow custom bar charts to your reports:
![image](/www-project-faction/assets/images/appstore2.png)

Generated report with graphics:
![image](/www-project-faction/assets/images/report.png)



### Current Road Map
- Open sourced the base application on github in dec 2023
- Adding API and plugin features - March 2024
- Streamlined remediation workflows - August 2024
- Update backend and code refactor - October 2025
- Adding 4 more plugins and integrations - December 2025
- Overhaul the UI to be more modern and simple - March 2026

