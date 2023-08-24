---
title: "How To Train Your Dragon: An Application of Ghidrathon to Automate Cyber Threat Intelligence Gathering"
layout: post
---

The continued development and industry-wide adoption of Mandiant's (Now part of Google Cloud) open-source project, **capa**, has allowed me the pleasure to become a Contributor to the project via Google's Summer of Code 2023 (GSoC). My project was the **capa: Ghidra Integration**, and its goal was to use Ghidrathon to enable capa to work with Ghidra's binary analysis framework as a backend. This article will go in-depth as to how we trained a dragon and enabled more reverse engineers to consult with decades worth of experience in a matter of minutes.

This project came with many hurdles to overcome such as:
* Executing Python3 and its modules within the Java Virtual Machine
* Exposing the Ghidra API to Python3
* Addressing differences between the Ghidra backend and other existing ones
* And many more...


