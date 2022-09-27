## Domain Enumeration Using PowerView

This blog post includes post exploitation enumeration methods using [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) that could help in lateral movement and data hunting.

---

### Getting Started

If you try running PowerView script with default settings and configuration, you might encounter errors, one such example is below.

![Error ng](https://user-images.githubusercontent.com/60311143/192285693-55bb4e49-1302-465f-b691-f30044e162f1.PNG)

We can get past this error by changing the execution policy to `RemoteSigned`. This allows running scripts.
```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```
![1](https://user-images.githubusercontent.com/60311143/192286179-d4a4730b-7d4d-40be-9695-315e2a2054f6.png)


### Get that Domain!

To get a domain object of the current domain run the following command. Get details of a partiular domain by running the command with `-Domain` parameter.
```
PS C:\Users\Administrator\Desktop> Get-Domain


Forest                  : SPRINT.local
DomainControllers       : {HYDRA-DC.SPRINT.local}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  :
PdcRoleOwner            : HYDRA-DC.SPRINT.local
RidRoleOwner            : HYDRA-DC.SPRINT.local
InfrastructureRoleOwner : HYDRA-DC.SPRINT.local
Name                    : SPRINT.local
```

To get domains present in the current forest, run the following command. Get details of a partiular forest by running the command with `-Domain` parameter.

```
PS C:\Users\Administrator\Desktop> Get-ForestDomain


Forest                  : SPRINT.local
DomainControllers       : {HYDRA-DC.SPRINT.local}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  :
PdcRoleOwner            : HYDRA-DC.SPRINT.local
RidRoleOwner            : HYDRA-DC.SPRINT.local
InfrastructureRoleOwner : HYDRA-DC.SPRINT.local
Name                    : SPRINT.local
```
Currently I have only one domain set up in my home lab therefore the `Get-Domain` and `Get-ForestDomain` cmdlets return the same results.
The amount of data returned in real scenarios can be huge and therefore we can narrow down our results to get only the required data as below.

```
PS C:\Users\Administrator\Desktop> Get-ForestDomain | select Name, Forest, DomainControllers

Name         Forest       DomainControllers
----         ------       -----------------
SPRINT.local SPRINT.local {HYDRA-DC.SPRINT.local}
```


To get the domain controller details run the following command.
```
PS C:\Users\Administrator\Desktop> Get-DomainController | select Forest, OSVersion, Name | fl


Forest    : SPRINT.local
OSVersion : Windows Server 2019 Standard Evaluation
Name      : HYDRA-DC.SPRINT.local
```
#### Domain Users

It's better to run the command with proper specifications like using `-Identity` to mention the users, this prevents huge amount of data being returned in response. 
```
PS C:\Users\Administrator\Desktop> Get-DomainUser -Identity jdoe, HulkSM | select logoncount, objectsid, displayname | fl


logoncount  : 0
objectsid   : S-1-5-21-2276919464-3290130273-2584351583-1107
displayname : Hulk SM

logoncount  : 0
objectsid   : S-1-5-21-2276919464-3290130273-2584351583-1108
displayname : John Doe
```
Get the list of enabled and disabled user accounts in the domain. 
```
PS C:\Users\Administrator\Desktop> Get-DomainUser -UACFilter NOT_ACCOUNTDISABLE | select distinguishedname, objectsid, cn | fl


distinguishedname : CN=Administrator,CN=Users,DC=SPRINT,DC=local
objectsid         : S-1-5-21-2276919464-3290130273-2584351583-500
cn                : Administrator

distinguishedname : CN=Hulk SM,CN=Users,DC=SPRINT,DC=local
objectsid         : S-1-5-21-2276919464-3290130273-2584351583-1107
cn                : Hulk SM

distinguishedname : CN=John Doe,CN=Users,DC=SPRINT,DC=local
objectsid         : S-1-5-21-2276919464-3290130273-2584351583-1108
cn                : John Doe

distinguishedname : CN=SQL Service,CN=Users,DC=SPRINT,DC=local
objectsid         : S-1-5-21-2276919464-3290130273-2584351583-1109
cn                : SQL Service

```
```
PS C:\Users\Administrator\Desktop> Get-DomainUser -UACFilter ACCOUNTDISABLE | select distinguishedname, objectsid, cn | fl


distinguishedname : CN=Guest,CN=Users,DC=SPRINT,DC=local
objectsid         : S-1-5-21-2276919464-3290130273-2584351583-501
cn                : Guest

distinguishedname : CN=krbtgt,CN=Users,DC=SPRINT,DC=local
objectsid         : S-1-5-21-2276919464-3290130273-2584351583-502
cn                : krbtgt
```

Get the smartcard authentication requirements.
```
PS C:\Users\Administrator\Desktop> Get-DomainUser -UACFilter NOT_SMARTCARD_REQUIRED | select cn

cn
--
Administrator
Guest
krbtgt
Hulk SM
John Doe
SQL Service
```

Get the accounts with no kerberos pre-authentication required(ASREP Roastable). This can help avoid roasting all the accounts in the domain.
```
Get-DomainUser -UACFilter DONT_REQ_PREAUTH | select cn, samaccountname
```

Get the users with SPN set.
```
PS C:\Users\Administrator\Desktop> Get-DomainUser -SPN | select distinguishedname, samaccountname, cn

distinguishedname                          samaccountname cn
-----------------                          -------------- --
CN=krbtgt,CN=Users,DC=SPRINT,DC=local      krbtgt         krbtgt
CN=SQL Service,CN=Users,DC=SPRINT,DC=local SQLService     SQL Service
```

Users with constrained delegation set.
```
Get-DomainUser -TrustedToAuth
```

Privileged users which are not marked as sensitive or not for delegation.
```
PS C:\Users\Administrator\Desktop> Get-DomainUser -AllowDelegation -AdminCount | select samaccountname

samaccountname
--------------
Administrator
krbtgt
HulkSM
```

#### Domain Groups

Get the members of a specified group.
```
PS C:\Users\Administrator\Desktop> Get-DomainGroupMember -Identity "Domain Admins"


GroupDomain             : SPRINT.local
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,OU=Groups,DC=SPRINT,DC=local
MemberDomain            : SPRINT.local
MemberName              : HulkSM
MemberDistinguishedName : CN=Hulk SM,CN=Users,DC=SPRINT,DC=local
MemberObjectClass       : user
MemberSID               : S-1-5-21-2276919464-3290130273-2584351583-1107

GroupDomain             : SPRINT.local
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,OU=Groups,DC=SPRINT,DC=local
MemberDomain            : SPRINT.local
MemberName              : Administrator
MemberDistinguishedName : CN=Administrator,CN=Users,DC=SPRINT,DC=local
MemberObjectClass       : user
MemberSID               : S-1-5-21-2276919464-3290130273-2584351583-500
```

Search for all the groups a user is a part of.
```
PS C:\Users\Administrator\Desktop> Get-DomainGroup -MemberIdentity HulkSM | select cn

cn
--
Denied RODC Password Replication Group
Schema Admins
Enterprise Admins
Domain Users
Group Policy Creator Owners
Domain Admins
```

#### Policies

Get the list of domain policies. The policies can be checked for other domain by specifying a domain using `-Domain`.
```
PS C:\Users\Administrator\Desktop> Get-DomainPolicy


Unicode        : @{Unicode=yes}
SystemAccess   : @{MinimumPasswordAge=1; MaximumPasswordAge=42; MinimumPasswordLength=7; PasswordComplexity=1; PasswordHistorySize=24;
                 LockoutBadCount=0; RequireLogonToChangePassword=0; ForceLogoffWhenHourExpire=0; ClearTextPassword=0;
                 LSAAnonymousNameLookup=0}
KerberosPolicy : @{MaxTicketAge=10; MaxRenewAge=7; MaxServiceAge=600; MaxClockSkew=5; TicketValidateClient=1}
RegistryValues : @{MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash=System.Object[]}
Version        : @{signature="$CHICAGO$"; Revision=1}
Path           : \\SPRINT.local\sysvol\SPRINT.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\Windows
                 NT\SecEdit\GptTmpl.inf
GPOName        : {31B2F340-016D-11D2-945F-00C04FB984F9}
GPODisplayName : Default Domain Policy
```

If you wish to look for a particular policy do as below. In this case we are looking for the kerberos policy.
```
PS C:\Users\Administrator\Desktop> (Get-DomainPolicy)."KerberosPolicy"


MaxTicketAge         : 10
MaxRenewAge          : 7
MaxServiceAge        : 600
MaxClockSkew         : 5
TicketValidateClient : 1
```

#### Domain Computers

Get the computer objects in the current domain. 
```
PS C:\Users\Administrator\Desktop> Get-DomainComputer | select dnshostname, useraccountcontrol, samaccounttype, objectsid | fl


dnshostname        : HYDRA-DC.SPRINT.local
useraccountcontrol : SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION
samaccounttype     : MACHINE_ACCOUNT
objectsid          : S-1-5-21-2276919464-3290130273-2584351583-1000

dnshostname        : THEPRACTITIONER.SPRINT.local
useraccountcontrol : WORKSTATION_TRUST_ACCOUNT
samaccounttype     : MACHINE_ACCOUNT
objectsid          : S-1-5-21-2276919464-3290130273-2584351583-1110
```
Get the domain computers with unconstrained delegation.
```
PS C:\Users\Administrator\Desktop> Get-DomainComputer -Unconstrained | select distinguishedname, name, samaccountname, dnshostname | fl


distinguishedname : CN=HYDRA-DC,OU=Domain Controllers,DC=SPRINT,DC=local
name              : HYDRA-DC
samaccountname    : HYDRA-DC$
dnshostname       : HYDRA-DC.SPRINT.local
```

Get the computers with constrained delegation.
```
Get-DomainComputer -TrustedToAuth | select distinguishedname, name, samaccountname, dnshostname
```

#### Organizational Units (OUs)

```
PS C:\Users\Administrator\Desktop> Get-DomainOU | select gplink, name, distinguishedname | fl


gplink            : [LDAP://CN={6AC1786C-016F-11D2-945F-00C04fB984F9},CN=Policies,CN=System,DC=SPRINT,DC=local;0]
name              : Domain Controllers
distinguishedname : OU=Domain Controllers,DC=SPRINT,DC=local
```
I will be covering GPOs, ACL, Trusts and a few other topics in upcoming parts. Thanks for the read!

Reference: [Powerview Documentation](https://powersploit.readthedocs.io/en/latest/Recon/). 
