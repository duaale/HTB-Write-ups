# Access Write-ups
Access is an &quot;easy&quot; difficulty machine, that highlights how machines associated with the physical security of an environment may not themselves be secure. Also highlighted is how accessible FTP/file shares can often lead to getting a foothold or lateral movement. It teaches techniques for identifying and exploiting saved credentials. 

**Enumeration**

We started scanning with nmap:

 We got 3 ports open 21, 23, 80

<p align="center">
Nmap Scan results: <br/>
<img src="https://i.imgur.com/381fTLi.png" height="80%" width="80%" alt="nmap results"/>
<br />
<br />

 
PORT 80: The site's default page, this doesn’t have much except give us an image
<p align="center">
<img src="https://i.imgur.com/DWSVRlc.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

**Directory Busting:**

I used the **dirsearch** tool but no interesting directories found!
<p align="center">
<img src="https://i.imgur.com/CJ2czS1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />


**FTP Port 21:**

I’ve checked the FTP port 21, logged in as “anonymous”. Found two directories sitting there. Looked into both and got backup.mdb in Backups directory and Access Control.zip under Engineer directory. I’ve transferred both into my local files.

<p align="center">
<img src="https://i.imgur.com/RYXh41f.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
 <img src="https://i.imgur.com/I9omKol.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

### Extracting credentials from backup.mdb

We checked the backup.mdb file and found it’s a Microsoft Access Database.

```visual-basic
file backup.mdb
```

Then we installed mdbtool to extract this database and then used to read the table names

```visual-basic
apt-get install mdbtools

mdb-tables backup.mdb
```

<p align="center">
<img src="https://i.imgur.com/VgzyoN9.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br />
<br />

 We used **While loop** to execute and read the table in **backup.mdb** file in a **Bash** script. This gives us human readable format.

<p align="center">
<img src="https://i.imgur.com/6Iuv6Td.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br />
<br />

We found an interesting table named ‘**auth-user**’ and we will use mdb-export to get the content of the table:

```visual-basic
mdb-export backup.mdb auth-user
```

![telnet password.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/a2f97c01-cd9d-48c9-8ed8-73587cb9200c/telnet_password.png)

We got some credentials:

1. backup_admin: admin
2. engineer: access4u@security

![auth_user.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/9cedb3c3-c522-4bca-8a1c-396ad43119e6/auth_user.png)

### Unzip the “Access Control.zip” File

We attempted to unzip however, it’s password protected. No password No unzip!

![no password no unzip.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/93f8240d-9eb6-4e08-a49f-bc12962ba7a5/no_password_no_unzip.png)

We know that we got some credentials from backup.mdb File. The correct password “access4u@security”.

![unzip.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/ab3e5e6d-d460-4eaf-9f49-c1091a98aaee/unzip.png)

We read the Access Control.pst with “**readpst**” tool. The **pst** extension is a Microsoft Outlook email Folder.

![cat as an mbox.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/2d3bdc06-dddc-4ea1-861b-005d39c38eb5/cat_as_an_mbox.png)

It was created another file named **Access control** with an **mdb** extension.

![password found.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/329db00a-86a8-465d-ab94-f22e3fee40a3/password_found.png)

We found an email communication with some credentials. This could be **telnet login** credentials.

![telnet login info.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/69c49015-f4b2-4750-afed-30f664450687/telnet_login_info.png)

### Telnet Login:

We **telnet** the box, and now we shell as Security!

![2024-03-02 23_59_40-kali-linux-2023.3-vmware-amd64 - VMware Workstation 17 Player (Non-commercial us.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/fef77f8e-a9a9-4e68-9fad-d66726bc7eb0/2024-03-02_23_59_40-kali-linux-2023.3-vmware-amd64_-_VMware_Workstation_17_Player_(Non-commercial_us.png)

Cat out User Flag:

![user flag.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/a22996f4-36f2-4c78-9517-af53c416bb8a/user_flag.png)

### Priv Esc:

We’ll start with enumeration the file system, users, services and more.

we tried to change the directory to Administrator but got Access denied!

![access denied.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/cecfd8fd-991c-4f5a-983b-5680105557a7/access_denied.png)

We checked that the administrator does not required any password!

```visual-basic
net user administrator
```

We used **runas** command, `runas` allows us to run commands as another user and the option `/savecred` allows us to use the command without asking for password. 

![rooted.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/6cbe88b4-69f4-48fb-8563-af0577126040/rooted.png)

Got the root flag. That‘s it!

**We rooted the Box!**

More info:

```visual-basic
mdb-sql backup.mdb  (for excel/database reading)

readpst   (for email reading)
```
