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


