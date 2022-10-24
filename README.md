# Sync-DynamicAdGroupMember

## Links
* [Blog post](https://autofocus.duennebacke.com/2022/10/10/dynamic-active-directory-groups/) on autofocus.duennebacke.com for a pratical example using the script

## Synopsis
Manages AD group members based on Get-ADUser filter query defined in an `extensionAttribute`.

## Syntax
```powershell
.\Sync-DynamicAdGroupMember.ps1 -ExtensionAttribute <int> [-SearchBase <String>] [-Server <String>] [-WhatIf] [-PassThru] [-VERBOSE]
```

## Description
The Sync-DynamicAdGroupMember.ps1 loops thru all AD groups that have a Get-ADUser filter query defined on a speficied extensionAttribute.
The script then fetches all AD users that match the query and syncs them with the groups members.
This means missing members are added and obsolete members are removed.
Manual changes to the members of the group are overwritten.

### Use case
Dynamic groups is a very useful feature in Azure AD. Since many organizations still use Active Directory as their source of truth for users, the same functionality can be useful there too.

### Requirements
* PowerShell module [ActiveDirectory](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
* Execution on a domain-joined Windows machine that has an active connection to a domain controller
* Execution by a user with permission to add/remove members in target AD groups

### Installation
* Download the script file:
    ```powershell
    Invoke-WebRequest -Uri "https://raw.githubusercontent.com/dominikduennebacke/Sync-DynamicAdGroupMember/main/Sync-DynamicAdGroupMember.ps1" -OutFile "Sync-DynamicAdGroupMember.ps1"
    ```
* Set up a scheduled task or a CI/CD job that runs the script every 5-10 minutes

### Scaling
After configuring the Get-ADUser filter query on a few group pairs keep an eye on the execution time of the script which should not be larger than the scheduling interval.

## Examples

### Example 1
Syncs all members for groups that have a filter query set in `extensionAttribute10` and provides output.
```powershell
.\Sync-DynamicAdGroupMember.ps1 -ExtensionAttribute 10 -VERBOSE
```
```
VERBOSE: Checking dependencies
VERBOSE: The secure channel between the local computer and the domain is in good condition.
VERBOSE: Fetching AD groups with a value in extensionAttribute10
VERBOSE: Syncing group members
VERBOSE: role-department-sales: department -eq 'sales'
VERBOSE: role-department-sales: (+) john.doe
VERBOSE: role-department-sales: (+) sam.smith
VERBOSE: role-department-sales: (-) tom.tonkins
```

### Example 2
Provides output of sync changes but does not actually perform them.
```powershell
.\Sync-DynamicAdGroupMember.ps1 -ExtensionAttribute 10 -WhatIf:$true
```
```
What if: role-department-sales: (+) john.doe
What if: role-department-sales: (+) sam.smith
What if: role-department-sales: (-) tom.tonkins
```

### Example 3
Provides output of sync changes but does not actually perform them, with additional output.
```powershell
.\Sync-DynamicAdGroupMember.ps1 -ExtensionAttribute 10 -WhatIf:$true -VERBOSE
```
```
VERBOSE: Checking dependencies
VERBOSE: The secure channel between the local computer and the domain is in good condition.
VERBOSE: Fetching AD groups with a value in extensionAttribute10
VERBOSE: Syncing group members
VERBOSE: role-department-sales: department -eq 'sales'
What if: role-department-sales: (+) john.doe
What if: role-department-sales: (+) sam.smith
What if: role-department-sales: (-) tom.tonkins
```

### Example 4
Only consideres the OU `OU=groups,DC=contoso,DC=com` looking for groups with extensionAttribute10 set.
Only consideres the OU `OU=users,DC=contoso,DC=com` looking for users when executing the Get-ADUser filter query.
This can speed up execution.
```powershell
.\Sync-DynamicAdGroupMember.ps1 -ExtensionAttribute 10 -GroupSearchBase "OU=groups,DC=contoso,DC=com" -UserSearchBase "OU=users,DC=contoso,DC=com"
```
## Parameters

### -ExtensionAttribute
extensionAttribute where the Get-ADUser filter query is specified in AD groups.

```yaml
Type: Integer
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -GroupSearchBase
Specifies an Active Directory path to search for groups.

```yaml
Type: String
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -UserSearchBase
Specifies an Active Directory path to search for users.

```yaml
Type: String
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -Server
Specifies the Active Directory Domain Services instance to connect to.

```yaml
Type: String
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -WhatIf
Shows what would happen if the cmdlet runs.

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

### -PassThru
Returns the group, its Get-ADUser filter query, user and modification type as PSCustomObject.
If -PassThru is not specified, this cmdlet does not generate any output.

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -VERBOSE
Provides additional output about the process.

## INPUTS
### None

## OUTPUTS

### None or PSCustomObject
Returns the group, its Get-ADUser filter query, user and modification type as PSCustomObject if the PassThru parameter is specified.
By default, this cmdlet does not generate any output.