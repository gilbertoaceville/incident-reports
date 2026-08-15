# Incident Report: Monday Morning Badge Lockout — Access Control Panels Operating on Stale Cache

**Status:** Draft
**Date of incident:** 15–17 August 2026 (degradation began Sat 15 Aug 02:24; user impact Mon 17 Aug 06:47)
**Author:** CA
**Severity:** SEV2 — access-control synchronization degraded campus-wide for ~53 hours, undetected

> **Reporting note.** The author arrived on site at 07:10 Monday, approximately 53 hours after this incident began. Everything prior is reconstructed from server logs and staff accounts; timeline entries are source-tagged, and conclusions not supported by evidence are marked as assumptions or open questions.

---

## Summary

A routine monthly patch reboot of the access-control server at 02:23 on Saturday 15 August left the panel-communication service stopped. No alert was raised and the failure went undetected for approximately 53 hours. Door controllers continued operating on their cached local access list, so the ~380 existing badge-holders were unaffected — but the 20 badges provisioned on Friday afternoon, plus one access level changed the previous Thursday, were absent from that cache and were rejected at every exterior door. Approximately 60 employees were delayed during Monday's arrival, queues formed at the Building A turnstiles, guards fell back to manual roster checks, and the plant manager escalated at 07:05. Starting the service manually at 07:22 restored synchronization; all affected badges were confirmed working by 07:41, ending roughly 54 minutes of user-visible disruption.

---

## Timeline

_All times are site-local. **Assumption:** the case pack gives weekdays but no calendar dates; dates below are mapped to Fri 14 – Mon 17 August 2026, which match the stated days of the week._

**Source key:** `Logged` = recorded by a system · `Reported` = account given by another person · `First-hand` = directly observed by the author

| Date | Time | Event | Source |
|------|------|-------|--------|
| Fri 14 Aug | 16:30 | HR processes 14 new hires and 6 badge replacements | Reported |
| Fri 14 Aug | 16:30–17:50 | Access admin provisions all 20 badges on the access-control server | Reported |
| Fri 14 Aug | 17:50 | One new badge tested at the Building A lobby reader — accepted | Reported |
| Sat 15 Aug | 02:00 | Monthly Windows-update window begins on the access-control server | Logged |
| Sat 15 Aug | 02:23 | Server reboots | Logged |
| Sat 15 Aug | **02:24** | **Panel-communication service fails to start — "dependent service not yet available"** | Logged |
| Sat 15 Aug | 02:26 | Database service completes startup | Logged |
| Sat 15 – Mon 17 | 02:26 – 07:22 | **No panel synchronization for approximately 53 hours. Door controllers operate on local database. No alert raised** | Logged |
| Sat 15 – Sun 16 | — | Guards badge through exterior doors; no rejections reported | Reported |
| Mon 17 Aug | 06:47 | Day-shift arrivals — some employees accepted, others rejected at every exterior door | Reported |
| Mon 17 Aug | 06:47 | Queue forms at Building A turnstiles; guards begin manual ID checks against the roster | Reported |
| Mon 17 Aug | 07:02 | Lobby guard identifies the rejected population: new hires, recent badge replacements, and one Thursday access-level change | Reported |
| Mon 17 Aug | 07:05 | Escalation to the plant manager | Reported |
| Mon 17 Aug | 07:10 | Author arrives on site | First-hand |
| Mon 17 Aug | 07:15 | Door controllers report "online — operating on local database" | First-hand |
| Mon 17 Aug | 07:15 | Server logs reviewed; 02:24 Sat panel-communication service failure identified | First-hand + Logged |
| Mon 17 Aug | 07:22 | Panel-communication service started manually; panels begin synchronizing | First-hand |
| Mon 17 Aug | 07:41 | All previously rejected badges confirmed working | First-hand |

### Notes on the timeline

_The table records events only; the observations below are analysis drawn from it._

**Key intervals**

| Interval | Duration | |
|---|---|---|
| Failure → detection | ~53 hours | 02:24 Sat → 07:15 Mon |
| Detection → resolution | 26 minutes | 07:15 → 07:41 Mon |
| User-visible disruption | ~54 minutes | 06:47 → 07:41 Mon |

Response was fast; detection was not. Remediation targets the first interval.

**What the Friday 17:50 test establishes.** That one badge was accepted at one reader at that time. It does not establish that the other 19 badges were tested, that other readers were tested, or that the panels held the new data independently of the server. Why the badge passed on Friday and failed on Monday is **not resolved by the available evidence** — the panel may never have received the download, or may not have retained it through the 02:24 failure. These are different faults with different fixes; see Remediation item 6.

**Open question.** The Thursday access-level change was also rejected, though a snapshot taken Friday evening should have contained it. Either the cached snapshot predates Thursday — which would place the start of degradation earlier than 02:24 Saturday — or the change reached the server after the last successful sync. Unresolved; see Remediation item 8.

---

## Impact

**Who was affected.** Two populations, and the difference between them explains the 53 hours.

- **Unaffected — approximately 380 existing badge-holders,** including every guard on weekend patrol. Anyone already present in the cached list badged in normally throughout. The system appeared healthy to almost everyone who used it.
- **Affected — 21 credentials, approximately 60 people delayed.** The 14 new hires, 6 badge replacements and 1 changed access level were rejected at every exterior door. The wider figure includes those queued behind them and those delayed by manual checks. **Assumption:** ~60 is as reported on the day and was not formally counted.

**Duration.**

| Measure | Duration |
|---|---|
| User-visible disruption | ~54 minutes (06:47–07:41 Mon) |
| System degradation | ~53 hours (02:24 Sat – 07:22 Mon) |

The 54 minutes is only the portion of the degradation window that coincided with people arriving for work — impact was bounded by the arrival schedule, not by anything the system or the process did.

**Operational impact.**

- Queues at the Building A turnstiles during peak arrival
- Guards diverted from patrol to manual ID verification against a printed roster — slower throughput, and a materially weaker check than a badge read
- Escalation to the plant manager at 07:05

**Security exposure.** A stale access list fails in both directions, and only one of them is visible. Credentials *added* after the last synchronization were absent, so legitimate people were refused. Any credential *revoked or downgraded* after the last synchronization would equally have been absent — meaning the panels would have continued to honour it for the full 53 hours, with nothing to indicate a problem, because a badge working is not an event anyone investigates.

**Did this materialise? Unknown — not established.** No revocations are recorded during the affected window, but no check was performed; see Remediation item 7. The window carried a latent exposure whose realisation depends on whether any revocation fell inside it. This is worth stating plainly because the loud symptom was false rejection, while the silent one would have been false acceptance.

**Not affected.** Doors continued to operate — the cached-database design is what prevented a failed server from locking down three buildings. The server database was undamaged and no provisioning work was lost; the 20 badges were correctly configured throughout and required no re-issue.

**Unknown / requires verification.**

- Whether any credential was revoked or downgraded between the last successful sync and 07:22 Monday
- Whether panel-side access transactions recorded during the window were buffered and uploaded on reconnection, or lost — if lost, the campus has a 53-hour audit gap
- The exact time of the last successful synchronization, which bounds the true start of degradation

---

## Root Cause

**Trigger.** The scheduled monthly Windows-update reboot of the access-control server at 02:23 Saturday — a routine, expected, correctly-scheduled maintenance action.

**Root cause.** Nothing in the system's configuration or operating procedure was capable of noticing that a routine reboot had left it degraded. The reboot is not the fault; it will recur next month by design. The fault is that a two-minute service-startup race was allowed to become a 53-hour silent outage.

**1. Service dependency not enforced, and no retry.** The panel-communication service failed at 02:24 with *"dependent service not yet available."* The database service it required completed startup at 02:26 — two minutes later — and no further start was attempted. A declared service dependency, or any restart-on-failure policy, would have recovered this automatically and produced no incident.

**2. No alerting on the degraded state.** The server had a stopped service; the panels reported *"online — operating on local database"* — precisely the signal that synchronization has failed. It was visible on the monitoring workstation for 53 hours with no alarm configured against it. **Layer 1 caused the failure; this layer caused the duration.**

**3. No post-patch verification.** The monthly patch window runs unattended at 02:00 Saturday, at a site with no weekend post-patch procedure. An automated change was made to a security-critical system and nothing confirmed it came back correctly. Detection was left to chance, and the chance that arrived was a queue on Monday morning.

**4. Verification tested the wrong layer.** Friday's provisioning was treated as verified on the strength of one badge at one reader. Whatever the underlying fault proves to be (see timeline note), a test of that scope could not have detected the condition that caused Monday's failure.

**Not a root cause: the cached-database design.** Local caching is why 380 people entered normally and why a failed server did not lock down a campus. The defect is that panels can run on cache *indefinitely and silently* — not that they do so at all. Remediation must not reduce panel autonomy.

---

## Corrective Actions

- **07:00–07:41 — mitigation.** Guards admitted rejected employees via manual ID checks against a printed roster. This addressed access, not the fault, and offered a weaker identity check than a badge read.
- **07:22 — resolution.** Panel-communication service started manually. Panels synchronized immediately; no restart, reconfiguration or re-provisioning was required, confirming server-side badge data had been correct throughout.
- **07:41 — verification.** Previously rejected badges re-tested and confirmed working; controllers confirmed returned to server-synchronized operation.

Everything above is reactive. A repeat of the same patch reboot tonight would produce the same outcome — the preventive work is entirely below.

---

## Remediation

Item 1 removes the failure mode rather than improving the odds against it. Item 2 is the backstop that makes any future loss of synchronization, from any cause, visible in minutes rather than days. Items 3–5 harden the process; items 6–8 close out unresolved questions.

| # | Action item | Type | Owner | Status | Due date |
|---|-------------|------|-------|--------|----------|
| 1 | Configure the panel-communication service to declare a hard dependency on the database service, with automatic restart-on-failure and retry. Validate by rebooting the server and confirming the service starts unattended | **Preventive** | [TBD] | Not started | [TBD] |
| 2 | Alert on loss of panel synchronization — trigger on any door controller reporting "operating on local database" for longer than 15 minutes, and on the panel-communication service being stopped. Must reach an on-call recipient outside business hours | **Preventive** | [TBD] | Not started | [TBD] |
| 3 | Add a mandatory post-patch verification step to the monthly maintenance procedure: confirm all services running and all panels server-synchronized, with named sign-off. Applies to weekend windows | Detective | [TBD] | Not started | [TBD] |
| 4 | Change badge-provisioning verification to confirm the credential has reached the panels, not only that a reader accepted it. Test at a minimum of two readers across different panels | Detective | [TBD] | Not started | [TBD] |
| 5 | Review whether the patch window should remain unattended at 02:00 Saturday, given ~53 hours can elapse before anyone is on site. Either move to attended hours or make item 2 a hard prerequisite for continuing unattended | Process | [TBD] | Not started | [TBD] |
| 6 | Establish with the vendor why the Friday-provisioned badge was accepted at 17:50 Friday and rejected Monday — whether the panel never received the download, or received and did not retain it | Open question | [TBD] | Not started | [TBD] |
| 7 | Determine whether any credential was revoked or downgraded between the last successful sync and 07:22 Monday, and whether it would have continued to be honoured. Closes the security-exposure unknown in Impact | Open question | [TBD] | Not started | [TBD] |
| 8 | Confirm whether panel-side transactions from the 53-hour window were buffered and uploaded, or lost — if lost, record the audit gap. Also resolve the Thursday access-level discrepancy, which may indicate the snapshot predates Friday | Open question | [TBD] | Not started | [TBD] |

**Done-condition:** items 1 and 2 implemented and validated by a test reboot that recovers unattended and alerts correctly when synchronization is deliberately interrupted; items 6–8 answered and their findings recorded here; one monthly patch cycle completed with post-patch verification signed off.
