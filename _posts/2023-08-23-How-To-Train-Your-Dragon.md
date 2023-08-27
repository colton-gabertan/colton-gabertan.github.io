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
<div align="center">
    <img src="/assets/capa_logo.png" height=200 width=400>
</div>

capa is the Mandiant FLARE Team's open-source project that automatically identifies capabilities in executable files. This tool is ran against suspected malware samples to map its extracted functionality to the [Mitre ATT&CK Framework] and allows reverse engineers to jump straight into areas of interest. This results in significantly reducing the time spent sifting through overwhelming amounts of low-level code to gain actionable intelligence against threats lurking within binaries. 

<img src="/assets/capa_run.gif">

>Since its conception, it has been widely integrated with other tools used by the malware analysis and reverse engineering community. These tools include: 
>* [VirusTotal] 
>* [HexRay's IDA Pro] 
>* [Vivisect] 
>* [Dnfile] 
>* [Binary Ninja] 

[Mitre ATT&CK Framework]: https://attack.mitre.org/
[VirusTotal]: https://blog.virustotal.com/2023/01/mandiants-capa-goresym-to-reinforce-vts.html
[HexRay's IDA Pro]: https://hex-rays.com/IDA-pro/
[Vivisect]: https://vivisect.readthedocs.io/en/latest/vivisect/intro.html
[Dnfile]: https://github.com/malwarefrank/dnfile
[Binary Ninja]: https://binary.ninja/

## What is Ghidra?
<div align="center">
    <img src="/assets/ghidra_logo.png" height=200 width=400>
</div>

[Ghidra] is the most popular open-source disassembly framework used by the reverse engineering and malware analysis community. It was created and is maintained by the National Security Agency. Ghidra has garnered its popularity due to its effectiveness, support, and for being free to use. It is fully equipped with a user interface, scripting console, and headless mode. Other features that it sports are the ability to act as a server as well as having a powerful decompilation engine alongside its disassembly capabilities. 

<img src="/assets/ghidra_ui.png">

> Because of its popularity, a huge motivation for the GSoC project was to bring capa into the hands of analysts who prefer to use Ghidra as their main tool.

[Ghidra]: https://ghidra-sre.org/

## What is Ghidrathon?

[Ghidrathon] solves the biggest issue that has been associated with Ghidra and is what ultimately opens the door for it to be integrated into a wealth of production information security analysis tools. One major issue that has held the Ghidra Framework back is that it is written in Java and only supported Jython, a Java-Python2 scripting interface. As Python has evolved and matured, many tools have opted to be written in and designed for Python3, which is incompatible with the Jython interface for the Ghidra API.  

**Visualization of How Ghidrathon Works:**

<img src="/assets/ghidrathon_vis.gif">

> Ghidrathon leverages another project called [Jep] to implant a CPython interpreter into the Java Virtual Machine (JVM) via the Java Native Interface (JNI). This allows the JVM to to execute Ghidra's Java bytecode as well as inject Python3 code to be executed by the CPython interpreter(s). Ghidrathon then bridges JVM & Python3 data in order to expose the Ghidra API and associated objects to interact seamlessly together. 

[Ghidrathon]: https://www.mandiant.com/resources/blog/ghidrathon-snaking-ghidra-python-3-scripting
[Jep]: https://github.com/ninia/jep

## Training the Dragon:

