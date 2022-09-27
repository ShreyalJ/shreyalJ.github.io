## Domain Enumeration Using PowerView Part 2
This blog post is the continuation of [Domain Enumeration Using PowerView](https://shreyalj.github.io/2022/09/27/Domain-Enumeration-with-PowerView-Part-1.html).

---

#### GPOs

Return all GPOs or specific GPO objects in AD. By using `Properties` it is possible to specify the properties of the output object.
```
PS C:\Users\Administrator\Desktop> Get-DomainGPO | select displayname, name, gpcfilesyspath, distinguishedname | fl


displayname       : Default Domain Policy
name              : {31B2F340-016D-11D2-945F-00C04FB984F9}
gpcfilesyspath    : \\SPRINT.local\sysvol\SPRINT.local\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}
distinguishedname : CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=SPRINT,DC=local

displayname       : Default Domain Controllers Policy
name              : {6AC1786C-016F-11D2-945F-00C04fB984F9}
gpcfilesyspath    : \\SPRINT.local\sysvol\SPRINT.local\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}
distinguishedname : CN={6AC1786C-016F-11D2-945F-00C04fB984F9},CN=Policies,CN=System,DC=SPRINT,DC=local

displayname       : Disable Windows Defender
name              : {37EDB51E-8F73-41B1-9700-FCF8D46C1FE8}
gpcfilesyspath    : \\SPRINT.local\SysVol\SPRINT.local\Policies\{37EDB51E-8F73-41B1-9700-FCF8D46C1FE8}
distinguishedname : CN={37EDB51E-8F73-41B1-9700-FCF8D46C1FE8},CN=Policies,CN=System,DC=SPRINT,DC=local
```

```
PS C:\Users\Administrator\Desktop> Get-DomainGPO -Identity '{6AC1786C-016F-11D2-945F-00C04fB984F9}' -Properties displayname, cn | fl


displayname : Default Domain Controllers Policy
cn          : {6AC1786C-016F-11D2-945F-00C04fB984F9}
```

Get all the GPOs in a domain that modify local group memberships through 'Restricted Groups' or Group Policy preferences.
```
Get-DomainGPOLocalGroup -ResolveMembersToSIDs
```
The `-ResolveMembersToSIDs` parameter is self explanatory.

Get the machines in a domain where user/group is a member of a specific local group.
```
Get-DomainGPOUserLocalGroupMapping -LocalGroup Administrators
```

Get the users/groups in a specified local group for a machine through GPO correlation.
```
Get-DomainGPOComputerLocalGroupMapping -ComputerName THEPRACTITIONER.SPRINT.local
```

#### ACL

Get ACLs for an AD object and resolve relevant GUIDs to their display names.
```
PS C:\Users\Administrator\Desktop> Get-DomainObjectAcl -Identity 'Hulk SM' -ResolveGUIDs


AceQualifier           : AccessAllowed
ObjectDN               : CN=Hulk SM,CN=Users,DC=SPRINT,DC=local
ActiveDirectoryRights  : ReadProperty
ObjectAceType          : User-Account-Restrictions
ObjectSID              : S-1-5-21-2276919464-3290130273-2584351583-1107
InheritanceFlags       : None
BinaryLength           : 60
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent, InheritedObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-32-554
AccessMask             : 16
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : inetOrgPerson
OpaqueLength           : 0
```

Get the object ACLs in the current (or specified) domain with modification rights set to non-built in objects.
```
PS C:\Users\Administrator\Desktop> Find-InterestingDomainAcl -Domain SPRINT.local -ResolveGUIDs


ObjectDN                : CN=MicrosoftDNS,CN=System,DC=SPRINT,DC=local
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : CreateChild, DeleteChild, ListChildren, ReadProperty, DeleteTree, ExtendedRight, Delete, GenericWrite, WriteDacl, WriteOwner
ObjectAceType           : None
AceFlags                : ContainerInherit
AceType                 : AccessAllowed
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-2276919464-3290130273-2584351583-1101
IdentityReferenceName   : DnsAdmins
IdentityReferenceDomain : SPRINT.local
IdentityReferenceDN     : CN=DnsAdmins,OU=Groups,DC=SPRINT,DC=local
IdentityReferenceClass  : group
```

#### Trusts

Get all domain trusts(Inbound or Outbound). To get the trusts for a specific domain, use `-Domain` parameter.
```
Get-DomainTrust
```

Get all forest trusts for the current forest. To get the trusts for a specified forest, use `-Forest` parameter.
```
Get-ForestTrust
```

#### Login and Sessions

Get all the Logged on user on the computers present in the current domain.
```
PS C:\Users\Administrator\Desktop> Get-DomainComputer | Get-NetLoggedon


UserName     : Administrator
LogonDomain  : SPRINT
AuthDomains  :
LogonServer  : HYDRA-DC
ComputerName : HYDRA-DC.SPRINT.local

UserName     : HYDRA-DC$
LogonDomain  : SPRINT
AuthDomains  :
LogonServer  :
ComputerName : HYDRA-DC.SPRINT.local
```

Get the session details on local machine or a remote machine by specifying the machine name using `-ComputerName` parameter.
```
PS C:\Users\Administrator\Desktop> Get-NetSession


CName        : \\[::1]
UserName     : Administrator
Time         : 0
IdleTime     : 0
ComputerName : localhost
```

In the next part, I'll be covering ldap queries introduction and usage for enumeration. Thanks for the read!
