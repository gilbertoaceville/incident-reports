# Questions

Things I don't know yet. Move an item to **Answered** once it's resolved — don't delete it, the answer is the useful part.

## Open

- [ ] _(add your next question here)_

## Answered

- [x] What's the difference between the root cause and the trigger?
  → The trigger is what happened right before things broke. The root cause is why that was enough to break them.

  Say a bad config change takes the site down. The config change is the trigger. The root cause is that nothing checked the config before it went live. Someone else could push a different bad config tomorrow and you'd be down again.

  Fix the trigger and you fix today. Fix the root cause and you fix every version of today.

- [x] Why must the timeline be exact?
  → Because the gaps between the timestamps are the whole point.

  How long before anyone noticed? How long from noticing to fixing? You can only answer those if the times are real. "Sometime that morning" tells you nothing to improve.

  Exact times also let you line the incident up against your logs and deploys later, and settle arguments about what happened first, which memory gets wrong all the time.

- [x] What makes a recommendation useful instead of vague?
  → It says who does what, and how you'll know it's done.

  Vague: "improve deployment safety."
  Useful: "Add a staging check to the payments deploy — Jamie — done when a bad config gets blocked before it reaches production."

  The test: hand it to someone who wasn't in the incident. If they know what to build and when to stop, it's a real action item. If not, it's a wish.

---

**Format:** state the question, then who or what could answer it.

```
- [ ] Why does X happen when Y? → ask Sam / check the logs
```
