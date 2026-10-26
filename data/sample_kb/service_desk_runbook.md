# Lumora Logistics — Service Desk Playbook

Version 2.4 · For Service Desk analysts (L1 and L2) · Owned by IT Operations

This playbook tells analysts how to handle the most frequent support requests. Follow the steps in order, record what you tried in the ticket, and escalate using the rules in each section.

---

## Ticket Priorities and Response Targets

Assign priority based on **impact** (how many people) and **urgency** (whether work is stopped).

| Priority | When to use it | First response | Target resolution |
|---|---|---|---|
| **P1 – Critical** | Outage affecting a whole site, the warehouse management system, or all users of a core service; any confirmed security breach | 15 minutes | 4 hours |
| **P2 – High** | A department or shift team cannot work and has no workaround | 30 minutes | 8 hours |
| **P3 – Medium** | One person cannot work, or a team is slowed but has a workaround | 4 hours | 3 working days |
| **P4 – Low** | Requests, questions, new software, minor issues with no business impact | 1 working day | 10 working days |

Rules:

- Anything involving a possible data breach, ransomware, or a stolen device starts at **P1** until Security says otherwise.
- Warehouse scanners and dispatch systems are business-critical: a full warehouse shift unable to scan is **P1**, not P2.
- Senior job titles do not raise priority on their own. Priority follows impact.
- If three or more tickets describe the same symptom within 30 minutes, link them to a single parent incident and raise the parent to P2 or higher.

---

## Email and Calendar Issues

First, find out the scope: **one person, or many?**

**Many users affected**

1. Check the Lumora Service Health page and the vendor status page for Outlook/Exchange.
2. Open a parent incident and link incoming tickets to it.
3. Post an update in the IT Status channel.
4. Do **not** ask individual users to reinstall Outlook, clear profiles, or reset passwords during a wider outage. This wastes their time and creates more tickets.

**One user affected**

1. Ask the user to try Outlook on the web. If the web version works, the problem is the desktop app, not the mailbox.
2. Check mailbox storage. Above 95% full, sending stops; ask the user to archive or empty Deleted Items.
3. Have the user quit Outlook fully, sign out of Office, and sign back in.
4. Check whether the user's password expired recently. Expired passwords often show up as repeated sign-in prompts.
5. If none of these help, repair the Office installation from the Lumora App Store.

Escalate to the **Messaging team (L2)** when mail flow rules, shared mailboxes, or delegated access are involved.

---

## Slow or Unresponsive Laptop

Gather the basics before changing anything: laptop model, age, when the slowness started, and whether it happens on the office network, at home, or both.

Check, in this order:

1. **Disk space:** less than 10% free space causes major slowdowns. Help the user clear Downloads and move files to Lumora Drive.
2. **Memory and CPU:** open Task Manager (Windows) or Activity Monitor (macOS) and look for a single application using most of the resources.
3. **Startup apps:** disable non-essential startup items. Do not disable anything owned by IT or Security.
4. **Updates:** a pending operating system update or restart is a common cause. Ask the user to save their work and restart.
5. **Security scan:** check whether a full antivirus scan is running. If so, it will finish; do not stop it.

Never uninstall or pause security software to speed up a laptop, even temporarily.

Escalate to **Desktop Engineering (L2)** when:

- hardware diagnostics report a disk, battery, or memory fault,
- the laptop shuts down, freezes, or shows a blue/black screen more than twice in a week after the steps above,
- the laptop is over four years old and below the performance baseline (offer a replacement request instead).

---

## Remote Access (Lumora Connect) Problems

1. Confirm the user's internet works without Lumora Connect.
2. Ask which region they selected and whether switching region helps.
3. Check the Service Health page for gateway issues in that region.
4. Check that the user's account is not locked and their password has not expired.
5. If on hotel or airport Wi-Fi, suggest a phone hotspot.

If a whole region is failing for multiple users, treat it as a parent incident and escalate to the **Network team** immediately.

---

## Account Lockouts and Password Resets

Before any reset, verify identity using **two** of the following:

- employee ID,
- a callback to the phone number registered in HR records (never a number the caller gives you),
- manager confirmation through a Help Portal approval.

Then:

- If the account is only locked, wait for the 30-minute auto-unlock or unlock it manually in the identity console.
- If the user forgot the password, issue a one-time temporary password through the identity console. It must be changed at next sign-in.
- Read the temporary password to the user over the verified callback only. Never send it by email or chat.

Never ask a user for their current password or a Lumora Verify code. If repeated lockouts happen without the user signing in, their password may be compromised: force a reset and notify Security.

---

## Suspected Phishing or Compromised Account

Treat every report seriously, even if the email looks harmless.

1. Ask whether the user only received the email, clicked a link, opened an attachment, or entered their password.
2. **Received only:** confirm they used *Report Phish*, then close the ticket as P4.
3. **Clicked or opened:** raise to P2. Ask the user to disconnect from Wi-Fi and keep the laptop on. Hand over to Security.
4. **Entered credentials:** raise to P1. Force a password reset, revoke active sessions, reset Lumora Verify, and hand over to Security immediately.

Do not run scans or delete files on a possibly infected device; Security needs it in its current state.

---

## Lost or Stolen Devices

1. Raise the ticket at **P1**.
2. Record the asset tag, last known location, time noticed, and whether a police report was filed.
3. Lock the device in the device management console straight away.
4. Notify Security and the user's manager.
5. If not recovered within 48 hours, Security approves a remote wipe.
6. Arrange a loan device so the user can keep working.

---

## Escalation Contacts

| Team | Handles | How to reach |
|---|---|---|
| Messaging | Mail flow, shared mailboxes, calendars | Help Portal queue *MSG-L2* |
| Desktop Engineering | Hardware faults, rebuilds, replacements | Help Portal queue *DESK-L2* |
| Network | Lumora Connect gateways, office Wi-Fi, site outages | Help Portal queue *NET-L2*; phone bridge for P1 |
| Security | Phishing, compromised accounts, lost devices, data incidents | security@lumora.example; on-call phone for P1 |
| Warehouse Systems | Scanners, dispatch and inventory applications | Help Portal queue *WMS-L2*; on-call phone for P1 |

When escalating, include: what the user reported, what you checked, what you changed, and the current impact. A ticket escalated with no notes will be returned to L1.

---

## Closing a Ticket

Before closing:

- confirm with the user that the issue is resolved, or record two contact attempts over two working days,
- write a short resolution note another analyst could follow next time,
- tag the ticket with the matching category so it appears in monthly reports.