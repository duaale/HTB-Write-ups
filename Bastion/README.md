## Bastion Write-ups:

### About Bastion:

Bastion is an Easy level Windows box which contains a VHD ( Virtual Hard Disk ) image from which credentials can be extracted. After logging in, the software **MRemoteNG** is found to be installed which stores passwords insecurely, and from which credentials can be extracted. 

## Enumeration and Reconnaissance:

### NMAP:

Ended up for a few ports open.

![nmap ports open.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/f45e25ee-762a-41fc-b4db-c63e5e3c2cf0/nmap_ports_open.png)

We need more info about the ports open:

Found a guest or anonymous user can be logged in SMB.

![full nmap 1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/e9d37666-1c83-457f-bc4e-185aa528fd9c/full_nmap_1.png)

![full nmap 2.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/1f7b7ded-8b43-43ad-934b-4a27ea4f40e3/full_nmap_2.png)

### SMB Enumeration:

Backups directory is more appealing apart of default shares.

```jsx
smbclient -U 'guest' \\\\bastion.htb\\Backups
```

![2024-06-03 18_23_36-kali-linux-2023.3-vmware-amd64 - VMware Workstation 17 Player (Non-commercial us.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/e24b34fb-1e2e-4f62-bf3e-8a130745ca32/2024-06-03_18_23_36-kali-linux-2023.3-vmware-amd64_-_VMware_Workstation_17_Player_(Non-commercial_us.png)

dive in the Backups shar folder. discovered note.txt file and “WindosImageBackup”

![backup list.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/27bd9d0b-b436-4dd5-b106-1fc43f79dbb4/backup_list.png)

VHD backup files are sitting on SMB. 

After a while searching found that we need to mount the VHD file.

![whole backup files.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/c18d05dc-f359-4db2-88e2-daeb13f4c674/whole_backup_files.png)

we could use “mget *” to download the file into our local kali machine.

![mget.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/5dd84645-4741-4143-afec-d732d7e04b4c/mget.png)

### Mounting VHD files via Remote Share

we can use “mount” to mount the whole Backups files through Remote Share.

**P.S:** At this point, its better to mount while you’re logged in as Root user. This will give an interactive way to access the files.

Here is a great post on Medium platform to follow.

https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25

Here is the process:

installed the following requirements to mount the VHD files.

```jsx
apt-get install libguestfs-tools

apt-get install cifs-utils
```

Created a directory called **backups.**

Logged with the following commands with no password.

Mounting the remote shares.

```jsx
root@kali ~ mount -t cift -o user=guest //10.10.10.134//Backups bastion
```

![mounted the whole file correct.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/ddd255cd-4f69-4b78-a36b-42a301f539cb/mounted_the_whole_file_correct.png)

Mounted all the backup files, but interested the vhd files.

Created “**mounted/mnt**” directories in root user.

![vhd files.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/1ba5587c-c5df-4a10-a538-238f887ab7d7/vhd_files.png)

Now, we need to mount the vhd files only. Here is process in using “**guestmount**”

created “**bastion**” directory in root user to store the vhd files.

```jsx
guestmount -a /root/mounted/mnt/backups/WindowsImageBackup/L4mpje-PC/Backup\ 2019–02–22\ 
124351/9b9cfbc4–369e-11e9-a17c-806e6f6e6963.vhd -m /dev/sda1 -- ro 
/root/bastion/

or

guestmount —add /root/mounted/mnt/backups/WindowsImageBackup/L4mpje-PC/Backup\ 2019–02–22\ 
124351/9b9cfbc4–369e-11e9-a17c-806e6f6e6963.vhd - - inspector - - ro 
/root/bastion/ -v
```

![mounted.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/01194a33-4cce-4086-b965-dcf3620c7004/mounted.png)

This is whole disk image file format that stored the entire contents of a computer’s hard drive.

Navigating the windows.

![all mounted files.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/d4f30b10-d856-4a44-82d8-673d212ffa22/all_mounted_files.png)

![system32.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/b3d0733b-068c-417a-9c06-5d141c57ce29/system32.png)

deep diving, we identified three interesting files **“SAM” “SECURITY” “SYSTEM”.** We need to decrypt them.

Then, created a directory called “**SAM**” to copy these interesting files to make it easy our decryption process.

![copy file to SAM.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/5f14860d-99e2-46c8-9c7b-1e682edee644/copy_file_to_SAM.png)

### Decrypting vhd files:

installed mRemoteNG-Decrypt into our kali to decrypt the vhd files. 

![Decrypt.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/ab5a4f45-d8c3-4fb6-924f-cc99adfd4437/Decrypt.png)

Decrypted the vhd files and was identified the clear text users and password hashes.

decrypted in two ways:

```jsx
samdump2 SYSTEM SAM

OR

secretsdump.py -sam SAM -security SECURITY -system SYSTEM local
```

The password of L4mpje user is on a clear text password without cracked the hashes.

![samhashes.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/ddb7292d-6c83-461c-a5df-3795751bef93/samhashes.png)

or

we can crack the hashes through Hashcat.

```jsx
hashcat -m 1000 hashes.vhd /usr/share/wordlists/rockyou.txt -r
/usr/share/hashcat/rules/best64.rule --force
```

Now, we have a valid user and a password to login as an SSH.

### Shell as L4mpje user:

```jsx
ssh L4mpje@10.10.10.134
```

we logged on it as L4mpje user and now we’re in a shell.

![shell.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/f6e10f7e-2c5d-44be-9a59-24bdaf7c7037/shell.png)

Got user flag:

![user flag.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/e72a9117-9894-4837-882c-cc7a837a67cf/user_flag.png)

### Privilege Escalation:

A bit of researching, the only interesting directory in Program Files is **mRemoteNG** which is s an open source remote connections manager that supports different protocols including RDP and SSH.

![interesting file.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/5134f689-9e0c-42cb-8f3c-4270e70a88e0/interesting_file.png)

Under mRemotand there is another interesting file named as conCof.xml file which has all the password hashes for RDPs and this can be decrypted.

![xml file found.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/366e7a74-de42-4671-859b-e07cf5828452/xml_file_found.png)

Identified all users password hash including the **Administrator** to login in RDP**.**

![admin RDP hash.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/b0df938b-1d42-4fae-8bc5-35db113f7144/admin_RDP_hash.png)

Now its time to decrypt the Admin user password hash that is sitting on mRemoteNG directory.

First, we download the mRemoteNG script in our Kali machine.

https://github.com/haseebT/mRemoteNG-Decrypt

Decyrptd the admin hash.

![admin hash cracked.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/8eb0c3e9-c330-488f-b906-74a75a992add/admin_hash_cracked.png)

### Shell as an Admin user:

we’ve an administrator user and the password is decrypted. Time to login via SSH.

```jsx
ssh Administrator@10.10.10.134
```

we’re in as an Administrator.

![logged in as admin.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/e8f97a38-2933-49b8-80e6-05305f8588bb/logged_in_as_admin.png)

Got root flag.

![root flag.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/d6875430-eb14-42e7-8063-9f491f29c465/bd52a4c8-f1da-4e12-87b4-e4b8338fb10c/root_flag.png)

The Box is rooted!
