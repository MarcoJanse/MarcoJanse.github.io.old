---
layout: post
title:  "Troubleshoot Windows Time in a domain"
categories: windows activedirectory
---


> Time is of the essence my dear

## Introduction

This is something that is very true in a Windows domain environment. When the system time between servers differs too much, al sorts of problems can arise. For example, in order for Kerberos to function securely, the time difference between the participating machines needs to be less than five minutes. Components like Active Directory replication and WSUS rely heavy on correct time settings as well.

This article focusses on Windows Time configuration within a domain. For local systems, other articles apply.

## Background

Let’s start with some background information from [Micrsooft Learn: Windows Time Service Tools and Settings](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings)

Most domain member computers have a time client type of **NT5DS**, which means that they synchronize time from the domain hierarchy.
The only typical exception to this is the domain controller that functions as the primary domain controller (PDC) emulator operations master of the forest root domain, which is usually configured to synchronize time with an external time source. 
To view the time client configuration of a computer, run the `W32tm /query /configuration` command from an elevated Command Prompt and look for the "[TimeProviders]" block and the `Type` in the command output. 

For more information, see [How Windows Time Service Works | Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/How-the-Windows-Time-Service-Works). 
You can run the command `reg query HKLM\SYSTEM\CurrentControlSet\Services\W32Time\Parameters` and read the value of NtpServer in the command output.

## How Windows Time works in a Windows domain hierarchy

The following part is very important to understand:

- By default, member servers synchronizes time with a Domain Controller in their respective domain for the correct time
- Domain controllers synchronizes time with a Domain Controller which hosts the PDC emulator role in their local domain
- The PDC emulator queries any Domain Controller or PDC emulator in the parent domain.
- The PDC emulator in the parent domain is the highest time authority in the forest and should synchronize it’s time from a reliable time source outside the domain.

The following figure from Microsoft Learn illustrates a path of time synchronization between computers in a domain hierarchy:

![Time Synchronization in an AD DS Hierarchy](/assets/images/post-2022-09-mslearn-time-synchronization-hierarchy.gif)

This means that under normal circumstances, you should not make any changes to any server in your domain in regards to Windows Time Service configuration, with the exception of the PDC emulator in the forest root domain.
In case you have only one domain, this is automatically the forest root or parent domain.

Furthermore, if you need to make changes to Windows Time, you should use the right tools to do so.

The right tool is not, and I repeat **NOT** the registry editor (regedit). Neither `net time` should be used for this task. Net time has been deprecated.

The right tool is `W32tm.exe`. You should be able to query and configure everything related to Windows Time using w32tm.exe. (At the time of writing, there isn’t a PowerShell cmdlet to manage Windows Time Service.)

## The PDC emulator

As mentioned before, the PDC emulator in the forest root domain should be configured to synchronize it’s time from an external time source. Otherwise, it will use it’s own hardware clock and there will be no guarantee that it will be accurate. Now you can either use an internal NTP server in your datacenter, or select a proper time server on the Internet. I usually choose the pool.ntp.org servers as a reliable time source. As my servers are based in the Netherlands, I choose the `nl.pool.ntp.org` servers as listed [here](https://www.ntppool.org/zone/@).

> Note: in order to be able to synchronize time from an ntp server on the Internet, UDP port 123 should be opened to the internet. (outgoing traffic)

To check which server holds the PDC emulator role, you can use the following command on any server in the root domain:

```shell
netdom query fsmo
```

This will look something like this:

```shell
Schema master               dc01.test.local
Domain naming master        dc01.test.local
PDC                         dc01.test.local
RID pool manager            dc02.test.local
Infrastructure master       dc02.test.local
```

### Synchronize PDC with NTP server

For example, from your root DC holding the PDC emulator role, use the following command to synchronize time with the nl.pool.ntp.org timeservers:

```shell
w32tm /config /manualpeerlist:0.nl.pool.ntp.org,1.nl.pool.ntp.org,2.nl.pool.ntp.org,3.nl.pool.ntp.org /syncfromflags:manual /reliable:yes /update
```

Stop and start the time

```shell
net stop w32time
net start w32time
```

After that, you should see the following Time-Service events in your System Event log on the PDC emulator:

```shell
Event 37: The time provider NtpClient is currently receiving valid time data from <your NTP sources here>
Event 35: The time service is now synchronizing the system time with the time source <your NTP sources here>
Event 139: The time service has started advertising as a time source.
Event 143: The time service has started advertising as a good time source.
Note: syncfromflags option:

MANUAL – sync from peers in the manual peer list
DOMHIER – sync from an AD DC in the domain hierarchy
NO – sync from none
ALL – sync from both manual and domain peers
```

After configuring your root PDC emulator, all other servers should by default use the domain hierarchy to synchronise the time. So under normal conditions, you are done.

## Testing

To test your domain members, run the following command:

```shell
w32tm /monitor
```

You should see the name of the server it’s synchronising time with, the ICMP delay (if any) the offset and the stratum.

If you for some reason want to force a server to use the domain hierarchy to update it’s time, use the following command:

```shell
w32tm /config /syncfromflags:domhier /update
```

After that, restart the time service as described before.

To reconfigure the previous PDC Emulator, in case of transferring/seizing the FSMO to another Domain Controller, run:

```shell
w32tm /config /syncfromflags:domhier /reliable:no /update
```

## Troubleshooting Windows Time Service 

Here’s a quick summary for troubleshooting time issues in your domain environment.

- Check where your PDC emulators are by using netdom query fmso
- Check the registry settings of your servers by using reg query HKLM\SYSTEM\CurrentControlSet\Services\W32Time\Parameters
- Check the System eventlog for Time-Service events and errors
- Use `w32tm/monitor` to check where your server is getting it’s time from, and what the current offset is.
- Check if the firewall (Windows and/or hardware firewall) is preventing succesful time synchronisation between servers. UDP port 123 should be open between member servers and all domain controllers and between all DC’s. Also, some hardware firewalls have a default ACL to block UDP port 123.
- If using virtual machines, make sure that your Domain Controller VM’s don’t synchronise their time from the physical host. By default, Hyper-V Virtual Machines synchronise their time from the physical host using the Hyper-V Time Synchronization Service. This should be disabled as described in the following Technet article: Running Domain Controllers in Hyper-V. For VMware, you should disable this option in VMware Tools, either from within the Guest OS, or from the Virtual Machine settings in VMware.

## Useful commands

And finally, here are some useful commands for troubleshooting the time service:

To check where your server is getting it’s time from, and what the current offset is:

```shell
w32tm /monitor
```

To force a resync of the time and check for any errors:

```shell
w32tm /resync
```

To display a strip chart of the offset between this computer and another computer:

```shell
w32tm /stripchart /computer:<computer you want to check against> /dataonly /samples:3
```

If, for some reason, you want to reset Windows time to it’s default settings, you can do as follows:

```shell
net stop w32time
w32tm /unregister
w32tm /register
net start w32time
```

## Useful resources

- [w32tm command help \| Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings?source=recommendations#run-w32tmexe)
- [What is Windows Time Service \| Microsoft Learn](https://learn.microsoft.com/en-us/archive/blogs/w32time/what-is-windows-time-service)
- [Windows Time Service Tools and Settings \| Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings)