# Incident Report: [Title]

**Status:** Draft | In Review | Final
**Date of incident:** YYYY-MM-DD
**Author:** [Name]
**Severity:** SEV1 | SEV2 | SEV3 | SEV4

---

## Summary

_A 2-4 sentence overview: what broke, for how long, who was affected, and how it was resolved. Someone should understand the whole incident from this paragraph alone._

## Timeline

_All times are site-local. **Assumption:** the case pack gives weekdays but no calendar dates; dates below are mapped to Fri 14 – Mon 17 August 2026, which match the stated days of the week._

**Source key:** `Logged` = recorded by a system · `Reported` = account given by another person · `First-hand` = directly observed by the author · `Inferred` = concluded from other evidence, not observed

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

_The table above records events only. The observations below are analysis drawn from it and are separated deliberately._

**Key intervals**

| Interval | Duration | |
|---|---|---|
| Failure → detection | ~53 hours | 02:24 Sat → 07:15 Mon |
| Detection → resolution | 26 minutes | 07:15 → 07:41 Mon |
| User-visible disruption | ~54 minutes | 06:47 → 07:41 Mon |

Response was fast; detection was not. Remediation should target the first interval.

**What the Friday 17:50 test does and does not establish.** The test confirms that one badge was accepted at one reader at that time. It does not confirm that the remaining 19 badges were tested, that other readers were tested, or that the door panels held the new badge data independently of the server. Why the badge was accepted on Friday and rejected on Monday is **not resolved by the available evidence** — the panel may never have received the download, or may have received it and not retained it through the 02:24 failure. These are materially different faults and should be established with the vendor rather than assumed. See Remediation.

**Open question — unresolved.** The employee whose access level changed on **Thursday** was also rejected. If the panels held a snapshot taken Friday evening, a Thursday change should have been present in it. Either the cached snapshot predates Thursday — which would place the start of degradation earlier than 02:24 Saturday and widen the window beyond 53 hours — or that change reached the server after the last successful sync. The case pack does not resolve this. See Remediation.
