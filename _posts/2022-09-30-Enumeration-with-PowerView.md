## Enumeration Using PowerView

This blog post includes post exploitation enumeration methods using PowerView that could help in lateral movement and data hunting.

---

### Getting Started

If you try running PowerView script with default settings and configuration, you might encounter error, one such example is below.

![Error ng](https://user-images.githubusercontent.com/60311143/192285693-55bb4e49-1302-465f-b691-f30044e162f1.PNG)

We can get past this error by changing the execution policy to `RemoteSigned`. This allows running scripts.
```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```
![1](https://user-images.githubusercontent.com/60311143/192286179-d4a4730b-7d4d-40be-9695-315e2a2054f6.png)


### Get that Domain!!!

To get a domain object of the current domain run the following command. Get details of a partiular domain by running the command with `-Domain'.
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

To get domains present in the current forest, run the following command. Get details of a partiular forest by running the command with `-Domain'.

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
Currently I have only one domain set up in my home lab therefore the `Get-Domain` and `Get-ForestDomain` return the same results.
The amount of data returned in real scenarios can be huge and therefore we can narrow down our results to get only the required data as below.

```
PS C:\Users\Administrator\Desktop> Get-ForestDomain | select Name, Forest, DomainControllers

Name         Forest       DomainControllers
----         ------       -----------------
SPRINT.local SPRINT.local {HYDRA-DC.SPRINT.local}
```


To get the domain controller details run the following command.
```
PS C:\Users\Administrator\Desktop> Get-DomainController


Forest                     : SPRINT.local
CurrentTime                : 9/26/2022 1:40:46 PM
HighestCommittedUsn        : 20523
OSVersion                  : Windows Server 2019 Standard Evaluation
Roles                      : {SchemaRole, NamingRole, PdcRole, RidRole...}
Domain                     : SPRINT.local
IPAddress                  : fe80::709b:2415:3143:32e2%6
SiteName                   : Default-First-Site-Name
SyncFromAllServersCallback :
InboundConnections         : {}
OutboundConnections        : {}
Name                       : HYDRA-DC.SPRINT.local
Partitions                 : {DC=SPRINT,DC=local, CN=Configuration,DC=SPRINT,DC=local,
                             CN=Schema,CN=Configuration,DC=SPRINT,DC=local, DC=DomainDnsZones,DC=SPRINT,DC=local...}
```
