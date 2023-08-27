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

After defining the class, I created a set of Ghidra feature extractor modules, namely `global_.py`, `file.py`, `function.py`, `basicblock.py`, and `insn.py` respectively. Each feature extractor module becomes scoped by increasing granularity. The scopes are as follows:

* Global Scope
  * Operating System detection
  * Architecture detection
* File Scope
  * File format & sections
  * File imports & exports
  * File strings
  * Statically-linked library functions
  * Embedded portable executable (PE) file detection
* Function Scope
  * Function call references
  * Loops within functions
  * Recursive function calls
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

# Github Contributions

| Deliverable | Pull Request |
|---|---|
| Global Feature Extraction | [Ghidra: Implement Global Feature Extraction](https://github.com/mandiant/capa/pull/1526)
| File Feature Extraction | [Ghidra: Implement File Feature Extraction](https://github.com/mandiant/capa/pull/1564)
| Function Feature Extraction | [Ghidra: Implement Function Feature Extraction](https://github.com/mandiant/capa/pull/1597)
| Basic Block Feature Extraction | [Ghidra: Implement Basic Block Feature Extraction](https://github.com/mandiant/capa/pull/1637)
| Instruction Feature Extraction | [Ghidra: Implement Instruction Feature Extraction](https://github.com/mandiant/capa/pull/1670)
| CI Workflow | [Ghidra: Implement CI Workflow](https://github.com/mandiant/capa/pull/1529)
| Ghidra Backend Unit Test | [Ghidra: Implement Unit Test](https://github.com/mandiant/capa/pull/1727)

All tracking of this project may be accessed via: [capa-ghidra pull requests](https://github.com/mandiant/capa/pulls?q=is%3Apr+label%3Aghidra). The main feature branch used to track GSoC-associated changes may be accessed [here](https://github.com/mandiant/capa/tree/backend-ghidra)

# Mini Demo

### Example: Ghidra Feature Extractor vs. Practical Malware Analysis Lab 16-01.exe_

<img src="/assets/gsoc_demo.gif">

### capa Ghidra Feature Extractor Report

```
wumbo@flarenix:~/capa$ analyzeHeadless ~/Desktop/ghidra_projects/ capa_test -process 'Practical Malware Analysis Lab 16-01.exe_' -ScriptPath ./capa/ghidra/ -PostScript capa_ghidra.py "./rules"
[...]
 (AutoAnalysisManager)
INFO  REPORT: Analysis succeeded for file: /Practical Malware Analysis Lab 16-01.exe_ (HeadlessAnalyzer)
INFO  SCRIPT: /home/wumbo/capa/./capa/ghidra/capa_ghidra.py (HeadlessAnalyzer)
┍━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ md5                    │ 7faafc7e4a5c736ebfee6abbbc812d80                                                   │
│ sha1                   │                                                                                    │
│ sha256                 │ 309217d8088871e09a7a03ee68ee46f60583a73945006f95021ec85fc1ec959e                   │
│ os                     │ windows                                                                            │
│ format                 │ Portable Executable (PE)                                                           │
│ arch                   │ x86                                                                                │
│ path                   │ /home/wumbo/capa/./tests/data/Practical Malware Analysis Lab 16-01.exe_            │
┕━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

┍━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ ATT&CK Tactic          │ ATT&CK Technique                                                                   │
┝━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┥
│ DEFENSE EVASION        │ Indicator Removal::File Deletion T1070.004                                         │
│                        │ Indicator Removal::Timestomp T1070.006                                             │
│                        │ Modify Registry T1112                                                              │
├────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
│ DISCOVERY              │ File and Directory Discovery T1083                                                 │
│                        │ Query Registry T1012                                                               │
│                        │ System Information Discovery T1082                                                 │
├────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
│ EXECUTION              │ Command and Scripting Interpreter T1059                                            │
│                        │ Shared Modules T1129                                                               │
│                        │ System Services::Service Execution T1569.002                                       │
├────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
│ PERSISTENCE            │ Create or Modify System Process::Windows Service T1543.003                         │
┕━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

┍━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ MBC Objective               │ MBC Behavior                                                                  │
┝━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┥
│ ANTI-BEHAVIORAL ANALYSIS    │ Debugger Detection::Process Environment Block BeingDebugged [B0001.035]       │
│                             │ Debugger Detection::Process Environment Block NtGlobalFlag [B0001.036]        │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ COMMUNICATION               │ Interprocess Communication::Create Pipe [C0003.001]                           │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ DEFENSE EVASION             │ Self Deletion::COMSPEC Environment Variable [F0007.001]                       │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ DISCOVERY                   │ File and Directory Discovery [E1083]                                          │
│                             │ System Information Discovery [E1082]                                          │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ EXECUTION                   │ Command and Scripting Interpreter [E1059]                                     │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ FILE SYSTEM                 │ Copy File [C0045]                                                             │
│                             │ Delete File [C0047]                                                           │
│                             │ Get File Attributes [C0049]                                                   │
│                             │ Read File [C0051]                                                             │
│                             │ Writes File [C0052]                                                           │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ OPERATING SYSTEM            │ Environment Variable::Set Variable [C0034.001]                                │
│                             │ Registry::Delete Registry Value [C0036.007]                                   │
│                             │ Registry::Query Registry Value [C0036.006]                                    │
│                             │ Registry::Set Registry Key [C0036.001]                                        │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ PROCESS                     │ Create Process [C0017]                                                        │
│                             │ Terminate Process [C0018]                                                     │
┕━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

┍━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ Capability                                           │ Namespace                                            │
┝━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┥
│ check for PEB BeingDebugged flag (24 matches)        │ anti-analysis/anti-debugging/debugger-detection      │
│ check for PEB NtGlobalFlag flag (24 matches)         │ anti-analysis/anti-debugging/debugger-detection      │
│ self delete                                          │ anti-analysis/anti-forensic/self-deletion            │
│ timestomp file                                       │ anti-analysis/anti-forensic/timestomp                │
│ create pipe                                          │ communication/named-pipe/create                      │
│ accept command line arguments                        │ host-interaction/cli                                 │
│ query environment variable (4 matches)               │ host-interaction/environment-variable                │
│ set environment variable                             │ host-interaction/environment-variable                │
│ get common file path                                 │ host-interaction/file-system                         │
│ copy file                                            │ host-interaction/file-system/copy                    │
│ delete file                                          │ host-interaction/file-system/delete                  │
│ check if file exists                                 │ host-interaction/file-system/exists                  │
│ get file attributes                                  │ host-interaction/file-system/meta                    │
│ read file on Windows (2 matches)                     │ host-interaction/file-system/read                    │
│ write file on Windows (3 matches)                    │ host-interaction/file-system/write                   │
│ check OS version                                     │ host-interaction/os/version                          │
│ create process on Windows (2 matches)                │ host-interaction/process/create                      │
│ terminate process (2 matches)                        │ host-interaction/process/terminate                   │
│ query or enumerate registry value (2 matches)        │ host-interaction/registry                            │
│ set registry value                                   │ host-interaction/registry/create                     │
│ delete registry value                                │ host-interaction/registry/delete                     │
│ create service                                       │ host-interaction/service/create                      │
│ delete service                                       │ host-interaction/service/delete                      │
│ modify service                                       │ host-interaction/service/modify                      │
│ link function at runtime on Windows                  │ linking/runtime-linking                              │
│ persist via Windows service                          │ persistence/service                                  │
┕━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

Script /home/wumbo/capa/./capa/ghidra/capa_ghidra.py called exit with code 0
```



