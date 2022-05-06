---
title: "Live Malware Reverse Engineering: WannaCry Ransomware (In Progress)"
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
> Tools Used: \
> *all are included in the FLAREVM
> * [FLAREVM]
> * [Ghidra]
> * [CFF Explorer VIII]
> * [PEiD]
> * [Resource Hacker]
> * [Detect It Easy]
> * [HxD]

## launcher.dll Analysis - Static: <a name="launcher.dll-static"></a>

To begin, I decided to isolate my malware analysis environment by working in a virtual machine and cutting off its connection to my network. WannaCry is commonly spread as a worm, which is exactly how I caught it. Setting up the vm environment without network connectivity is essential in ensuring that none of it leaked into my local network during analysis.
Due to our analysis being static, meaning we will not run the binary, there is low risk to us; however, it is a good habit to take precautions when working with actual malicious software.

Before disassembling the binary, I wanted to take a look at the file headers to see what I may be dealing with. This information will let me know if the malware is packed or obfuscated at all. I was introduced to a nice tool for this called, CFF Explorer VIII, which comes standard in the FLAREVM.

One place I like to start is in the import directory to see what .exe's or .dll's that this malware tries to use. There are two imported, `KERNEL32.dll` and `MSVCRT.dll`.

![image](https://user-images.githubusercontent.com/66766340/152454687-38cf8643-3eb8-41f1-91a3-3424f62b41ed.png)
###### File Header Info

KERNEL32.dll is used for core functions such as handling memory, files, and hardware. A closer look at MSVCRT.dll and we can find ntdll.dll, which is unusual for regular programs to import. And, it is used as an interface to the Windows kernel. I wanted to double check that the binary wasn't packed, with PEiD. Sure enough, it's detected as a C++ binary that is used as a dynamic link library.

![image](https://user-images.githubusercontent.com/66766340/152456233-8c3edbfa-7107-4589-b90a-e4adac35fa81.png)
###### PEiD Info

From here, I loaded the sample into a new Ghidra project and started poking around to gather basic information about the binary. I tend to like starting by taking a look at the functions, and in this instance, the one that pokes out is called `PlayGame()`.

![image](https://user-images.githubusercontent.com/66766340/152134039-51bc9b4d-5f93-45e8-ba7d-3d88f3ff2859.png)
###### `PlayGame()` Function

A peek at its decompilation and we're greeted by a call to `sprintf`, meaning it begins by writing to a string. Its parameters are already pretty clear that it forms a path. I noted it as a pre-comment: `C:\WINDOWS\mssecsvc.exe`. I also decided to rename the string it was writing to as `mssecsvc_path`.

![image](https://user-images.githubusercontent.com/66766340/152135700-5f49524f-0737-41a4-a207-a2ce2850e2a9.png)
###### Decompilation of `PlayGame()`

Out of curiosity, I decided to search up the name of this executable to see if it was a common Microsoft binary, or if it was a malicious one trying to hide in plain sight. Just to confirm that I am working with WannaCry, `mssecsvc.exe` is a known, common, [host-based signature]. 

Pressing further into this function, there is a call to `FUN_10001016()`. A quick glance at its decompilation and we can see that it's making use of the windows api to do some resource handling. It starts by trying to locate a resource, specifically one by the handle of `0x65` or `101` in decimal. I renamed this variable `rsrc_101_handle`. It then loads it into another variable, which I dubbed `rsrc_101_data` and locks it into another variable, `rsrc_101_ptr`. Lastly, it gets the size and stores it in my `rsrc_101_size` variable, and it writes to the `mssecsvc_path` we found in the previous function.

Since the function is all about resource handling, it was fitting to rename it to `write_101_to_mssecsvc()`. It also returned int values, so I changed the function signature to ensure that it was of the proper form in the decompilation.

![image](https://user-images.githubusercontent.com/66766340/152451220-96d50783-ac11-4ac5-8922-940b2731ded6.png)
###### Decompilation of `write_101_to_mssecsvc()`

Onto the next function that gets called by `PlayGame()`, `FUN_100010ab()`. Within the declared variables, there are two structs, `_STATUPINFOA` and `_PROCESS_INFORMATION`, I renamed the local variables accordingly to `startup_info` and `process_type`. These two structs are used to create processes. I took the liberty to research exactly what the initialized flags do and commented the function above each line. Seeing that this function creates a process from the `mssecsvc_path`, it was fitting to rename it to `createBadProcess()`
> `CreateProcessA()` [documentation]

![image](https://user-images.githubusercontent.com/66766340/152450092-c8f8c213-acb1-402a-b1fa-8b72ac505540.png)
###### Decompilation of `createBadProcess()`

With this information, hopping back into `PlayGame()`, we can see that it is the function responsible for unpacking the rest of the malware and running it as a process on the host machine. This technique of creating its own process is a bit of a loud tactic. If we opened up the task manager, we would be able to see it running and just kill it from there.
> A more sophisticated technique would be to attach itself to a running process. It would do so by locating a running process, allocating memory, and injecting code into it. This can be referred to as "Process Injection".  

![image](https://user-images.githubusercontent.com/66766340/152451395-425a011a-1448-4fe6-81d6-54498124d5ae.png)
###### `PlayGame()` Revisited

Essentially, this initial binary hides the rest of the malware by storing an embedded program into its resources, which it later calls upon to continue execution. This is more likely where we will see the main functionality of WannaCry. We know for sure that it looked for a resource called 101. So, I deployed Resource Hacker on the binary to find resource 101 stored in a folder called "W". From here, I extracted the resource in to a separate .bin file, W101.bin. 

### Recap - Loading Via Process Creation:

Just to solidify what we've observed so far, WannaCry begins stealthily by writing one of the binary's resources into an executable file, and executing it as a process.

![image](https://user-images.githubusercontent.com/66766340/153566969-82ea565e-d7b6-4eb0-b6a7-669a2eb84eb0.png)
###### Code execution flow chart

---

## mssecsvc.exe Analysis - Static: <a name="mssecsvc.exe-static"></a>

After extracting the resource, I took a look at this binary via CFF Explorer and saw that we will have to dig further to gain more intel on it.

![image](https://user-images.githubusercontent.com/66766340/153568475-d8fd1360-5731-4dd8-ba99-a8705e4a7587.png)
###### CFF Explorer on W101

Although, we can't get the resource to be detected as a PE file, pressing on with Ghidra almost says otherwise. There are no imports, which is odd, and a bunch of functions. A lot more than the binary that created the process. 

After finding it strange that there are no imports detected by the other utilities, I looked at the strings of the binary and it became clear that this binary contains PE file format information along with various imports and possibly more exports.

![image](https://user-images.githubusercontent.com/66766340/153569755-fe3b82a8-bea6-4757-b71e-0627ca8dd902.png)
###### W101 Strings

Based on that snippet alone, it's apparent that this binary could have a lot of power with the ability to write to files, create them, launch processes, and more. However, before moving forward or resorting to dynamic analysis of `launcher.dll` to extract the file, we can take a closer look at the extracted resource and see if we can resolve the file format issue statically.

### Fixing the File Format

Opening the extracted resource with a hex editor, we can see that it's just a couple of bytes that break our desired format. PE files begin with an `IMAGE_DOS_SIGNATURE` at offset 0. This can be identified as ascii `MZ` or in hex, `4D 5A`. Here, we can see that offset 0 has a couple of bytes, before reaching the IMAGE_DOS_SIGNATURE. 

![image](https://user-images.githubusercontent.com/66766340/156541450-38644ee2-5463-4462-8d92-976dadf99489.png)
###### Unwanted bytes

Simply deleting these bytes and saving the rest of the hexdump allows us to export it in proper PE format. Thanks to the analysis conducted earlier, it is a safe bet to name it `mssecsvc.exe`.

### WinMain() for mssecsvc.exe

Hopping back into Ghidra to take a look at its disassembly and decompilation, we find a defined entry point, which is some standard code that leads to a `WinMain()` call. Within the WinMain call, the malware continues execution.

Past all of the local variables, we see what appears to be some string manipulation. I believe Ghidra failed to resolve a string, so we will shelf that for now, but we can still see what it does with the data. Upon fixing the variable names and types, the function becomes a bit more clear as to what it's doing with the data.

![image](https://user-images.githubusercontent.com/66766340/167205679-d64863f1-bd0e-4e88-875c-a48dc2cc09ba.png)
###### string functionality

It appears to be copying the string into a buffer. My name choice is derived from the fact that after this string operation, there are calls to `InternetOpenA()` and `InternetOpenUrlA()`. We may be able to resolve this specific url with dynamic analysis; however, this is confirmation that the malware now has networking functionality and takes advantage of it. 

To get a better grip on it, we must edit the function signature to reflect the parameters and return type of the Windows API calls. One issue I ran into is that Ghidra cannot resolve the HINTERNET return type, but it's simply a handle to be passed to subsequent api calls as per the convention of using this module. If the operation is successful, it will return the handle to be passed, else it returns a NULL.

![image](https://user-images.githubusercontent.com/66766340/167208575-9dcbbf34-c301-43c1-a218-e767339284c1.png)
###### Internet functionality

Executing the `InternetCloseHandle()` calls, it will either return a boolean true or false. After these calls, there is another custom function, which we will press into next.

I've dubbed this one `execution_handling()` as we can see that the first thing it does is call `GetModuleFileNameA()` with the path to itself, and checks the argument count. If it is ran with no arguments, it calls another function, which I've dubbed `taskche_init()`, and returns.
> we will press into `taskche_init()` later on.

![image](https://user-images.githubusercontent.com/66766340/167210458-61b03861-f4ff-4794-80df-cc9c199a697d.png)
###### argc checking for execution flow

### Execution With Command Line Args

If `mssecsvc.exe` was called with an argument, it will skip over `taskche_init()` and open a service called `mssecsvc2.0`. If successful, it will run another function that can modify the service configs.

![image](https://user-images.githubusercontent.com/66766340/167212120-559f7eb5-5b9e-4fae-9229-d03805d24e95.png)
###### launching malicious service from mssecsvc.exe

The function which I've dubbed `change_service_configs()` accepts the `SC_HANDLE` and an int value as parameters. Ghidra failed to resolve a bit more data, but the call to `ChangeServiceConfig2A()` may tip us off as to what data may be contained within the `SERVICE_FAILURE_ACTIONS` struct. 

Calling this module, especially with its second paramater as 2, is what refers to the actions being taken if the service fails. It mostly pertains to rebooting, and reset periods.

![image](https://user-images.githubusercontent.com/66766340/167212606-1c2de6cf-0317-48b3-83d9-b750de9eb2c6.png)
###### change_service_configs()

Upon returning from this function, the malware will initialize members of the `SERVICE_TABLE_ENTRYA` struct with the lpServiceName of `mssecsvc2.0` and an lpServiceProc pointing to a label, which we will investigate further.

![image](https://user-images.githubusercontent.com/66766340/167215352-c003ea98-a8c4-4dad-8ed2-02321de8cd5f.png)
###### Service Name & Service Procedure Pointer

Within the lpServiceProc label, the malware begins to get into some threading. With the call to `RegisterServiceCtrlHandlerA()`, it passes the service name, along with another `LPHANDLER_FUNCTION` specified by another label. 

![image](https://user-images.githubusercontent.com/66766340/167216554-4a34c262-7f09-471e-849d-3f08dbd74daa.png)
###### More service procedures

Hopping into this label, there is a switch-case control flow structure that defines some data based on the passed parameter. It will modify regions of data with new integers/flags and set the service status. Shelfing that for now, back in the caller, it will check on the service status, further modify regions of data and re-set the service status. After doing so, there is another defined function that gets into threading. I've dubbed it `parallel_processing()`.

![image](https://user-images.githubusercontent.com/66766340/167217407-0b465c99-7a60-4b63-95aa-360beadc985b.png)
###### More service handling and Threading

The first function called in `parallel_processing()` calls another process called `WSAStartup()` with the parameters 0x202 for wVersionRequired and a pointer to another struct that contains socket information for networking. 

If it's unsuccessful, it will return from this function back to the caller, but if it is, it will call a function, which I've dubbed `crypto_provider()`.

![image](https://user-images.githubusercontent.com/66766340/167219780-985dee69-229f-4a37-9821-c2e56bb946fe.png)
###### custom wsa_startup() -> crypto_provider()

Essentially, `crypto_provider()` attempts to call `CryptAcquireContextA()` twice in order to define or get a cryptographic service handle, which is most likely to be used to encrypt files for the ransomware component of the malware. 

It then calls `InitializeCriticalSection` to set up a critical section that the threads will be accessing. One interesting thing to note is that there is a string defined as `"Microsoft Base Cryptographic Provider v1.0"` passed to `CryptAcquireContextA()`. This cryptographic service provider defaults to RSA for public key handling and data encryption. 

![image](https://user-images.githubusercontent.com/66766340/167220333-de6d09ee-9016-40df-81e1-c60cf55604a0.png)
###### crypto_provider()

Back to the caller, `wsa_startup()`, it continues execution with another defined function that calls some file handling modules. 

### Further Unpacking

Similarly to the analysis of the `launcher.dll`, we will continue to peel back the layers of this piece of malware. With `mssecsvc.exe` loaded into a new Ghidra project, we can take a peek at the functions and find another one that's similar to `PlayGame()` from the other binary. I've opted to re-name it to `create_mssecsvc_service()` as it takes advantage of more Windows API calls to create this malicious service. 

![image](https://user-images.githubusercontent.com/66766340/166123444-99c965e8-732a-45af-b81d-071344749543.png)
###### `create_mssecsvc_service()`

One interesting to note is that creates another string with `sprintf()`, and it defines it with the path to an executable and runs it with what appears to be command line arguments, specifically `-m security`. The flags passed to `CreateServiceA()` also ensure that it has full access, runs as its own process, and runs on start-up.

A couple of host-based indicators would include the strings that define the service name and display name:
```
"mssecsvc2.0"
"Microsoft Security Center (2.0)"
```
###### Host-based indicator strings

From here, there is another function that does a handful of things. Taking advantage of the Windows API, it creates and writes files, and also creates another process. I've named it `write_taskche_and_qeriuwjhrf()`.

It begins by defining some pointers to the functions from `kernel32.dll`. We can rename and re-type these variables that store the pointers to it's appropriate corresponding function. This makes it easier to follow in the disassembly and decompilation as we know which function get's called in which order.

![image](https://user-images.githubusercontent.com/66766340/166123956-6e287b31-2076-4813-97cf-8e1e733cc2a9.png)
###### Pointers to Windows API functions

Pushing a bit further into this function, it also locates resource 1831 and writes it to `taskche.exe`. It then defines a path for `taskche.exe` as well as one for `qeriuwjhrf`. Upon first glance at the `sprintf()` calls, Ghidra's decompilation does not compensate for a differing amount of passed arguments. So, fixing this by editing the function signature produces a cleaner output in the decompilation, revealing the format string being written.
> another way to check is to see the arguments pushed onto the stack in the disassembly

![image](https://user-images.githubusercontent.com/66766340/166124266-468a609b-3c3f-4178-8bca-b047143c1409.png)
###### `sprintf()` args in cdecl calling convention

![image](https://user-images.githubusercontent.com/66766340/166124100-273c7ba2-5ae6-4437-b124-730968e60f64.png)
###### `taskche.exe` path & `qeriuwjhrf` path

These serve as more host-based indicator strings:
```
"C:\Windows\taskche.exe"
"C:\Windows\qeriuwjhrf"
```

The malware then tries to move `taskche.exe` to the path specified for `qeriuwjhrf`. My theory as to why this process occurs is because the malware may be checking to see if it is the first time it is being ran on the system. The indication of this would be the existence of the `queriuwjhrf` path. Seeing that it is programmed to accept command line inputs, there may be other flags that alter its execution in subsequent instances. 

After this `MoveFileExA()` call, it begins to set up the parameters to call `CreateProcessA()` with some obscure-looking string modifications. This may be due to an inaccurate decompilation/ disassembly, so I decided to shelf the analysis, and simply accept that it calls `CreateProcessA()` with the file handle that points to `taskche.exe`.

![image](https://user-images.githubusercontent.com/66766340/166895138-6313b7d8-180d-4341-b817-42e9347fb3a9.png)
###### call to `CreateProcessA` from `taskche.exe`

Going back over the disassembly of `write_taskche_and_qeriuwjhrf()`, I investigated a local variable stored on the stack at location `[ebp-0x103]`. Its cross-reference pointed to what appears to be the return address of the previous function, which looks to be the `main()` of `taskche.exe`.

![image](https://user-images.githubusercontent.com/66766340/166895734-0bad457b-6cb3-4f85-a70f-44d209d93d60.png)
###### `taskche.exe`'s main()

With this, let's once again re-cap with the execution of `mssecsvc.exe`. It begins by creating a service from the executable, running it with the arguments of `-m security`. It then writes its resource 1831 to another executable called taskche.exe and moves this file to `C:\Windows\queriuwjhrf`, has some string modifications on `taskche.exe`'s path, and subsequently creates another process from it.

![image](https://user-images.githubusercontent.com/66766340/166897671-3cb596f6-f585-4196-a9d9-14dd1a88b2e5.png)
###### `mssecsvc.exe`'s execution re-cap

With our static analysis coming to somewhat of a halt at this point, we can further unpack the malware and dive into more of its functionality. At this point, it has established itself as a malicious service that has full system access and runs on startup.

---

## taskche.exe Analysis - Static <a name="taskche.exe-static"></a>

Since we know that it uses its resource 1831 to write `taskche.exe`, this will contain the data we need to analyze the binary. Once again deploying Resource Hacker, we can see that the offset 0 contains the `IMAGE_DOS_SIGNATURE` of 4D 5A in hex. This indicates that we are in fact dealing with another PE executable file. We can then save this resource as `taskche.exe` and begin further analysis of the malware.

![image](https://user-images.githubusercontent.com/66766340/166899513-499cd2c0-5af2-4ca7-acd1-79abcfdd648d.png)
###### R resource - taskche.exe

With the analysis so far, we've identified a theme as to how it unpacks itself. It's not very sophisticated in this manner as all it has been doing so far is writing binaries from its resource section into new .exe's to be dropped. Curious to see if there is more to come, we can take a peek at its resource section using CFF Explorer VIII.

Surely enough, we can see that there is more data stored here, likely to be more files or malware that will get installed and ran. 

![image](https://user-images.githubusercontent.com/66766340/167108323-7e79ed20-09d2-4f48-8b52-234c9730ec52.png)
###### taskche.exe's resources

Furthermore, we can also take a look at the imports of this executable, and find out that there are tell-tale signs that the malware will likely have network functionality within this PE. The indications of this include WS2_32.dll, iphlpapi.dll, and WININET.dll. The presence of ADVAPI32.dll also opens up an avenue that may indicate that the malware will set up a persistance mechanism on the system. This can typically be done by editing registry keys.

![image](https://user-images.githubusercontent.com/66766340/167109807-917840d5-c27b-4232-942c-d4ebea0518e6.png)
###### taskche.exe's imports

Pressing on to its disassembly in Ghidra, we are greeted by an entry point in the functions. This will serve as the basis of our analysis for `taskche.exe`. 

[here]: /0xcjg-HoneyPot/
[FLAREVM]: https://github.com/mandiant/flare-vm
[Ghidra]: https://ghidra-sre.org/
[host-based signature]: https://www.2-spyware.com/file-mssecsvc-exe.html
[documentation]: https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa
[CFF Explorer VIII]: https://ntcore.com/?page_id=388
[PEiD]: https://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml
[Resource Hacker]: https://resource-hacker.en.softonic.com/
[Detect It Easy]: https://github.com/horsicq/Detect-It-Easy
[HxD]: https://mh-nexus.de/en/hxd/
