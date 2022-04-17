---
title: "0xcjg HoneyPot"
layout: post
---

# Security Research: Basic Malware HoneyPot Project

## Overview:

This project is what's known as a [honeypot]. Essentially, it is a fake server used to bait for cyber attacks. We intentionally leave it vulnerable in order to gain an understanding of how one conducts their attacks. In my particular honeypot, I have it set up specifically to try to detect and collect malware samples for further research and reverse engineering practice.
> Services\ Frameworks used:
> * [Google Cloud Platform] 
> * [Modern Honey Network]
> * [Dionaea]
> * [FLAREVM]

## Setup:

Due to my intentions with the honeypot, I did not see the need to beef it up and make it look particularly enticing for an attacker to try to infiltrate and escalate privileges. Mine was to serve as more of a monitor of internet traffic to see what worms are floating around and propogating systems.

In essence, I hosted two virtual servers on the Google Cloud Platform. One for monitoring ingress traffic and another to serve as the honeypot itself. There were virtually no firewall rules on the honeypot, so most traffic would be free range. Both were simple ubuntu servers with MHN and Dionaea configured. When setup correctly, the MHN framework allows us to `http` into the admin server to view the user interface in which we can see some analytics about the incoming attacks.
> There are even some neat features integrated such as HoneyMap, which maps out the incoming attacks in real time.

With MHN serving as a sort of SIEM, Dionaea is the service that will be monitoring and capturing malware samples. It does so by analyzing incoming traffic, running shellcode, and extracting malicious files as they come in. It saves the binaries in a directory that we can export into our malware analysis environment.

### Admin UI
![image](https://user-images.githubusercontent.com/66766340/150939244-6b87e92f-8efe-4327-a853-8e4b5c22f04e.png)

### HoneyMap
![HoneyMap](/assets/honeymap.gif)

From here we have a clean UI to view some interesting stats; however, my end goal with the honeypot was to collect malware samples. To do so, in the honeypot server, I used `screen` to run a continuous `tcpdump` that would write to a `.pcap` file. After letting it run and collect some traffic, I `scp`ed the pcap and viewed the data with `wireshark` in my FLAREVM. We also have the option to export the traffic logs into a `.json` from the honeypot.

### Continuous `tcpdump` as a Detached Process
![tcpDump](/assets/tcpdump.gif)

Thanks to `wireshark`, we can actually download the intercepted data. At this point, we can have Dionaea collect samples that it can recognize, and we'll have the traffic to export any interesting binary data that comes in as well. This feature is what allows us to pull any binaries or malicious traffic containing code into our malware analysis environment. We can hash files to cross-check samples with databases such as [VirusTotal], check to see if any antivirus scans (like malwarebytes, bitdefender, etc.) will pick up on our samples, and even get into reverse engineering live malware.

## HoneyPot Findings:

### 23 JAN 2022 - 26 JAN 2022

These findings are accumulated after just three days of up-time. Out of curiosity, I pulled the pcap that the tcpdump has been writing to and found a pretty staggering amount of network traffic. Most of the traffic consisted of port-scans; however, it was relatively easy to see some areas where intrusion attempts had been made.

### Attempted FTP Exploits
![image](https://user-images.githubusercontent.com/66766340/151256295-151edc01-b39d-446e-8fb7-38d7dff7eaee.png)

This is just one of the [many examples] of FTP-based intrusions that I was able to see in my honeypot's network traffic. In this attempt, the attacker was trying to poke around in the filesystem, which was just empty. Neat to see, but it is not quite what I'm looking for. However, I will be sure to include more things I find interesting.

### SSH Brute Forcing
```
Jan 23 20:45:10 honeypot-1 sshguard[788]: Blocking 31.7.57.130 for 1680 secs (4 attacks in 38 secs, after 2 abuses over 14978 secs)
Jan 23 22:01:14 honeypot-1 sshguard[788]: Blocking 104.248.43.70 for 840 secs (4 attacks in 264 secs, after 1 abuses over 264 secs)
Jan 23 23:08:09 honeypot-1 sshguard[788]: Blocking 31.7.57.130 for 3360 secs (4 attacks in 38 secs, after 3 abuses over 23557 secs)
Jan 24 06:43:55 honeypot-1 sshguard[788]: Blocking 104.248.141.51 for 840 secs (4 attacks in 61 secs, after 1 abuses over 61 secs)
Jan 24 10:54:50 honeypot-1 sshguard[788]: Blocking 104.248.141.51 for 1680 secs (4 attacks in 63 secs, after 2 abuses over 15116 secs)
Jan 24 22:58:03 honeypot-1 sshguard[788]: Blocking 36.110.228.254 for 840 secs (4 attacks in 17 secs, after 1 abuses over 17 secs)
Jan 24 23:02:31 honeypot-1 sshguard[788]: Blocking 171.22.76.74 for 840 secs (4 attacks in 17 secs, after 1 abuses over 17 secs)
Jan 25 00:45:25 honeypot-1 sshguard[788]: Blocking 155.93.160.203 for 840 secs (4 attacks in 85 secs, after 1 abuses over 85 secs)
Jan 25 07:41:23 honeypot-1 sshguard[788]: Blocking 31.7.57.130 for 6720 secs (4 attacks in 38 secs, after 4 abuses over 140751 secs)
Jan 26 08:25:46 honeypot-1 sshguard[788]: Blocking 207.180.238.8 for 840 secs (4 attacks in 90 secs, after 1 abuses over 90 secs)
Jan 26 09:23:51 honeypot-1 sshguard[788]: Blocking 31.7.57.130 for 13440 secs (4 attacks in 39 secs, after 5 abuses over 233299 secs)
Jan 26 13:55:12 honeypot-1 sshguard[788]: Blocking 120.224.50.233 for 840 secs (4 attacks in 32 secs, after 1 abuses over 32 secs)
Jan 26 17:08:12 honeypot-1 sshguard[788]: Blocking 121.5.9.52 for 840 secs (4 attacks in 280 secs, after 1 abuses over 280 secs)
```

These alerts are from the auth.log indicating attempts of ssh brute-forcing. After only three days of being up, my honeypot was already getting tested by loud tactics. By what sshguard could catch, there were 12 definite attempts; however, looking at the full log, there are actually a handful of persistent ip's that test periodically to avoid getting blocked by sshguard. 

### URL Manipulation
![image](https://user-images.githubusercontent.com/66766340/151514217-6aa829d3-796a-4493-80bf-76924a48fb81.png)

It seems the attackers that send these `GET` requests for the `/.env` file are looking to scrape data from that config file in order to run a [successful SMTP attack] on web apps that have debug mode enabled.

A useful indication of compromise is if they also try to POST with these [bytes]:
```
0000   42 01 0a 8a 00 03 42 01 0a 8a 00 01 08 00 45 00   B.....B.......E.
0010   00 3c 4f f4 40 00 76 06 20 38 9f f2 ea 10 0a 8a   .<O.@.v. 8......
0020   00 03 24 e0 00 50 64 94 42 c1 00 82 6d f0 50 18   ..$..Pd.B...m.P.
0030   02 03 9e d5 00 00 30 78 25 35 42 25 35 44 3d 61   ......0x%5B%5D=a
0040   6e 64 72 6f 78 67 68 30 73 74                     ndroxgh0st
```
Which are also pulled directly from [my pcap].

There are also a handful of `POST` requests that try to upload scripts to directories within the honeypot; however, since they don't exist, the requests failed. Also, any attempts that contained potential malware urls lead to dead ends so far. Other than the `GET /.env` requests, there is similar traffic also looking for config files in order to gain some good footholds on actual websites or cloud servers which may be compromised in that manner.

---

### 27 JAN 2022 - 30 JAN 2022

### Collecting Malware Samples

After a few more days of letting it run, I checked on the findings via the MHN ui and saw a ton of traffic taking advantage of SMB. Exploiting SMB is very popular with worms and can be used to easily spread malware. Thanks to Dionea being configured to actually capture and save the files, I've successfully collected many samples, ready to be analyzed.

Crosschecking with VirusTotal yields us results, and these binaries are, in fact, live malware.

### VirusTotal Results
![image](https://user-images.githubusercontent.com/66766340/151722579-cbd391ca-5a1c-42c3-b5ee-cc0407e240a8.png)
> This one specifically is [WannaCry Ransomware].

Dionaea was also capable enough to capture [droppers], instead of full programs as well, leaving a lot to explore within the collected samples after only a week of being up. 

In total, I was able to collect 105 samples. Some of them are duplicates from the same worms, and others are unique. With all of the samples in the same directory on my malware environment, I decided to conduct an antivirus scan on it to see how many it will actually detect. For my experiment, I chose to configure MalwareBytes to run a scan on the directory.

### MalwareBytes Results on Sample Directory
![image](https://user-images.githubusercontent.com/66766340/151770470-c7c899b8-69a9-403c-aa06-2392743c45ec.png)

Suprisingly, MalwareBytes could only recognize 79% of the malware, and would leave 21% of the malware on the system. For comparison, I ran the same scan, but with Bitdefender instead. Bitdefender caught even less than MalwareBytes, picking up 71% or the samples, leaving 29% still on the system.

### Bitdefender Results on Sample Directory
![image](https://user-images.githubusercontent.com/66766340/151780585-2fe34a1d-e2d1-4745-9280-afc492b757c2.png)

---

With both commercial antivirus softwares not picking up 100% of the malware collected from the honeypot, this serves as a good argument to there still being a lot of research to be done through the use of honeypots such as this one. Malware is constantly changing and evolving, and security research must be able to keep up in order to continue defending against it.

[honeypot]: https://blog.malwarebytes.com/101/2021/05/what-is-a-honeypot-how-they-are-used-in-cybersecurity/
[Modern Honey Network]: https://github.com/pwnlandia/mhn
[Google Cloud Platform]: https://cloud.google.com/free/
[Dionaea]: https://dionaea.readthedocs.io/en/latest/
[FLAREVM]: https://github.com/mandiant/flare-vm
[VirusTotal]: https://www.virustotal.com/gui/home/url
[many examples]: https://github.com/colton-gabertan/xcjg-honeypot/blob/Index/honeypotFindings/ftp.pcap
[successful SMTP attack]: https://thedfirreport.com/2021/02/28/laravel-debug-leaking-secrets/
[bytes]: https://security.stackexchange.com/questions/255881/what-does-a-post-like-0x5b5d-somename-try-to-achieve
[my pcap]: https://github.com/colton-gabertan/xcjg-honeypot/blob/Index/honeypotFindings/http.pcap
[WannaCry Ransomware]: https://www.malwarebytes.com/wannacry
[droppers]: https://resources.infosecinstitute.com/topic/malware-spotlight-droppers/

