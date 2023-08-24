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


## What is capa?
<img align="center" src="/assets/capa_logo.png">

capa is the Mandiant FLARE Team's open-source project that automatically identifies capabilities in executable files. This tool is ran against suspected malware samples to map its extracted functionality to the Mitre ATT&CK Framework and allows reverse engineers to jump straight into areas of interest. This results in significantly reducing the time spent sifting through overwhelming amounts of low-level code to gain actionable intelligence against threats lurking within binaries. 

<img src="/assets/capa_run.gif">

Since its conception, it has been widely integrated with other tools in the malware analysis and reverse engineering community. These tools include [VirusTotal], [HexRay's IDA Pro], [Vivisect], [Binary Ninja], and now [Ghidra]. 

[VirusTotal]: https://blog.virustotal.com/2023/01/mandiants-capa-goresym-to-reinforce-vts.html
[HexRay's IDA Pro]: https://hex-rays.com/IDA-pro/
[Vivisect]: https://vivisect.readthedocs.io/en/latest/vivisect/intro.html
[Binary Ninja]: https://binary.ninja/
[Ghidra]: https://ghidra-sre.org/

## What is Ghidra?
<img align="center" src="/assets/ghidra_logo.png" height=250 width=400>

[Ghidra] is the most popular open-source disassembly framework used by the greater reverse engineering and malware analysis community. It was created and is maintained by the National Security Agency and has garnered its popularity due to its effectiveness, support, and for being free to use. It is fully equipped with a user interface, scripting console, and headless mode. Other features that is sports are the ability to act as a server as well as a powerful decompilation engine alongside its disassembly capabilities. 

<img src="/assets/ghidra_ui.png">