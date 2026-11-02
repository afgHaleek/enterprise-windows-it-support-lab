# LAB-015 — Active Directory User Lifecycle Management (Joiner–Mover–Leaver)

**Environment:** NovaTech Enterprise Windows IT Support Lab  
**Domain:** `corp.novatech.test` (`NOVATECH`)  
**Domain controller:** `DC01`, Windows Server 2025, `192.168.10.10`  
**Client:** `CLIENT01`, domain-joined Windows 11  
**Tools:** Active Directory Users and Computers, Group Policy Management, Windows Event Viewer, PowerShell ActiveDirectory module  
**Scope:** Controlled, fictional helpdesk scenarios in an isolated VirtualBox lab.

## Objective and outcome

This lab exercised the complete Joiner–Mover–Leaver (JML) lifecycle and related first-line account support. The work included creating and verifying an account, assigning role-based permissions, testing sign-in and resource access, resetting a password, reproducing and resolving a lockout, transferring an employee between departments, disabling and retaining an account, and producing an audit report.

**Completed:** Joiner, Mover, Leaver, and basic account reporting. The screenshots below provide the evidence trail. A separate HR file share was not created; HR access was verified through group membership and the removal of old Finance access.

## 1. Joiner — Emma Collins, Finance

A fictional new employee, **Emma Collins** (`emma.collins`), was created in the Finance OU with department `Finance`, title `Finance Assistant`, and a temporary password requiring change at first sign-in. The account was verified using `Get-ADUser`, and membership was assigned to the existing `GG_Finance_Users` global security group.

The established **AGDLP** relationship was `Emma → GG_Finance_Users → DL_Finance_Modify → Finance folder permissions`. No direct membership in `DL_Finance_Modify` was necessary. This keeps employee membership separate from resource permission assignments.

```powershell
Get-ADUser 'emma.collins' -Properties Department,Title,Enabled,PasswordLastSet |
    Select-Object Name,SamAccountName,Department,Title,Enabled,PasswordLastSet
Add-ADGroupMember -Identity 'GG_Finance_Users' -Members 'emma.collins'
Get-ADPrincipalGroupMembership 'emma.collins' | Select-Object Name
```

r![Emma Collins account in the Finance OU](../../screenshots/active-directory/user-lifecycle/01-emma-account-created.png)

*Evidence 01: Emma Collins account in the Finance OU.*
![Account properties verified in PowerShell](../../screenshots/active-directory/user-lifecycle/02-emma-account-verified.png)

*Evidence 02: Account properties verified in PowerShell.*
![Membership in the Finance global security group](../../screenshots/active-directory/user-lifecycle/03-emma-finance-group-membership.png)

*Evidence 03: Membership in the Finance global security group.*

On `CLIENT01`, Emma signed in with her domain account, completed the required password change, and verified identity using `whoami`. The Finance share `\\DC01\Finance` was accessible, and the existing Finance drive-mapping GPO provided an `F:` mapping.

r![Successful Emma domain sign-in](../../screenshots/active-directory/user-lifecycle/04-emma-domain-logon.png)

*Evidence 04: Successful Emma domain sign-in.*
![Emma access to Finance shared folder](../../screenshots/active-directory/user-lifecycle/05-emma-finance-folder-access.png)

*Evidence 05: Emma access to Finance shared folder.*
![Finance drive-mapping GPO applied](../../screenshots/active-directory/user-lifecycle/06-emma-finance-gpo-applied.png)

*Evidence 06: Finance drive-mapping GPO applied.*
![Finance drive mapped to F:](../../screenshots/active-directory/user-lifecycle/07-emma-finance-drive-mapped.png)

*Evidence 07: Finance drive mapped to F:.*

## 2. Helpdesk account support — password reset

A simulated forgotten-password request was handled by checking account state, resetting a temporary password using a secure prompt, requiring a change at next sign-in, and verifying successful authentication. No password was recorded in screenshots or the repository.

```powershell
Get-ADUser 'emma.collins' -Properties Enabled,LockedOut,PasswordLastSet |
    Select-Object Name,Enabled,LockedOut,PasswordLastSet
$newPassword = Read-Host 'Enter temporary password' -AsSecureString
Set-ADAccountPassword -Identity 'emma.collins' -Reset -NewPassword $newPassword
Set-ADUser -Identity 'emma.collins' -ChangePasswordAtLogon $true
```

r![Emma account status before password reset](../../screenshots/active-directory/user-lifecycle/08-emma-account-status-before-reset.png)

*Evidence 08: Emma account status before password reset.*
![Password reset verification](../../screenshots/active-directory/user-lifecycle/09-emma-password-reset-verified.png)

*Evidence 09: Password reset verification.*
![Emma authentication after password reset](../../screenshots/active-directory/user-lifecycle/10-emma-login-after-password-reset.png)

*Evidence 10: Emma authentication after password reset.*

## 3. Helpdesk account support — lockout investigation

The initial default domain password policy was inspected: minimum password length **7**, complexity enabled, and `LockoutThreshold = 0` (no default-domain lockout). The lab then configured a controlled **five-attempt** lockout threshold and **10-minute** duration/observation window in the Default Domain Policy. These were lab-only settings, not production security recommendations.

Michael Brown (`michael.brown`) was initially enabled and unlocked. After deliberate incorrect sign-in attempts, `Get-ADUser` confirmed `LockedOut = True` and `BadPwdCount = 5`. The workflow included inspecting Security Event ID **4740** (when present), unlocking the account, and verifying that authentication was restored.

```powershell
Get-ADDefaultDomainPasswordPolicy |
    Select-Object LockoutThreshold,LockoutDuration,LockoutObservationWindow
Get-ADUser 'michael.brown' -Properties LockedOut,BadPwdCount,LastBadPasswordAttempt |
    Select-Object Name,LockedOut,BadPwdCount,LastBadPasswordAttempt
Unlock-ADAccount -Identity 'michael.brown'
```

The root-cause principle was that unlocking an account does not correct an application, scheduled task, saved credential, or device repeatedly submitting an outdated password. A recurring lockout would require correlating events and correcting the credential source.

r![Original default domain lockout threshold of zero](../../screenshots/active-directory/user-lifecycle/12-domain-password-lockout-policy.png)

*Evidence 12: Original default domain lockout threshold of zero.*
![Michael account before controlled lockout](../../screenshots/active-directory/user-lifecycle/13-michael-account-before-lockout.png)

*Evidence 13: Michael account before controlled lockout.*
![Configured lab account lockout policy](../../screenshots/active-directory/user-lifecycle/14-domain-account-lockout-policy-configured.png)

*Evidence 14: Configured lab account lockout policy.*
![Effective domain lockout policy verified](../../screenshots/active-directory/user-lifecycle/15-domain-lockout-policy-verified.png)

*Evidence 15: Effective domain lockout policy verified.*
![Michael locked after five bad password attempts](../../screenshots/active-directory/user-lifecycle/16-michael-account-locked.png)

*Evidence 16: Michael locked after five bad password attempts.*
![Event Viewer account lockout investigation](../../screenshots/active-directory/user-lifecycle/17-michael-lockout-event-4740.png)

*Evidence 17: Event Viewer account lockout investigation.*
![Michael account unlocked](../../screenshots/active-directory/user-lifecycle/18-michael-account-unlocked.png)

*Evidence 18: Michael account unlocked.*
![Michael successfully signs in again](../../screenshots/active-directory/user-lifecycle/19-michael-login-restored.png)

*Evidence 19: Michael successfully signs in again.*

## 4. Mover — Emma transfers from Finance to HR

Emma was transferred to HR and given the title `HR Coordinator`. Both her **attributes** and **OU location** were changed; editing `Department` alone does not relocate an AD object. The existing Finance membership was documented *before* removal to preserve before/after evidence.

```powershell
Set-ADUser 'emma.collins' -Department 'HR' -Title 'HR Coordinator'
Get-ADUser 'emma.collins' | Move-ADObject `
    -TargetPath 'OU=HR,OU=Users,OU=NovaTech,DC=corp,DC=novatech,DC=test'
```

Since no HR groups existed, `GG_HR_Users` (Global, Security) and `DL_HR_Modify` (Domain Local, Security) were created in the NovaTech Groups OU. The global group was nested inside the domain local group; Emma joined the global group. She was then removed from `GG_Finance_Users` to enforce least privilege.

```powershell
Add-ADGroupMember -Identity 'DL_HR_Modify' -Members 'GG_HR_Users'
Add-ADGroupMember -Identity 'GG_HR_Users' -Members 'emma.collins'
Remove-ADGroupMember -Identity 'GG_Finance_Users' -Members 'emma.collins' -Confirm:$false
Get-ADPrincipalGroupMembership 'emma.collins' | Select-Object Name
```

Following a full sign-out and sign-in on `CLIENT01`, the new security token showed HR group membership and no Finance group membership. Access to `\\DC01\Finance` was denied, and the Finance drive-mapping GPO was no longer applied. The lab confirmed access **revocation**; no HR resource ACL was configured in this exercise.

r![Emma department, title and HR OU verified](../../screenshots/active-directory/user-lifecycle/20-emma-transferred-to-hr.png)

*Evidence 20: Emma department, title and HR OU verified.*
![Finance group membership before revocation](../../screenshots/active-directory/user-lifecycle/21-emma-finance-membership-before-removal.png)

*Evidence 21: Finance group membership before revocation.*
![HR global group nested in HR domain local group](../../screenshots/active-directory/user-lifecycle/22-hr-group-nesting.png)

*Evidence 22: HR global group nested in HR domain local group.*
![Emma added to HR global security group](../../screenshots/active-directory/user-lifecycle/23-emma-added-to-hr-group.png)

*Evidence 23: Emma added to HR global security group.*
![Finance group removed from Emma](../../screenshots/active-directory/user-lifecycle/24-emma-finance-membership-removed.png)

*Evidence 24: Finance group removed from Emma.*
![Fresh Windows logon token shows HR membership](../../screenshots/active-directory/user-lifecycle/25-emma-hr-logon-token.png)

*Evidence 25: Fresh Windows logon token shows HR membership.*
![Finance share access denied after transfer](../../screenshots/active-directory/user-lifecycle/26-emma-finance-access-denied.png)

*Evidence 26: Finance share access denied after transfer.*
![Finance GPO no longer applies to Emma](../../screenshots/active-directory/user-lifecycle/27-emma-finance-gpo-removed.png)

*Evidence 27: Finance GPO no longer applies to Emma.*

## 5. Leaver — Michael Brown offboarding

The offboarding simulation first recorded Michael's account state and memberships. His `Department` and `Title` attributes were empty, and he was a member only of the default `Domain Users` group; therefore **there were no additional direct departmental memberships to remove**.

The account was disabled using `Disable-ADAccount`, verified with `Enabled = False`, and a fresh Windows sign-in was refused with the message that the account was disabled. A dedicated `Disabled-Users` OU was created under NovaTech, and Michael was moved there for retention review rather than deleted immediately.

```powershell
Get-ADUser 'michael.brown' -Properties Enabled,Department,Title,MemberOf |
    Select-Object Name,SamAccountName,Enabled,Department,Title,DistinguishedName
Disable-ADAccount -Identity 'michael.brown'
New-ADOrganizationalUnit -Name 'Disabled-Users' `
    -Path 'OU=NovaTech,DC=corp,DC=novatech,DC=test' `
    -Description 'Disabled employee accounts awaiting retention review'
Get-ADUser 'michael.brown' | Move-ADObject `
    -TargetPath 'OU=Disabled-Users,OU=NovaTech,DC=corp,DC=novatech,DC=test'
```

The final verification confirmed `Enabled = False`, `LockedOut = False`, and a distinguished name under `OU=Disabled-Users`. This prevents normal new domain authentication; it does not by itself prove all existing sessions or cloud sessions have been revoked.

r![Michael account and memberships before offboarding](../../screenshots/active-directory/user-lifecycle/28-michael-before-offboarding.png)

*Evidence 28: Michael account and memberships before offboarding.*
![Account disabled using PowerShell](../../screenshots/active-directory/user-lifecycle/29-michael-account-disabled.png)

*Evidence 29: Account disabled using PowerShell.*
![CLIENT01 refuses new sign-in for disabled account](../../screenshots/active-directory/user-lifecycle/30-michael-disabled-login-denied.png)

*Evidence 30: CLIENT01 refuses new sign-in for disabled account.*
![Account moved to Disabled-Users OU](../../screenshots/active-directory/user-lifecycle/31-michael-moved-to-disabled-users-ou.png)

*Evidence 31: Account moved to Disabled-Users OU.*
![Membership review shows only Domain Users](../../screenshots/active-directory/user-lifecycle/32-michael-offboarding-group-review.png)

*Evidence 32: Membership review shows only Domain Users.*
![Final audit confirms disabled account in retention OU](../../screenshots/active-directory/user-lifecycle/33-michael-offboarding-final-audit.png)

*Evidence 33: Final audit confirms disabled account in retention OU.*

## 6. PowerShell account audit and CSV export

A report of NovaTech OU users and child OUs was generated with `Get-ADUser`. It included `Name`, `SamAccountName`, `Department`, `Enabled`, `LockedOut`, `PasswordExpired`, and `LastLogonDate`. The observed report contained **Sarah Murphy (Finance, enabled), James Wilson (IT, enabled), Michael Brown (disabled), and Emma Collins (HR, enabled)**. No users were shown as locked out or password-expired at the time of the report.

```powershell
$report = Get-ADUser -SearchBase 'OU=NovaTech,DC=corp,DC=novatech,DC=test' `
    -Filter * -Properties Enabled,LockedOut,PasswordExpired,Department,LastLogonDate |
    Select-Object Name,SamAccountName,Department,Enabled,LockedOut,PasswordExpired,LastLogonDate
$report | Format-Table -AutoSize
$report | Export-Csv -Path 'C:\NovaTech-Reports\AD-User-Audit.csv' `
    -NoTypeInformation -Encoding UTF8
Import-Csv 'C:\NovaTech-Reports\AD-User-Audit.csv' |
    Format-Table Name,SamAccountName,Enabled,LockedOut -AutoSize
$disabledUsers = $report | Where-Object { $_.Enabled -eq $false }
$disabledUsers | Format-Table Name,SamAccountName,Department -AutoSize
```

The first CSV import failed due to a filename typo (`AD-User-Audit-csv` instead of `AD-User-Audit.csv`). After correcting the extension, the import succeeded. The disabled-account filter returned Michael Brown. `LastLogonDate` is approximate replicated logon information, not a precise forensic last-logon record.

r![AD user audit across NovaTech OUs](../../screenshots/active-directory/user-lifecycle/34-active-directory-account-audit.png)

*Evidence 34: AD user audit across NovaTech OUs.*
![CSV import succeeded after correcting filename](../../screenshots/active-directory/user-lifecycle/35-active-directory-audit-csv-export.png)

*Evidence 35: CSV import succeeded after correcting filename.*
![Filtered disabled account report](../../screenshots/active-directory/user-lifecycle/36-disabled-accounts-summary.png)

*Evidence 36: Filtered disabled account report.*

## Findings and operational lessons

- **OU placement and group membership are independent.** Moving an account to HR changes GPO scope but does not automatically remove Finance group access.
- **Group nesting supports least privilege.** Users belong to role-based global groups; resource permissions are delegated through domain local groups.
- **Logon tokens must refresh.** A full sign-out and sign-in was used to validate changed group access.
- **Account lockout is a symptom.** Repeated lockouts require investigation of the authentication source, not repeated unlocking alone.
- **Disabling is not deleting.** Retained accounts support approved data retention and audit workflows, but existing sessions and other identity systems require separate review.
- **Reporting is repeatable.** PowerShell produced a readable account audit and a portable CSV; a filename typo was detected and corrected during validation.


