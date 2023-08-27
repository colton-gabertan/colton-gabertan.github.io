---
title: "Google Summer of Code 2023 - capa: Ghidra Integration"
layout: post
---

<div align="center">
    <img src="/assets/ghidra_backend_logo.png">
</div>

capa is the Mandiant (Google Cloud) FLARE team's open source tool that is used to automatically identify capabilities of programs. Reverse engineers and malware analysts run capa against suspected malware samples in order to uncover and identify underlying functionality. This allows analysts to report on actionable intelligence against the threats lurking within binaries. 

Since its conception, capa has received industry-wide adoption via platform integrations and by supporting several popular backends spawned from other open-source & proprietary infosec projects. These projects include: Virustotal, IDA Pro, Vivisect, Dnfile, and Binary Ninja. My goal this summer was to further expand capa adoption by integrating one of the most popular software reverse engineering frameworks, Ghidra, as a backend.  


# Deliverables and Status

<div align="center">
    <img src="/assets/gsoc_project_progress.png">
</div>

All planned deliverables for the Google Summer of Code (GSoC) period have been completed and integrated. The main functionality, the feature extractors, serve as the core of the Ghidra backend, allowing us to tap into the rich databases populated by Ghidra analysis. In order to implement the Ghidra feature extractors, capa is designed to use an abstract `FeatureExtractor` class for each backend. In my case, we used the `GhidraFeatureExtractor` class to initialize capa's access to the Ghidra databases. 

After defining the class, I created a set of Ghidra feature extractor modules, namely `global_.py`, `file.py`, `basicblock.py`, and `insn.py` respectively. Each feature extractor module becomes scoped by increasing granularity. The scopes are as follows:

* Global Scope
  * Operating System detection
  * Architecture detection
* File Scope
  * File format & sections
  * File imports & exports
  * File strings
  * Statically-linked library functions
  * Embedded portable executable (PE) file detection
* Basic Block Scope
  * Tight loops
  * Embedded stack strings
* Instruction Scope
  * Instruction mnemonics
  * Number and offset operands
  * Operating System level API calls
  * Embedded bytes and strings
  * Cross-section control flow & file segment access
  * Indirect & referenced calls
  * Bitwise non-zeroing XOR features
  * Windows Process Environment Block (PEB) accesses
  * Obfuscated instruction call exploitation

The final two deliverables of the project were designed to help with the continuity of developing and integrating the Ghidra backend. capa uses Github Actions runners to run a Continuous Integration (CI) workflow that executes automated unit tests for each backend. For Ghidra feature extraction, it is tested on two concurrently running Ubuntu servers that use Python 3.8 and 3.11. Each server is automatically configured with capa and its associated dependencies, the latest Ghidra release, and the latest Ghidrathon release. After configuration, the runner executes an automated unit test via pytest. The automated unit test runs capa with the Ghidra feature extractor against a sample to assert that specific features from each associated scope are found.





