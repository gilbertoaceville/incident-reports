# Incident Report: SOC Operating With Alarm Notifications Muted Due to Chronic False-Positive Volume

**Status:** Draft
**Date of incident:** Condition discovered April 2026; duration prior to discovery unknown
**Author:** [Author name]
**Severity:** SEV2 — site monitoring effectively degraded for an extended, unmeasured period

---

## Summary

During an unstructured review of surveillance and access-control systems in April 2026, alarm notifications were found to be muted at SOC operator workstations — a response to sustained alarm volume of 700+ per day (~30/hour, roughly one every two minutes), the majority of it false positives from untuned video analytics and access-control thresholds. A muted SOC is functionally an unmonitored SOC: for an unknown period, the site had camera coverage but no reliable alerting on it. Retuning the analytics — tightening detection zones, retraining object classification, and rerouting non-critical events to silent logging — reduced volume to 50+ per day (~1/hour). Notifications can now be run un-muted.

---

## Timeline

> **Note on precision:** exact timestamps are not recoverable. Alarm volume was not being measured and the discovery was incidental rather than alert-driven, so this timeline is reconstructed to month granularity. The inability to produce an exact timeline is itself a finding — see Root Cause and Remediation.

| Date | Event |
|------|-------|
| Prior to March 2026 | Analytics and alarm thresholds left at commissioning defaults; alarm volume reaches 700+/day |
| Unknown | Operators mute alarm notifications at workstations. Not reported, not escalated, not logged |
| March 2026 | Author joins the team |
| April 2026 | Unstructured review of surveillance/access-control systems begins |
| April 2026 | Muted notification state discovered incidentally during review |
| April 2026 | Root-cause analysis: Door Held Open identified as top recurring alarm; analytics found firing on wind/tree movement and misclassifying stationary equipment as vehicles |
| Late April 2026 | Detection zones tightened; object classification retrained (~100 samples); non-critical events rerouted to silent historical logging |
| Late April 2026 | Post-change volume confirmed at 50+/day (~1/hour) |
| [TBD] | Notifications un-muted and verified — **not yet confirmed complete** |

---

## Impact

- **Affected personnel:** 6 SOC operators, all workstations
- **Systems affected:** 50+ cameras (Avigilon Control Center analytics), ~16 doors (OnGuard access control)
- **Site:** Redacted
- **Duration of impact:** Unknown. The muted state predates the reviewer's arrival in March 2026 and its start was never recorded
- **Missed events:** **Unknown — not reviewed.** Alarm response was not measured, so it cannot be established whether genuine events were missed while notifications were muted. Door Forced Open is the alarm type of greatest concern, as it is both security-relevant and was subject to the same suppression. A retrospective review is required to close this out — see Remediation
- **Data loss or corruption:** None. Events continued to be recorded to historical logs throughout; only active notification was suppressed
- **Nature of exposure:** Detection capability was intact; alerting was not. The site retained forensic (after-the-fact) coverage but lost effective real-time response for the duration

---

## Root Cause

The trigger for the muted state was alarm volume. The root cause is that nothing in the system's design or operation was responsible for keeping that volume survivable.

Three contributing layers:

**1. No tuning at commissioning.** Analytics and alarm thresholds appear never to have been fine-tuned at install. Detection zones included the public roadway beyond the fence and a tree line that moved in wind; object classification misread stationary equipment as vehicles; the Door Held Open threshold sat at a 15-second default on high-traffic doors where 15 seconds is normal use. Each produced steady false positives from day one. These were shipped defaults left in place, not a configuration that drifted.

**2. No owner for alarm health.** Alarm tuning was not assigned to any role. Configuration authority in OnGuard — thresholds, event routing — sat outside the SOC, while the daily experience of the alarm volume sat inside it, and no process connected the two. Operators worked the alarms as configured and had no mechanism to change them or to register that the volume had become unworkable. Muting was the one lever available at the workstation, and it was a reasonable response to an unworkable alarm load. The failure is that the configuration was allowed to reach that state and stay there unowned.

**3. No alarm-volume metric.** Volume was not tracked, so there was no number to escalate, no threshold that triggered action, and no record of when the condition began. 700 alarms per day persisted indefinitely because nothing in the process was capable of noticing it.

Layer 1 explains the false positives. Layers 2 and 3 explain why they went unaddressed for an unknown length of time — and those are the ones that generalise to every other site and install.

---

## Corrective Actions

**Completed:**

- **Tightened detection zones.** Inclusion zones restricted to facility property and immediate environs, excluding the public roadway and the wind-affected tree line. This was the highest-yield single change
- **Retrained object classification.** Approximately 100 samples supplied by example to correct misclassification of stationary equipment as vehicles. Post-training classification accuracy verified as correct
- **Scoped person-only classification to office areas.** Alerting is filtered to people within the offices. The factory floor retains full detection including vehicles, where vehicle presence remains a legitimate event
- **Rerouted non-critical events to silent historical logging.** Removed from pop-up and audible notification; still recorded and available for review

**Result:** alarm volume reduced from 700+/day to 50+/day, and from ~30/hour to ~1/hour — roughly a 93% reduction. At approximately one alarm per hour, notifications are viable un-muted and genuinely unusual activity is distinguishable from background.

---

## Remediation

| # | Action item | Owner | Status | Due date |
|---|-------------|-------|--------|----------|
| 1 | Confirm notifications un-muted on all 6 operator workstations; verify they remain on at follow-up | [TBD] | Not started | [TBD] |
| 2 | Extend Door Held Open threshold from 15s to 45–60s on high-traffic doors | [TBD] | Not started | [TBD] |
| 3 | Retrospectively review Door Forced Open events during the muted period to establish whether real events were missed; close out the "unknown" in Impact | [TBD] | Not started | [TBD] |
| 4 | Walk-test tightened detection zones to confirm no coverage gaps were introduced by the exclusions | [TBD] | Not started | [TBD] |
| 5 | Add analytics tuning and an alarm-volume acceptance check to the commissioning checklist, so no camera or door goes live on defaults | [TBD] | Not started | [TBD] |
| 6 | Assign a named owner for monthly alarm-volume review, with a defined threshold that triggers retuning (proposed: >100/day) | [TBD] | Not started | [TBD] |
| 7 | Review OnGuard permissions so operators can flag or escalate alarm noise even where they cannot change configuration themselves | [TBD] | Not started | [TBD] |

**Done-condition for this incident:** items 1–3 closed, and one month of volume data showing sustained ≤100 alarms/day with notifications un-muted.
