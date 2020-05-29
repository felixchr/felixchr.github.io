---
title: Use VirtualBox to Setup Linux Dev Environment on Windows 10
categories: [tips, operations]
tags: [development, virtualbox, windows, linux, environment]
description: Use VirtualBox to Setup Linux Dev Environment on Windows 10
---

# The Background

I need to develop Python and Bash scripts for our applications which running on Linux but my workstation is a Windows 10 laptop. I would like to test immediately on my local machine instead of test on remote servers, in spite of that I have Jenkins set up for testing.
My company's VPN has very tough network restrictions. It force all outbound traffic goes to VPN gateway. That is to say, I cannot reach to any other IP addresses other than 127.0.0.1 locally.


# Install and Configure VirtualBox

I would not elaborate how to install VirtualBox on Windows 10. It is straightforward. Just download and install.

Guest OS installation is not my topic as well. 


## Network Configuration

### Network Types

**Bridge**: act like another computer in the same network. It can get same IP addresses with your computer.

**NAT**: Use your computer as gateway. Both your computer and VM will have IP in same subnet

**Host-only**: accessible only from Host (I'm not if this is correct. Maybe updated later)

### Use NAT Network

Due to the restriction of my company's VPN, I had to use **NAT Network** for my guest OS.

In *Network* tab of **Settings** of VM, set the adapter to attach to **NAT Network**.

The default network is 10.0.2.0/24. 10.0.2.1 and 10.0.2.2 are occupied. You'd better to use static instead of DHCP because we'll need to set Port Forwarding. You won't want to change the Port Forwarding frequently if you have multiple VMs.

### Port Forwarding

Port Forwarding is globally. It's in File -> Preference -> Network -> NatNetwork. Click *Edit NatNetwork* and *Port Forwarding*. 

## Share Folder Between Host and Guest

### Use Samba

This was tested with VMWare Player and CentOS 6. After I switched to VirtualBox, I didn't try it as my VPN restricts me to access any IP other than 127.0.0.1. I put it here as this may help your situation.

1. Install samba

{% highlight shell %}
$ sudo yum -y install samba
{% endhighlight %}

2. Configure Samba server

   The configuration file is `/etc/samba/smb.conf`

{% highlight shell %}
[global]
    workgroup = MYGROUP
    server string = Samba Server Version %v
    log file = /var/log/samba/log.%m
    max log size = 50
    security = user
    passdb backend = tdbsam
    load printers = no
    cups options = raw
[homes]
    comment = Home Directories
    browseable = no
    writable = yes
{% endhighlight %}

3. configure password

{% highlight shell %}
[root@diamond samba]# smbpasswd -a felixc
New SMB password:
Retype new SMB password:
Added user felixc.
{% endhighlight %}

4. Mount on Windows

   On windows, run: \\\<IP-ADDR\>Use "local\usernmae" and password to login.

### Use VirtualBox shared folder

*This is tested with VirtualBox 6.1 and CentOS 7.* This is my current test environment.

1. install guest addons
   1.1 Devices -> Insert Guest Additions CD images
   
   1.2 update kernel on Guest OS
   
{% highlight shell %}
$ sudo yum update kernel -y
$ sudo yum install kernel-headers kernel-devel gcc make -y
$ sudo init 6
{% endhighlight %}
   
   1.3. mount cdrom and install the additions to OS.
   
{% highlight shell %}
# check the cdrom device
$ blkid
$ sudo mount /dev/sr0 /a
$ cd /a
$ sudo ./VBoxLinuxAdditions.run
{% endhighlight %}
   
2. set up share folder in VirtualBox on host

   Use menu VM -> Settings -> Shared Folders, 

   <picture here>

3. Add user to **vboxfs** group in order to gain permission to the shared folder

{% highlight shell %}
$ sudo usermod -G vboxfs felixc
{% endhighlight %}

4. mount on guest OS
{% highlight shell %}
$ VBoxControl sharedfolder list
$ mount.vboxsf share /app
{% endhighlight %}

   

## Development Environment

### Use VS Code Remote Development Plugin

VS Code is awesome! The Remote Development plugin works good with Linux environment.

I installed following plugins on my host for remote development:

1. Remote Development
2. Remote - SSH
3. Remote - SSH: Editing Configuration Files
4. Remote - WSL

Following plugins are installed on develop server:

1. Python
2. GitLens
3. Prettier

VS Code supports **pipenv** very well. It loads python virtualenv automatically.

### Use Git Bash for Windows

Git bash is a very good way to provide Bash environment and GIT. It would be best alternative for Windows Command Line.

# Issues and Fixes

1. rc=-5640

This is due to some antivirus software. So far I haven't found any fix for it. But you can try to few times to start your VM. Sometimes I tried more than ten times in order to start my VM.

Someone says increasing the memory size would increase the starting success rate. I set my VM to 2GB RAM and it seems better now.

2. Cannot PING gateway under NAT Network

This happens very often. To fix it, use VBoxManage to restart the network. I wrote a bash script and put it into PATH of my Git Bash. Each time when I have this issue I run it.

*/c/fc/local/bin/vboxnat-restart.sh*

{% highlight shell %}
#!/bin/bash
echo "Stopping..."
/c/Program\ Files/Oracle/VirtualBox/VBoxManage.exe natnetwork stop --netname NatNetwork
sleep 2
echo "Starting..."
/c/Program\ Files/Oracle/VirtualBox/VBoxManage.exe natnetwork start --netname NatNetwork
echo "Done"

{% endhighlight %}