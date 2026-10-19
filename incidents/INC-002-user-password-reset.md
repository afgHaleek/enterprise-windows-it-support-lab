# INC-002 — Active Directory User Password Reset

**Environment:** NovaTech Enterprise Windows IT Support Lab  
**Domain:** `corp.novatech.test` (`NOVATECH`)  
**Domain controller:** `DC01` — Windows Server 2025  
**Client:** `CLIENT01` — Windows 11, domain joined  
**Account:** `NOVATECH\sarah.murphy` (fictional Finance employee)  
**Status:** Password reset and mandatory-change prompt verified; successful sign-in after changing the password should be confirmed separately.

## 1. Ticket scenario

An employee reports that they cannot remember their domain password. The support technician must verify the account state, reset the password securely, require a password change at the next logon, and confirm the user can sign in.

> **Security:** In production, verify the requester's identity using the organization's approved procedure before resetting a password. Do not record passwords in screenshots, tickets, scripts or Git.

## 2. Initial account check

On `DC01`, inspect the account:

```powershell
Get-ADUser -Identity 'sarah.murphy' `
    -Properties Enabled,LockedOut,PasswordExpired,PasswordLastSet |
    Format-List Name,SamAccountName,Enabled,LockedOut,PasswordExpired,PasswordLastSet
```

The captured evidence confirms that `sarah.murphy` exists and is enabled. The screenshots do **not** establish the values of every additional property above.

## 3. Password reset procedure

Prompt for a temporary password without displaying it:

```powershell
$password = Read-Host 'Enter temporary password for Sarah' -AsSecureString
```

Reset the account password:

```powershell
Set-ADAccountPassword `
    -Identity 'sarah.murphy' `
    -Reset `
    -NewPassword $password
```

Require a password change at next logon:

```powershell
Set-ADUser `
    -Identity 'sarah.murphy' `
    -ChangePasswordAtLogon $true
```

**Why two commands?** `Set-ADAccountPassword` performs the administrative reset; `Set-ADUser -ChangePasswordAtLogon $true` sets the separate account requirement to choose a new password.

## 4. Verification

On `DC01`, inspect the underlying password-set attribute:

```powershell
Get-ADUser 'sarah.murphy' -Properties pwdLastSet |
    Select-Object Name,pwdLastSet
```

**Observed:** `pwdLastSet` was `0`. This indicates that the account must change its password at the next applicable logon.

On `CLIENT01`, Windows displayed the password-change form for Sarah Murphy. This confirms that the mandatory password-change requirement was presented to the user.

**Observation requiring further investigation:** The first reported sign-in after the reset did not request a password change; after signing out and trying again, Windows presented the change-password screen. The precise reason for the first behavior was **not established**. Potential factors include session state, cached authentication or timing. No root cause is claimed.

### Final verification to complete

After Sarah sets a new password and signs in successfully, run on `CLIENT01`:

```powershell
whoami
hostname
```

Expected: `novatech\sarah.murphy` and `CLIENT01`.

On `DC01`, verify the password-set timestamp:

```powershell
Get-ADUser 'sarah.murphy' -Properties PasswordLastSet |
    Select-Object Name,Enabled,PasswordLastSet
```

After a successful password change, `PasswordLastSet` should normally show a date/time. Record the actual results before marking the ticket fully resolved.

## 5. Domain password and lockout policy

Command run on `DC01`:

```powershell
Get-ADDefaultDomainPasswordPolicy |
    Format-List MinPasswordLength,ComplexityEnabled,MaxPasswordAge,LockoutThreshold,LockoutDuration,LockoutObservationWindow
```

**Observed lab values:**

| Setting | Value | Meaning |
| --- | --- | --- |
| `MinPasswordLength` | `7` | Minimum configured password length |
| `ComplexityEnabled` | `True` | Password complexity rules enabled |
| `MaxPasswordAge` | `42.00:00:00` | Maximum age: 42 days |
| `LockoutThreshold` | `0` | No account lockout threshold in the default domain policy |
| `LockoutDuration` | `00:10:00` | Configured duration: 10 minutes, not operative for threshold zero |
| `LockoutObservationWindow` | `00:10:00` | Configured observation window: 10 minutes |

**Implication:** Deliberately entering incorrect passwords will not produce an ordinary threshold-based lockout under this default policy. A later lockout exercise will require a controlled lab policy change. Fine-grained password policies, if configured, can change the effective policy for specific accounts.

## 6. Evidence

The image links below assume this Markdown file is saved in the repository's `incidents/` directory.

1. [Windows prompts Sarah to change password](../screenshots/active-directory/troubleshooting/01-sarah-password-change-prompt.png)
2. [DC01 reset commands and `pwdLastSet = 0`](../screenshots/active-directory/troubleshooting/02-password-reset-pwdlastset.png)
3. [Default domain password and lockout policy](../screenshots/active-directory/troubleshooting/03-default-domain-password-policy.png)

## 7. Lessons learned

- Password **reset** and **change at next logon** are separate administrative operations.
- A `pwdLastSet` value of `0` is expected when a mandatory password change is configured.
- A Windows password-change prompt proves the requirement was presented, but not that the password change and subsequent sign-in completed successfully.
- Always distinguish observed evidence from hypotheses about unexpected behavior.
- Inspect the actual domain lockout threshold before attempting a lockout simulation.

## 8. Next exercise

**INC-003 — Account Lockout Troubleshooting:** Inspect effective password policies, configure a controlled lab-only lockout threshold, reproduce an account lockout, examine its cause and verify recovery.
