# Lab 03 — Jira Service Management Help Desk + Active Directory + Splunk

> **Domain:** redblue / redblue.local
> **Network:** 192.168.10.0/24 — VirtualBox NAT Network
> **Hardware:** HP ENVY x360 i5-8250U 16GB RAM
> **Author:** Virginie Solomon-Bonhomme
> **Date:** May 2026
> **Status:** Complete

---

## Table of Contents

1. [Skills Demonstrated](#skills-demonstrated)
2. [What I Built](#what-i-built)
3. [My Environment](#my-environment)
4. [What I Configured](#what-i-configured)
5. [The Tickets I Worked](#the-tickets-i-worked)
6. [What I Found in Splunk](#what-i-found-in-splunk)
7. [Next Steps](#next-steps)
8. [What I Learned](#what-i-learned)

---

## Skills Demonstrated

- Setting up a Jira Service Management project from scratch
- Creating and configuring IT help desk request types
- Working a full ticket queue from creation to resolution
- Documenting every ticket with internal notes
- Performing Active Directory actions to resolve tickets
- Resetting passwords and unlocking accounts via ADUC
- Onboarding and offboarding users in Active Directory
- Processing access requests with documented approval
- Escalating security incidents with Splunk evidence
- Running emergency PowerShell remediation across all domain accounts
- Verifying every action in Splunk using Windows Security Event logs
- Connecting SIEM evidence directly to help desk tickets

---

## What I Built

Lab 03 is the final lab in a three part series. In Lab 01 I built a Splunk SIEM and detected real attacks against my Active Directory domain. In Lab 02 I built the identity infrastructure — OUs, security groups, and user accounts. In this lab I built a professional IT help desk using Jira Service Management and worked seven tickets that connect directly to the work done in the previous two labs.

Every ticket has three parts — the Jira ticket documenting the request, the Active Directory action that resolves it, and the Splunk EventCode confirming the action was logged automatically. This three part workflow is how a real IT help desk operates.

The ticket queue starts with routine everyday requests and builds to full Critical security incident tickets referencing the brute force attack and credential dumping events from Lab 01.

---

## My Environment

| VM | OS | IP | Role |
|---|---|---|---|
| Splunk | Ubuntu Server 22.04 | 192.168.10.10 | SIEM — audit trail verification |
| ADDC01 | Windows Server 2022 | 192.168.10.7 | Domain Controller — identity changes |
| Target-PC | Windows 10 | 192.168.10.100 | Domain workstation |
| Host Machine | Windows 11 | — | Jira Service Management browser access |

---

## What I Configured

### Jira Service Management Setup

I created a free Jira Service Management account at atlassian.com and set up a project using the IT Service Management template.
Project name: RedBlue IT Help Desk
Project key:  RIHD
Workspace:    RedBlue IT Operations

The IT Service Management template gave me built in queues, SLA timers, and request type categories out of the box. I customized the request types to match my lab scenarios:

| Request Type | Category |
|---|---|
| New Employee Onboarding | Service Request |
| Employee Offboarding | Service Request |
| Password Reset | Service Request |
| Account Lockout | Service Request |
| Access Request | Service Request |
| Hardware Issue | Service Request |
| Security Incident | Incident |

---

### New Users Added to Active Directory

Before working tickets I created five new employees in Active Directory via ADUC to use as routine ticket subjects:

| Name | Username | OU | Group |
|---|---|---|---|
| Marcus Thompson | m.thompson | Finance | Finance-ReadOnly |
| Priya Patel | p.patel | HR | HR-ReadOnly |
| Daniel Nguyen | d.nguyen | IT | IT-Admins |
| Aaliyah Brooks | a.brooks | Management | Management-ReadOnly |
| Carlos Rivera | c.rivera | Finance | Finance-ReadOnly |

Each user was created with must change password at next logon enabled.

---

## The Tickets I Worked

### Ticket 01 — New Hire Onboarding — Marcus Thompson

Marcus Thompson joined the Finance department and needed his AD account confirmed and group access assigned.

**What I did in Jira:**
Created a Service Request ticket with Medium priority. After confirming the AD account and group membership I added an internal note with the resolution details and resolved the ticket.

**What I did in ADUC:**
Confirmed m.thompson existed in the Finance OU and was a member of Finance-ReadOnly.

**What Splunk showed:**
EventCode 4720 confirmed the account creation was logged automatically.

```splunk
index=endpoint host=ADDC01 EventCode=4720
| table _time, EventCode, Account_Name, ComputerName
| sort -_time
```

<img width="1916" height="861" alt="Screenshot 2026-05-26 132810" src="https://github.com/user-attachments/assets/7dcbd65c-6e43-4024-b39b-22b11ca9b3c2" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_12_52_44" src="https://github.com/user-attachments/assets/556083a0-676f-44fb-bb33-5e80fa696d2a" />
<img width="1909" height="865" alt="Screenshot 2026-05-26 134921" src="https://github.com/user-attachments/assets/eaa7e6e4-53a1-45b4-b1d9-8d7815c3792f" />



---

### Ticket 03 — Password Reset — Priya Patel

Priya Patel could not log into her domain account and needed a password reset.

**What I did in Jira:**
Created a Service Request ticket with Medium priority. I documented that identity was verified before taking any action.

**What I did in ADUC:**
Right clicked Priya Patel in the HR OU, selected Reset Password, set a temporary password, and checked must change password at next logon.

**What I learned about EventCodes here:**
I initially searched for EventCode 4723 and got no results. After investigating I learned that 4723 is generated when a user changes their own password. When an admin resets a password it generates EventCode 4724 instead. I updated my documentation to reflect the correct EventCode.

**What Splunk showed:**
EventCode 4724 confirmed the admin password reset was logged.

```splunk
index=endpoint host=ADDC01 EventCode=4724
| table _time, EventCode, Account_Name, ComputerName
| sort -_time
```


<img width="1916" height="865" alt="Screenshot 2026-05-26 141705" src="https://github.com/user-attachments/assets/dc2d4c83-b4e2-4589-bc80-076110c7ef0d" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_14_20_11" src="https://github.com/user-attachments/assets/e2a7c4c2-7573-490f-92ee-110a5fa7e066" />
<img width="1909" height="972" alt="Screenshot 2026-05-26 152822" src="https://github.com/user-attachments/assets/f111d716-e0b0-43d3-b4e9-ffe64d3fb398" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_15_22_20" src="https://github.com/user-attachments/assets/c0b1f6d4-d8ba-4eed-9be0-f455da5319a9" />





---

### Ticket 06 — Offboarding — Tom Harris

Tom Harris left the company and his account needed to be formally offboarded. His account had already been disabled and moved to the Disabled OU during Lab 02. This ticket documents that action formally.

**What I did in Jira:**
Created a Service Request ticket with High priority documenting the offboarding request, the 30 day retention policy, and the scheduled deletion date.

**What I did in ADUC:**
Confirmed t.harris was in the Disabled OU with the account disabled. Confirmed group memberships had been removed.

**What Splunk showed:**
EventCode 4725 confirmed the account disable was logged.

```splunk
index=endpoint host=ADDC01 EventCode=4725
| table _time, EventCode, Account_Name, ComputerName
| sort -_time
```

<img width="1919" height="922" alt="Screenshot 2026-05-26 191344" src="https://github.com/user-attachments/assets/aaf52367-4dc0-4356-93d7-ad73d60c6ba6" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_19_02_22" src="https://github.com/user-attachments/assets/0544af8b-8750-475f-ac2e-21fec52c9107" />
<img width="1903" height="864" alt="Screenshot 2026-05-26 191937" src="https://github.com/user-attachments/assets/e926bb23-412b-4d6d-b0d4-a4fca297950b" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_19_06_42" src="https://github.com/user-attachments/assets/cdf4ffd9-4519-49b0-8b03-d07960c1f77b" />



---

### Ticket 07 — Access Request — James Wilson

James Wilson in Management requested read access to Finance resources for quarterly reporting. This ticket required documented approval before any AD action was taken.

**What I did in Jira:**
Created a Service Request ticket with Medium priority. Before touching Active Directory I added an internal note documenting the approval — who approved it and when. This is the most important step in any access request. Access is never granted without documented approval.

**What I did in ADUC:**
Only after the approval was documented in Jira did I go to ADUC and add j.wilson to the Finance-ReadOnly group via the Member Of tab.

**What Splunk showed:**
EventCode 4728 confirmed the group membership addition was logged.

```splunk
index=endpoint host=ADDC01 EventCode=4728
| table _time, EventCode, Account_Name, ComputerName
| sort -_time
```



<img width="995" height="708" alt="Screenshot 2026-05-26 192709" src="https://github.com/user-attachments/assets/2c48c1b0-7aa2-455e-aa9a-913b6cd8e8fd" />
<img width="1910" height="864" alt="Screenshot 2026-05-26 193013" src="https://github.com/user-attachments/assets/630f92d1-f66e-4840-afba-cdd37f499ae0" />
<img width="1915" height="864" alt="Screenshot 2026-05-26 194042" src="https://github.com/user-attachments/assets/2427787a-2fc8-41c1-b6a0-e5bc3b1fbe08" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_19_35_23" src="https://github.com/user-attachments/assets/a6263d73-bf73-42e0-a56b-c850fea79e9c" />





---

### Ticket 09 — Brute Force Attack — Steven Williams

While reviewing Splunk logs I discovered a brute force attack against the s.williams account on Target-PC. This was not reported by a user — I found it myself during a manual Splunk investigation.

**What I did in Jira:**
Created a Critical Incident ticket. I gathered all evidence from my own systems before taking any action in Active Directory.

**What my own logs showed me:**
I did not need access to the attacker's system to investigate. Windows automatically recorded everything in the Security Event logs:

```splunk
index=endpoint EventCode=4625
Account_Name=swilliams
| table _time, Account_Name, ComputerName, 
  Logon_Type, Source_Network_Address
| sort -_time
```

The raw event showed:
- Source Network Address: 192.168.10.250
- Workstation Name: kali — recorded automatically by Windows
- Logon Type 3 — the attack came over the network
- Status 0xC000006A — wrong password being tried repeatedly
- 47 total failed attempts

This taught me something important. I always assumed you needed access to the attacker's machine to know where an attack came from. Windows records this information automatically in every failed logon event. My own logs told me everything.

**What I did in ADUC:**
Confirmed s.williams was already disabled from prior IR-001 actions.

**What I built after:**
DET-002 Brute Force Failed Logons alert was built after finding this attack manually so it would be caught automatically in future.


<img width="1911" height="861" alt="Screenshot 2026-05-26 211150" src="https://github.com/user-attachments/assets/935362e2-dc73-4894-ac29-c20ea85f505f" />
<img width="1917" height="848" alt="593775834-4a3bd3a0-f4ed-4b6f-9d7f-6d892a5444bf" src="https://github.com/user-attachments/assets/0b087c3d-3a65-495e-a4a6-90059a445e3e" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_20_43_43" src="https://github.com/user-attachments/assets/ed07bedb-7ed1-4a8d-91c7-f1b7a48192d2" />
<img width="1280" height="720" alt="VirtualBox_ADDC01_26_05_2026_21_30_18" src="https://github.com/user-attachments/assets/4b9ce030-04d4-4d28-ab65-92c12d556fee" />
<img width="1918" height="971" alt="Screenshot 2026-05-26 213603" src="https://github.com/user-attachments/assets/c753fa8a-5e40-41ce-8c8b-8ef7c510918d" />
<img width="1913" height="926" alt="Screenshot 2026-05-26 213636" src="https://github.com/user-attachments/assets/e12030ee-7f76-4332-92c2-db0a03660f4e" />


---

### Ticket 10 — T1003 Credential Dumping — All Domain Accounts

While searching Splunk I found a Sysmon event showing that a process had accessed lsass.exe on ADDC01 with full memory permissions. This was the most serious event in the entire lab.

**What I did in Jira:**
Created a Critical Incident ticket referencing IR-001. I gathered Splunk evidence first before taking any action.

**What Splunk showed:**

```splunk
index=endpoint host=ADDC01
sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
technique_id=T1003
| table _time, host, technique_id, technique_name
| sort -_time
```

The raw Sysmon event showed:
- host: ADDC01
- technique_id: T1003
- technique_name: Credential Dumping
- TargetImage: C:\Windows\system32\lsass.exe
- GrantedAccess: 0x1fffff — full memory access

**What I did in PowerShell — three phases:**

Phase 1 — Containment — disabled all accounts:
```powershell
Get-ADUser -Filter * -SearchBase "DC=redblue,DC=local" |
Where-Object {
    $_.SamAccountName -notlike "Administrator" -and
    $_.SamAccountName -notlike "Guest" -and
    $_.SamAccountName -notlike "krbtgt"
} | ForEach-Object {
    Disable-ADAccount -Identity $_.SamAccountName
    Write-Host "DISABLED: $($_.Name)" -ForegroundColor Red
}
```

Phase 2 — Eradication — forced password reset on all accounts:
```powershell
Get-ADUser -Filter * -SearchBase "DC=redblue,DC=local" |
Where-Object {
    $_.SamAccountName -notlike "Administrator" -and
    $_.SamAccountName -notlike "Guest" -and
    $_.SamAccountName -notlike "krbtgt"
} | ForEach-Object {
    Set-ADAccountPassword -Identity $_.SamAccountName `
        -Reset `
        -NewPassword (ConvertTo-SecureString "TempPass2026!" -AsPlainText -Force)
    Set-ADUser -Identity $_.SamAccountName -ChangePasswordAtLogon $true
    Write-Host "PASSWORD RESET: $($_.Name)" -ForegroundColor Yellow
}
```

Phase 3 — Recovery — re-enabled all accounts except swilliams:
```powershell
Get-ADUser -Filter * -SearchBase "DC=redblue,DC=local" |
Where-Object {
    $_.SamAccountName -notlike "Administrator" -and
    $_.SamAccountName -notlike "Guest" -and
    $_.SamAccountName -notlike "krbtgt" -and
    $_.SamAccountName -notlike "swilliams"
} | ForEach-Object {
    Enable-ADAccount -Identity $_.SamAccountName
    Write-Host "RE-ENABLED: $($_.Name)" -ForegroundColor Green
}
```

swilliams remained disabled — already compromised in the brute force incident.

**What Splunk confirmed:**
```splunk
index=endpoint host=ADDC01
(EventCode=4725 OR EventCode=4724 OR EventCode=4722)
| table _time, EventCode, Account_Name, ComputerName
| sort -_time
```

Every action was automatically logged.

*Screenshot: Ticket 10 open — Critical priority*
*Screenshot: Splunk T1003 raw Sysmon event*
*Screenshot: PowerShell red DISABLED output*
*Screenshot: PowerShell yellow PASSWORD RESET output*
*Screenshot: PowerShell green RE-ENABLED output*
*Screenshot: Splunk showing all IR actions logged*
*Screenshot: Ticket 10 resolved with IR-001 referenced*

---

## What I Found in Splunk

Every action I took in Active Directory generated a Windows Security Event that Splunk collected automatically. Here is the complete reference for every EventCode that appeared across all seven tickets:

| EventCode | What It Means | Which Ticket |
|---|---|---|
| 4720 | Account created | Tickets 01 and 02 |
| 4724 | Admin password reset | Ticket 03 |
| 4725 | Account disabled | Ticket 06 |
| 4728 | Added to security group | Ticket 07 |
| 4625 | Failed logon attempt | Ticket 09 |
| 4722 | Account enabled | Ticket 10 |
| Sysmon EID 10 | Process accessed lsass | Ticket 10 |

### Complete Lab 03 Audit Trail

```splunk
index=endpoint host=ADDC01
(EventCode=4720 OR EventCode=4722 OR EventCode=4724
OR EventCode=4725 OR EventCode=4728)
| table _time, EventCode, Account_Name, ComputerName
| sort -_time
```

*Screenshot: Complete Lab 03 audit trail in Splunk*

---

## Next Steps

I plan to build DET-003 in Splunk as a dedicated Critical severity alert for lsass access events, closing the detection gap identified during the T1003 investigation. I also want to add a Group Policy lab showing how security policies are enforced at the OU level, which connects directly to the identity infrastructure built in Lab 02.

---

## What I Learned

**Jira is how real IT teams track work.** Working through a proper ticket queue showed me that IT support is not just fixing problems — it is managing a documented workflow where every action is traceable. The ticket is the record. Without it the work did not happen as far as anyone else is concerned.

**Approval before access is the most important habit.** Ticket 07 was the most important ticket in the lab for my IAM career goal. The approval step — documenting who approved the access and when before touching Active Directory — is the control that separates professional identity management from just making changes whenever someone asks.

**Windows logs tell you more than you think.** During Ticket 09 I assumed I would need access to the attacker's machine to know where the attack came from. I was wrong. Windows automatically recorded the source IP address and the attacking machine name in every failed logon event. My own logs gave me everything I needed. That changed how I think about investigations.

**PowerShell is the right tool for bulk IR actions.** When Ticket 10 required disabling every domain account I could have done it one by one through ADUC. Three PowerShell commands handled it in seconds with a color coded output showing exactly what happened to each account. In a real incident that speed difference matters.

**The audit trail builds itself.** Every single action I took across all seven tickets appeared in Splunk automatically. I did not have to do anything extra. That means in a real environment there is no hiding what you did — good or bad. Working carefully and documenting thoroughly are not optional.

**EventCode 4723 vs 4724.** I made a mistake during Ticket 03 searching for EventCode 4723 when I should have been searching for 4724. EventCode 4723 fires when a user changes their own password. EventCode 4724 fires when an admin resets it. Catching that mistake and correcting it taught me more than getting it right the first time would have.

---

*Document ID: LAB-03-JIRA-HELPDESK-2026 | Author: Virginie Solomon-Bonhomme | Domain: redblue.local | Network: 192.168.10.0/24 | Date: May 2026*
