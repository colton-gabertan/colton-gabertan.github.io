---
title: "Live Malware Reverse Engineering: WANACRY (In Progress/ Revamping)"
layout: post
---

After becoming interested in malware analysis and reverse engineering, I decided to spin up a honeypot to collect live samples of malware. When analyzing the binaries that my honeypot managed to capture, I found that the most common one was detected as the infamous WannaCry Ransomware.
> *If you're interested, I have a report of my honeypot project [here]*

This repo will be going over my process of analysis for this sample, explaining common reverse engineering techniques with the goals of:
* Finding host-/ network-based signatures for detection
* Determining exactly what the malicious binary does from high to low level


---
## Table of Contents:

* [launcher.dll Analysis - Static](#launcher.dll-static)
* [mssecsvc.exe Analysis - Static](#mssecsvc.exe-static)
* [taskche.exe Analysis - Static](#taskche.exe-static)

---
## Malware Analysis Environment

> Tool(s) Used: 
> * [FLAREVM]
>
> To begin, I decided to isolate my malware analysis environment by working in a virtual machine and cutting off its connection to my network. WannaCry is commonly spread as a worm, which is exactly how I caught it. Setting up the vm environment without network connectivity is essential in ensuring that none of it leaked into my local network during analysis.
>
> Typically virtual machine agents will allow us to change our adapter's settings to be host-only instead of NAT or bridged, which is my preferred setting for malware analysis. Due to our analysis being static, meaning we will not run the binary, there is low risk to us; however, it is a good habit to take precautions when working with actual malicious software.

## launcher.dll Analysis - Static: <a name="launcher.dll-static"></a>

**Basic Static Analysis:**

Before disassembling the binary, I wanted to take a look at the file headers to see what I may be dealing with. This information will let me know if the malware is packed or obfuscated at all. I was introduced to a nice tool for this called, CFF Explorer VIII, which comes standard in the FLAREVM.

Figure 1 below is the output when initially loading the binary. When analyzing Windows malware, knowing its architecture and what kind of file it is has quite a bit of value to it as we'll need to adjust our techniques accordingly. 

![image](https://user-images.githubusercontent.com/66766340/175823535-51106d93-0a7e-4ae4-a00f-8b2cfae21d62.png)
###### Figure 1: launcher.dll file headers

As we can see, the `File Type` section tells us that it is a 32-bit Portable Executable, or PE for short. PE is the file format that Windows uses for both its executables (.exe) and dynamic link libraries (.dll). Inherently, exe's are meant to import and run code from dll's; however, dll's can also contain executable code as well.

Furthermore, the `File Info` section tells us that it is also written in C++ and is a dll. Even in modern malware, it is still common to see 32-bit binaries as modern systems both support that architecture and it allows authors to work with smaller memory constraints.

When dealing with dll's, especially ones we like to classify as launchers, the malware will typically be able to install and execute other components of the malware. There are many techniques to do so, and we'll be taking a look at how WANACRY does it. 

Exploring within CFF Explorer, we can come across its `Export Directory` and this dll contains one called PlayGame. This is illustrated in Figure 1.1.

![image](https://user-images.githubusercontent.com/66766340/175824374-88fb6164-3df0-49b7-9b9f-8d25fe64035a.png)
###### Figure 1.1: PlayGame Export

The `Import Directory` also contains some valuable information as well. This section lets us know which components of the Windows API that the malware uses, which in turn hints at its functionality. Figure 1.2 displays the imports from KERNEL32.dll, which is very commonly imported by malware as it provides essential operating system functionality.

![image](https://user-images.githubusercontent.com/66766340/175824971-8e7fcfe4-2483-4f25-8ac8-0fb414e7c5c6.png)
###### Figure 1.2: Windows API Imported Functions

Luckily this malware is straightforward and does nothing to hide its imports. Some samples may import functions by ordinal so we can't see the names or may even use only GetProcAddress and LoadLibrary to dynamically import others, forcing us to manually extract the imports.

This malware has the ability to write to and create files on the disk as well as handle its resource section. This prompts us to explore the `Resource Directory` next. This is also a common place that malware can embed extra configs or binary data for extraction and use. Based on the imports, it is highly likely that this malware will extract another executable file from its resource section and launch it.

![image](https://user-images.githubusercontent.com/66766340/175825172-bef5f2bd-2848-4d7a-9699-a849deec068c.png)
###### Figure 1.3: Resource Directory

There is in fact a resource directory named W101 with an ID of 1033. We can take a quick peek at its resource using Resource Hacker to get an idea of what the malware may be using its resource for.

As suspected, the malware's resource section contains another PE file. With the malware's imports combined with its resource, inductive reasoning tells us that this dll will most likely write this resource to another executable or dll on the disk to continue execution. The binary that we are observing right now is the main malware's launcher.

![image](https://user-images.githubusercontent.com/66766340/175825449-9b25c743-8a03-4e86-be4d-757db3b20497.png)
###### Figure 1.4: Resource contains another executable PE file

The tell-tale sign that this is a pe file in the hexdump is that it contains the DOS header. Usually located at offset 0, we can find 0x4D5A or 'MZ'. This is the sort of file signature for PE files. Based on this hexdump, this PE file would actually be slightly corrupted but very easily fixable. We will do so later in the analysis of the full malware.

To wrap up our basic static analysis, we can take a look at the malware's strings. I suspect there to be very many within this binary specifically as it contains more executables within it; however, I will highlight the interesting/ useful ones.

The FLAREVM comes with a tool called `FLOSS`, the Fireeye Labs Obfuscated String Solver. This handy tool is a replacement for the traditional `strings` tool as it can extract strings constructed on the stack, run decoding routines, and extract static strings. 

```
launcher.dll
C:\%s\%s
WINDOWS
mssecsvc.exe
```
###### Figure 1.5: Interesting strings

For the purposes of this binary, we can observe a few that tip us off to what the name of the embedded binary is as well as the name of the one we are observing. Since it was caught in my honeypot using an automated service, I only got the hash of the file, but the characteristics of this piece of malware are in-line with one to be named `launcher.dll`.

**Advanced Static Analysis:**

Now that the basic static analysis has given us an idea as to what this malware does, or is capable of doing, we can move on to disassembling it. I chose to use IDA. The first place I tend to look at is how many functions the malware has.

We are aware of an export named `PlayGame`, which is likely where the main functionality lies, and the subroutines listed are probably also called by this export function.

![image](https://user-images.githubusercontent.com/66766340/175826245-f66d199e-e15b-4c3d-bd1f-65875f7ac336.png)
###### Figure 2: launcher.dll's functions

We can click on `PlayGame` to explore its disassembly and find that it begins by crafting a format string using the strings we discovered at the end of our basic analysis. 

![image](https://user-images.githubusercontent.com/66766340/175826345-12a7bdf6-19cf-473e-8d0e-36ada7213a85.png)
###### Figure 2.1: PlayGame function

Based on the pushed parameters to `sprintf`, the malware will create a string:

```
"C:\WINDOWS\mssecsvc.exe" 
```

and store it in the Dest address. This address is a global variable in the .data section at 0x10003038. It is also structured as a string with a size of 260. This address also has cross-references to sub_10001016 and sub_1000101AB, which are called after the construction of the format string. 

Upon disassembling sub_10001016, we find the malware deploying its resource handling and file creation imports to write its resource to the disk

![image](https://user-images.githubusercontent.com/66766340/175826732-ab6d33dc-8696-42fe-897f-f51e2e32a985.png)
###### Figure 2.2: sub_10001016

As we can see, this confirms that the file in the malware's resource will be named:

`mssecsvc.exe`

and will be located in the:

`C:\Windows\`

directory. The global variable, `Dest` is pushed as the first parameter to CreateFileA. The format string stored the path and filename in this section of memory in the PlayGame function. After returning from CreateFileA, the malware mov's the value in eax to edi, which is a file handle to the created file if successful. 

![image](https://user-images.githubusercontent.com/66766340/175828151-456febec-218c-41f1-b95a-94e7e799ed54.png)
###### Figure 2.2.1: file path data handling

edi is then checked against a negative value to either bail or continue execution. If the file handle is passed into this register, we can see that it is passed as the hFile parameter for WriteFile. For clarity, I've re-named this function `write_mssecsvc`.

To statically follow the path of execution, we return to PlayGame and explore the next called function, sub_100010AB.

![image](https://user-images.githubusercontent.com/66766340/175827082-961396bc-1200-40c6-88b1-e38ae893d2e7.png)
###### Figure 2.3: sub_100010AB

This subroutine contains the call to CreateProcessA, also with the path stored in Dest passed to it as a parameter. The malware executes the binary it wrote to the disk via process creation. The malware's choice to launch it as a standalone process makes it highly detectable, but its choice of name, `mssecsvc.exe` does not look unusual and may successfully hide in plain sight to people who don't know what they're looking for. We can name this function `launch_mssecsvc_process` and revisit `PlayGame` to get a clearer picture of the export.

![image](https://user-images.githubusercontent.com/66766340/175827765-d995e457-f74a-4ba5-a44a-498e38e7b762.png)
###### Figure 2.4: PlayGame revisited

Furthermore, knowing that WANACRY is ransomware, the nature of this type of malware does not allow for much forensic investigation if it is fully executed successfully on the victim machine. 

To continue our analysis of the full malware, we need to extract the resource now that we know that a process is launched from it. This resource is also named:

`mssecsvc.exe`

**Summary & Indicators of Compromise:**

Given that `launcher.dll` contains no networking functionality, this section of the malware does not contain the worm component; however, it is the launcher. It writes its resource section to an executable file to the path:

C:\Windows\mssecsvc.exe

This leaves a host-based indicator to be the presence of `mssecsvc.exe` as well as a process launched from this executable. There is no evidence of any persistence or privilege escalation capabilities.

> **Host-based Indicators:** \
> File(s) Dropped:
> * C:\Windows\mssecsvc.exe




## mssecsvc.exe Analysis - Static: <a name="mssecsvc.exe-static"></a>

**Extracting the Resource:**






[here]: https://colton-gabertan.github.io/0xcjg-HoneyPot/
[FLAREVM]: https://github.com/mandiant/flare-vm
