---
title: "Google Summer of Code 2023 - capa: Ghidra Integration"
layout: post
---

<div align="center">
    <img src="/assets/ghidra_backend_logo.png">
</div>

capa is the Mandiant FLARE team's open source tool that is used to automatically identify capabilities of programs. Reverse engineers and malware analysts run capa against suspected malware in order to uncover its underlying functionality by matching extracted features to a well-defined collection of rules. This allows analysts to quickly narrow their scope down to areas of interest within a sample, taking advantage of significant speed gains provided by years of cumulative  research.  

Since its conception, capa has received industry-wide adoption via platform integrations and by supporting several popular backends spawned from other open-source & proprietary infosec projects. These projects include: [Virustotal], [HexRay's IDA Pro], [Vivisect], [Dnfile], and [Binary Ninja]. My goal this summer was to further expand capa adoption by integrating one of the most popular software reverse engineering frameworks, Ghidra, as a backend.  

[Mitre ATT&CK Framework]: https://attack.mitre.org/
[VirusTotal]: https://blog.virustotal.com/2023/01/mandiants-capa-goresym-to-reinforce-vts.html
[HexRay's IDA Pro]: https://hex-rays.com/IDA-pro/
[Vivisect]: https://vivisect.readthedocs.io/en/latest/vivisect/intro.html
[Dnfile]: https://github.com/malwarefrank/dnfile
[Binary Ninja]: https://binary.ninja/


# Deliverables and Status

<div align="center">
    <img src="/assets/gsoc_project_progress.png">
</div>

All planned deliverables for the Google Summer of Code (GSoC) period have been completed and integrated. The main functionality, the feature extractors, serve as the core of the Ghidra backend, allowing us to tap into rich databases populated by Ghidra analysis. In order to implement the Ghidra feature extractors, capa is designed to use an abstract `FeatureExtractor` class for each backend. In my case, we used the `GhidraFeatureExtractor` class to initialize capa's access to the Ghidra databases. 

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

# Challenges

The Ghidra feature extractor was only made possible by another Mandiant FLARE team project, [Ghidrathon](https://github.com/mandiant/Ghidrathon). Ghidra natively supports Jython, a Java-Python integration that is only compatible with Python 2. This rendered Python 3 modules and projects, such as capa, incompatible with the native scripting interface. Additionally, Ghidrathon had not been used to this extent, so a large portion of development went towards uncovering potential issues that may arise in similar projects.   

### Found Ghidrathon Issues -

**Data Type Conversion:**

The first problem encountered was identifying data type conversions to be handled as we passed these constructs to be processed by either the Java side or the Python 3 side. The most prominent example was byte extraction. Bytes returned by calls to the Ghidra API would be returned as `singed ints`, versus a compatible Python 3 `bytes` type. 

This required us to handle the conversion in Python 3 before passing this data to the appropriate routines and functions. The first step was to convert the `signed ints` to `unsigned ints`. Fortunately, all this required was a bitwise `& 0xFF` for each `int`. From there, we needed to cast it to the Python 3 `bytes` type. The original implementation took advantage of the builtin `to_bytes()` function; however, our performance testing revealed that this was incredibly inefficient.

To remediate the performance issue, we took advantage of Python 3 list comprehensions as well as the builtin `byte()` casting. This improved the speed of our conversions approximately 100x. The pull request addressing this issue may be found [here](https://github.com/mandiant/capa/pull/1761). 

**CPython Module Accessibility:**

Ghidrathon implants CPython interpreters into the Java Virtual Machine (JVM) in order to allow Python 3 scripts to be injected into the JVM. Originally, this caused issues as re-importing a Python module would cause crashes, due to modules not supporting multiple instances of an interpreter within the same process. Because we run everything from a single Java process for Ghidra feature extraction, this required an entire re-architecting of Ghidrathon.   

To remedy this issue, Ghidrathon implemented shared interpreters in [Ghidrathon v2.2.0](https://github.com/mandiant/Ghidrathon/releases/tag/v2.2.0) to allow accessibility of each module to each shared interpreter. 

**Maintaining the Ghidra State of Objects for each CPython Shared Interpreter:**

After implementing shared interpreters, this also meant that the context of each object exposed to the interpreters remained the same for each one. This resulted in sequential runs of the Ghidra feature extractor to be using the wrong state of an object, therefore producing incorrect results. Ghidrathon does the crucial job of exposing necessary objects that relate to each Ghidra database. Namely, the Ghidra feature extractor heavily makes use of `currentProgram` and `monitor` to access data needed for capa processing.  

These objects were originally accessible as normal Python 3 objects, for example, `currentProgram.getFunctionManager()`. However, to maintain the proper state, these exposed objects were added to the `builtins` scope, changing the way we interact with them. Now, the same line as above would need to be treated as a call to a module, i.e. `currentProgram().getFunctionManager()`.

These changes were addressed in the release of [Ghidrathon v3.0.0](https://github.com/mandiant/Ghidrathon/releases/tag/v3.0.0).

### Conclusion -

As Ghidrathon had been a relatively new and untested project, the capa: Ghidra Integration served as a great stepping stone to improving the feasibility of having it support others. This contributes greatly to the industry by allowing most existing Python 3 binary analysis tooling access to the feature-rich Ghidra Framework. 

# Acknowledgement:

I'd like to thank the Mandiant FLARE team and all of the GSoC mentors who volunteered their time to guide myself and others. A special acknowledgement goes to my primary mentor, Mike Hunhoff. He contributed greatly to my growth by asking me the questions I failed to ask myself, providing sound advice & great design suggestions, and for the super quick Ghidrathon changes necessary to roll the project to production!


