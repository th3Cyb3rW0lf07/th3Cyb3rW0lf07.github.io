---
layout: post
title: Volatility - Memory forensics made simple
subtitle: Volatility challenge
categories: Forensics
tags: [security, forensics, hacking]
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
![volatility_1](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/7d03d9dc-fe6f-4bcc-860f-b8c88dd75f6e)


And we have the answer to our first question on the challenge!
![volatility_2](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/79b3a3d3-efae-40f5-b889-3194cd28f969)

Next, we look at the processes
`vol.py -f dump.mem windows.psscan` then `vol.py -f dump.mem windows.pstree`
![volatility_3](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/0f5891d6-c2d2-443e-9ec6-f7c513ab51d8)

When looking at the results of the process tree, notice there are 2 processes named lsass.exe.This should not be, which means one of the processes is malicious. Pay attention to the PID and Parent PID. The malicious process has a PID 7592 and a Parent PID 3996 which points to explorer.exe. Definitely malicious!!
We now have the answer to our second question!!
![volatility_4](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/aaf50fe9-b2ce-4440-a08f-0aac781ccc23)

Let's analyze the malicious process more. `vol.py -f dump.mem windows.pslist --pid 7592 --dump` This way we can anlyse the binary that's involved with this process. Checking the hashsum and running it through virustotal, we get this
![vol_5](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/70c86794-3d97-437e-9255-f5c3bcf2d840)

We see that the binary is malicious and the actual name is winpeas.exe, a tool used for privilege escalation on Windows hosts.
And there we go, 3rd question answered!!!
![vol_6](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/f0ae0409-bf34-4598-bfa1-41e58a3892df)

To answer the next question, we need to investigate the sessions that were opened on the host. Since we already know that the PID for our malicious process is 7592, we can run `vol.py -f dump.mem windows.sessions | grep 7592`
![vol_7](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/a82a01c4-b6c5-4e8c-be5e-dca90c7a16fb)

There we go! We have the compromised account and the answer to question 4!!!!
![vol_8](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/4be13b13-4173-4a4e-b79c-be99af71495b)

Now, let's answer our final question! We need to get the password of the compromised user account. `vol.py -f dump.mem windows.hashdump`
![vol_9](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/0717e693-9e67-4f2a-84f5-2782dcafd408)

Take the nthash and run it through [Crackstation](crackstation.net/)
![vol_10](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/c3f9acda-d1db-4de8-bb8a-2c766bf0fcf4)

There's our password and the answer to the final question!
![vol_11](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/fde1a17e-96aa-478c-a8c2-1dafc19ac910)

### And there you have it h4x0r, that's a little guide on volatility and what it can do. If you want to practice using the tool, you can head over to this [room](https://tryhackme.com/room/memoryforensics) on TryHackMe and do your ting.
### Till next time, happy hacking :)
