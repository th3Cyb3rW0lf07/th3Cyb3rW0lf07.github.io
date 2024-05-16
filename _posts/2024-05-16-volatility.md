---
layout: post
title: Volatility - Memory forensics made simple
subtitle: Volatility
categories: Walkthrough
tags: [forensics, walkthrough]
---

### Oi!! Another writeup, another challenge. Need to do more of these 😮‍💨.
### Welp, in this writeup we'll be looking at Volatitlity, my preferred tool for memory analysis

[Volatility](https://www.volatilityfoundation.org/) is an open-source memory forensics framework used in Malware analysis and Incident Response. This framework is CLI-based and is programmed in Python. It's supported on Windows, Linux, and MacOS

When security breaches occur on endpoints, there usually is a footprint left by the perpetrator. Many attackers seem successful clearing any traces they may have left behind by clearing any system logs, using LOLBins, and clearing any network trace they may have left behind. This is where memory forensics comes in.

Memory forensics involves analysis of the volatile data found on endpoints, i.e data found in the RAM. Most times after a cyber attack, the first reaction would be to power off the machine to stop certain malicious actions from taking place, but most of the activity carried out on the machine will be running in memory. Tools like [FTK Imager](https://www.exterro.com/ftk-imager) can be used to extract the memory dump for later analysis.

### What does Volatilty dig out?
Volatility has different in-built plugins that can be used to sift through the data in any memory dump. You can scan for pretty much anything ranging from drivers, to dlls, even listing processes that could have injected malicious code in them. Let's see how Volatility can be put to use. I'll be using a memory dump from the [Memory Analysis](https://app.letsdefend.io/challenge/memory-analysis) challenge on LetsDefend and we'll answer the questions attached to that challenge.

### Time to get into action
For this analysis, I'll be using REMnux, the virtual isolated environment for analysis. The volatility tool is already installed there and when you run `vol.py -h` you'll see the list of options you can run with it.

We'll start by getting little information on the victim endpoint
`vol.py -f dump.mem windows.info`
![Screenshot from 2023-10-16 12-39-56](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/fae5ff6d-a39a-4c77-b5ae-a99bfd65676d)

And we have the answer to our first question on the challenge!
![Screenshot from 2023-10-16 12-46-10](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/e59856ec-970e-432f-9024-c7861e95839d)

Next, we look at the processes
`vol.py -f dump.mem windows.psscan` then `vol.py -f dump.mem windows.pstree`
![Screenshot from 2023-10-16 12-56-58](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/7458aba2-5959-4de2-afdf-786b548f9266)

When looking at the results of the process tree, notice there are 2 processes named lsass.exe.This should not be, which means one of the processes is malicious. Pay attention to the PID and Parent PID. The malicious process has a PID 7592 and a Parent PID 3996 which points to explorer.exe. Definitely malicious!!
We now have the answer to our second question!!
![Screenshot from 2023-10-16 12-58-35](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/3f6aa5d7-f4d1-45f7-b1c9-b0de1bc34b72)

Let's analyze the malicious process more. `vol.py -f dump.mem windows.pslist --pid 7592 --dump` This way we can anlyse the binary that's involved with this process. Checking the hashsum and running it through virustotal, we get this
![Screenshot from 2023-10-16 13-09-13](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/7029e5fb-6b21-45b3-80df-289438b1e7b6)

We see that the binary is malicious and the actual name is winpeas.exe, a tool used for privilege escalation on Windows hosts.
And there we go, 3rd question answered!!!
![Screenshot from 2023-10-16 13-10-22](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/40d20d86-f2c0-42af-8c61-026117039d4d)

To answer the next question, we need to investigate the sessions that were opened on the host. Since we already know that the PID for our malicious process is 7592, we can run `vol.py -f dump.mem windows.sessions | grep 7592`
![Screenshot from 2023-10-16 13-15-38](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/9ff99705-0c57-40dd-a9b3-5223f250b01d)

There we go! We have the compromised account and the answer to question 4!!!!
![Screenshot from 2023-10-16 13-16-26](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/24109abd-1670-417b-aede-38fd212cada4)

Now, let's answer our final question! We need to get the password of the compromised user account. `vol.py -f dump.mem windows.hashdump`
![Screenshot from 2023-10-16 13-33-53](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/49454576-299c-4a10-9e31-a038398d2aa5)

Take the nthash and run it through [Crackstation](crackstation.net/)
![Screenshot from 2023-10-16 13-21-58](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/9afa78a7-a8b5-4b70-a01f-58cee09cb48f)

There's our password and the answer to the final question!
![Screenshot from 2023-10-16 13-20-47](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/e91ed2c8-5efc-4779-9c82-318544233fc1)

### And there you have it h4x0r, that's a little guide on volatility and what it can do. If you want to practice using the tool, you can head over to this [room](https://tryhackme.com/room/memoryforensics) on TryHackMe and do your ting.
### Till next time, happy hacking :)

<script src="https://tryhackme.com/badge/894204"></script>
