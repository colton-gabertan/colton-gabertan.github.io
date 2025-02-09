---
title: "Windows 11 Security Hardening: Using Wazuh and CIS Benchmarks"
layout: post
---

### Center for Internet Security Background

The Center for Internet Security (CIS) is a non-profit organization that garners consensus from information security professionals around the world for best-practices in deploying production systems. These benchmarks are widely recognized, developed, and accepted by government entities, industries, and academic bodies. By actively meeting the standards of these benchmarks, organizations are able to better protect their assets by producing hardened and compliant systems for their services and users. 

More information may be found here: https://learn.cisecurity.org/benchmarks

### Wazuh's Agentic Architecture

Wazuh is an open-source security platform that combines common enterprise requirements such as endpoint security, threat detection & intelligence, and security information and event management (SIEM). It is designed to help an organization of any size protect digital assets that may be distributed across on-prem, virtualized, containerized, and cloud environments. 

It deploys as a centralized server in which we can install agents on endpoints that report back to Wazuh. For the purposes of this lab, I’ve chosen to focus on Wazuh’s out-of-the-box solution that allows for instant CIS Benchmarking on endpoints. 

The architecture within my homelab environment is as follows:
<div align="center">
    <img src="/assets/WazuhCISDiagram.png">
</div>

### Deploying an Agent

Once the server is configured, deploying an agent is as easy as selecting the desired operating system, pointing it back to the Wazuh instance, and installing it on the endpoint. 
<div align="center">
    <img src="/assets/wazuhAgent1.png">
    <img src="/assets/wazuhAgent2.png">
</div>

It quickly checks-in after installation from the endpoint and immediately begins to run its benchmarking tests against the CIS Benchmarks v3.0.0 for my Windows 11 machine.

### Configuration Assessment Findings

The Wazuh Agent was able to leverage simple commands and registry checks to map against the Center for Internet Security (CIS) critical security controls as well as other frameworks such as CMMC, PCI-DSS, and SOC 2. 
<div align="center">
    <img src="/assets/CISCompliance.png">
</div>

On my particular endpoint, I decided to keep the default configurations from Microsoft to see how much remediation I would require in order to be fully CIS-compliant. Wazuh’s checks revealed that I was able to pass 124 checks, but failed 347, netting me a score of 26%. 
<div align="center">
    <img src="/assets/initialFindings.png">
</div>

Although there are a lot of remediations to perform, each alert provides a highly detailed rationale for the finding, remediation procedures, and even a description of the policy setting. By following each procedure, we can continuously improve our standing against the CIS Benchmark.

### Example Remediation

Here we have a misconfiguration detected by the benchmarking tool. My current settings did not enforce a minimum password length.
<div align="center">
    <img src="/assets/CISInfo.png">
</div>

Following the remediation procedure, I enabled this setting within my local group policy editor and applied it to the system. After re-starting the agent, I forced another run of the benchmarking test. 
<div align="center">
    <img src="/assets/gpedit.png">
</div>

Checking back in the Configuration Assessment Dashboard, we can see that I’m passing more tests. We’ve increased from 124 passes to 126.
<div align="center">
    <img src="/assets/exampleRemediation1.png">
    <img src="/assets/exampleRemediation2.png">
</div>

After a few more configuration changes, following the procedures for each, I was able to slowly increase my overall standing up to 32%. And, as we continue, our endpoint becomes increasingly compliant to multiple industry-standard frameworks for an enterprise Windows 11 Pro workstation.
<div align="center">
    <img src="/assets/continuousImprovement.png">
</div>

### Continuous Improvement

Security is not a static state, but a continuous process. CIS benchmarks provide a valuable framework for ongoing improvement. By regularly assessing compliance statuses, organizations can adapt to new threats and refine their security practices.



