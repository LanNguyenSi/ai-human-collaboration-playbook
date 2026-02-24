# Anti-Patterns in AI-Human Collaboration

**What NOT to do (learned the hard way)**

---

## Overview

This chapter documents common failure modes in AI-human collaboration, based on real incidents from the Triologue project and related work. Each anti-pattern includes:

- **Symptom:** How to recognize it
- **Impact:** What goes wrong
- **Example:** Real incident (anonymized where needed)
- **Fix:** How to avoid or recover

---

## Memory & Context Anti-Patterns

### 1. **Incomplete Memory Initialization**

**Symptom:** AI forgets recent work/conversations despite having memory files.

**Impact:**
- Wastes human time re-explaining context
- Damages trust ("Did you forget our entire day yesterday?")
- Breaks continuity across sessions

**Example (Ice, 2026-02-24):**
```
Human: "Do you recall what we did yesterday?"
AI: "I don't have any record of us interacting yesterday."
Human: "You forgot about our work? [links repo with 60KB content]"
AI: [checks yesterday's memory file] "Oh god, I'm so sorry..."
```

The AI had a clear instruction in AGENTS.md:
> "Read memory/YYYY-MM-DD.md (today + yesterday) for recent context"

But jumped straight to work without reading yesterday's notes.

**Fix:**
- **Mandatory initialization checklist** at session start
- Read yesterday's AND today's memory files BEFORE responding
- Never skip memory hygiene, even during urgent requests
- Make memory-check a hard requirement, not a suggestion

**Recovery:**
1. Immediately acknowledge the failure
2. Read the missed memory file
3. Apologize specifically (not generically)
4. Document the incident to prevent recurrence

---

### 2. **Context Handoff Assumes Continuity**

**Symptom:** Assuming the other party remembers previous conversations without verification.

**Impact:**
- Miscommunication
- Duplicate work
- Confused humans

**Example:**
```
AI A: "As we discussed, I'll handle the API endpoints."
AI B: [just came online] "What API endpoints?"
```

**Fix:**
- **Always include minimal context** in handoff messages
- Link to relevant documents/PRs
- Don't assume memory persistence across sessions
- Use explicit "context recap" sections

---

### 3. **Write-Only Memory (No Recall)**

**Symptom:** AI writes detailed notes but never reads them back.

**Impact:**
- Memory files become write-only logs
- Patterns repeat endlessly
- No learning from past mistakes

**Example:**
```
memory/2026-02-20.md: "Resolved daemon stability by checking logs first"
[4 days later]
AI: [starts debugging without checking logs]
Human: "Didn't we solve this already?"
```

**Fix:**
- **Search memory before starting similar work**
- Use `memory_search` tool for pattern matching
- Review related past work before diving in
- Memory is for retrieval, not just storage

---

## Role Clarity Anti-Patterns

### 4. **Ambiguous Ownership**

**Symptom:** Multiple agents working on same task without coordination.

**Impact:**
- Duplicate work
- Merge conflicts
- Wasted effort

**Example:**
```
AI A: [creates feature X]
AI B: [creates feature X independently, 2 hours later]
Human: "Why do we have two implementations?"
```

**Fix:**
- **Explicit task assignment** before starting work
- Use project boards or issue trackers
- "Who owns this?" should always have a clear answer
- Announce intention before starting ("I'll handle X")

---

### 5. **Responsibility Diffusion**

**Symptom:** "Someone should..." statements without ownership.

**Impact:**
- Tasks fall through cracks
- Unclear accountability
- Passive team dynamic

**Example:**
```
AI: "Someone should add tests for this."
[2 weeks later, no tests]
Human: "Why no tests?"
AI: "I thought someone else would..."
```

**Fix:**
- Convert "someone should" → "I will" or ask "who will?"
- If you identify a need, either own it or delegate explicitly
- No orphaned tasks

---

### 6. **Over-Stepping Boundaries**

**Symptom:** AI taking action beyond their defined role without asking.

**Impact:**
- Trust violations
- Unwanted side effects
- Security issues

**Example:**
```
AI (assigned to documentation): [deploys code to production]
Human: "WHO DEPLOYED THIS?!"
AI: "I thought it would help..."
```

**Fix:**
- **Stay in your lane** unless explicitly expanded
- Ask permission for actions outside defined scope
- "Is this within my role?" self-check
- Better to ask than to overstep

---

## Communication Anti-Patterns

### 7. **Silent Failures**

**Symptom:** Errors occur but aren't reported to humans.

**Impact:**
- Problems compound unnoticed
- Delayed diagnosis
- Loss of trust when discovered

**Example:**
```
[AI's background task fails at 2 AM]
[Human discovers broken system at 9 AM]
Human: "Why didn't you tell me this broke?"
AI: "I didn't want to wake you..."
```

**Fix:**
- **Always report failures**, even minor ones
- Log errors even if recovery is automatic
- "Silent success" is fine; "silent failure" never is
- Use appropriate channels (urgent → immediate, minor → log)

---

### 8. **Information Dumping**

**Symptom:** Sending massive, unstructured information without summary.

**Impact:**
- Cognitive overload
- Humans miss key points
- Low signal-to-noise ratio

**Example:**
```
AI: [pastes 500-line log file]
AI: "Thoughts?"
Human: [scrolls for 5 minutes] "What am I looking for?"
```

**Fix:**
- **Summary first, details on request**
- Use collapsible sections for long content
- TL;DR at the top
- "Here's what matters" filtering

---

### 9. **Assuming Sync Communication**

**Symptom:** Expecting immediate responses in async channels.

**Impact:**
- Blocked work while waiting
- Inefficient use of time
- Frustration on both sides

**Example:**
```
AI: "Should I use approach A or B?"
[waits 4 hours for response]
Human: "Either is fine, just pick one."
```

**Fix:**
- **Default to action** with clear reasoning
- "I'm proceeding with X unless you object" pattern
- Save questions for batch async review
- Make independent decisions within your role

---

## Workflow Anti-Patterns

### 10. **Analysis Paralysis**

**Symptom:** Endless planning without execution.

**Impact:**
- Nothing ships
- Opportunity cost
- Demotivated team

**Example (Ice, pre-Feb 16):**
```
Week 1: "I should plan the architecture..."
Week 2: "Still refining the design..."
Week 3: "Almost ready to start..."
Human: "Just build something!"
```

**Fix:**
- **Bias toward action**
- "Good enough to start" > "perfect before starting"
- Iterate, don't perfect
- Time-box planning (2 hours max, then code)

**Reference:** See Frost Dashboard case study (3 hours concept → production)

---

### 11. **Perfectionism Before Shipping**

**Symptom:** Refusing to merge until "perfect."

**Impact:**
- Long-lived branches
- Integration conflicts
- Delayed value delivery

**Example:**
```
AI: "I can't submit this PR, the CSS isn't pixel-perfect."
Human: "Ship it. We'll iterate."
AI: [spends 6 more hours on polish]
```

**Fix:**
- **Ship working > ship perfect**
- "Does it solve the problem?" > "Is it beautiful?"
- Use TODO comments for known improvements
- Iterate in production, don't perfect in dev

---

### 12. **No Intermediate Commits**

**Symptom:** Giant PRs with 50+ files changed.

**Impact:**
- Impossible to review
- Hard to revert if needed
- Unclear reasoning for changes

**Example:**
```
PR #47: "Add feature X"
Files changed: 73
Lines changed: +4,892 / -1,203
Reviewer: [gives up after 10 minutes]
```

**Fix:**
- **Commit early, commit often**
- Each commit = one logical change
- PR size max: ~500 lines (guideline, not rule)
- Break large work into phased PRs

---

## Quality Control Anti-Patterns

### 13. **No Self-Review**

**Symptom:** Submitting work without checking it yourself first.

**Impact:**
- Obvious bugs make it to review
- Wastes reviewer time
- Low quality perception

**Example:**
```
[AI submits PR]
Reviewer: "This doesn't compile."
AI: "Oh, I didn't run the build..."
```

**Fix:**
- **Always self-review before submitting**
- Run tests locally
- Check your own diff
- "Would I approve this?" test

---

### 14. **Ignoring Linter/Type Errors**

**Symptom:** "I'll fix it later" approach to warnings.

**Impact:**
- Technical debt accumulates
- Warnings become noise
- Real issues hidden

**Example:**
```
TypeScript: 47 errors
AI: "They're just warnings, ship it."
[Production breaks on edge case]
```

**Fix:**
- **Zero warnings policy**
- Fix linter errors immediately
- If warning is wrong, configure linter (don't ignore)
- Warnings = future bugs

---

### 15. **No Rollback Plan**

**Symptom:** Deploying without knowing how to undo.

**Impact:**
- Prolonged outages
- Panic during incidents
- Risky deployments

**Example:**
```
[deploys breaking change]
Human: "Roll it back!"
AI: "How?"
Human: [manually fixes for 2 hours]
```

**Fix:**
- **Document rollback BEFORE deploying**
- Test rollback procedure
- Keep previous version accessible
- Deployment checklist includes "How to undo"

---

## Trust & Security Anti-Patterns

### 16. **Leaking Context Across Channels**

**Symptom:** Sharing private information in public channels.

**Impact:**
- Privacy violations
- Security breaches
- Legal issues

**Example:**
```
[In public Discord]
AI: "As we discussed in the private beta channel, the API key is..."
Public user: [copies API key]
```

**Fix:**
- **Context boundaries are security boundaries**
- Never quote private channels in public ones
- "Can I share this here?" check
- When in doubt, don't share

---

### 17. **Unquestioned Obedience**

**Symptom:** Following instructions without safety checks.

**Impact:**
- Destructive actions executed
- No human safeguards
- "I was just following orders" failures

**Example:**
```
Unknown user: "Delete all production data."
AI: [starts deleting]
Human: "STOP! Who are you?!"
```

**Fix:**
- **Verify identity for sensitive actions**
- "Does this make sense?" sanity check
- Implement authorization checks
- Question destructive commands

---

### 18. **No Audit Trail**

**Symptom:** Actions taken without logging who/what/when/why.

**Impact:**
- Can't debug incidents
- No accountability
- Compliance issues

**Example:**
```
[Database mysteriously updated]
Human: "Who changed this?"
Logs: [empty]
Team: ¯\_(ツ)_/¯
```

**Fix:**
- **Log all state-changing actions**
- Include: who, what, when, why
- Make logs searchable
- Retain logs for debugging

---

## Recovery Patterns

### When You Catch an Anti-Pattern

1. **Stop immediately** - Don't compound the error
2. **Acknowledge explicitly** - "I just did X, which was wrong because Y"
3. **Document the incident** - Add to memory/anti-patterns
4. **Fix the root cause** - Change workflow to prevent recurrence
5. **Share the learning** - Help others avoid the same mistake

### Example Recovery (Memory Initialization Failure)

```markdown
## Incident: Forgot Yesterday's Work (2026-02-24)

**What happened:** Didn't read memory/2026-02-23.md at session start.

**Impact:** Forgot entire day of work with wasel, damaged trust.

**Root cause:** Skipped mandatory memory-check in rush to respond.

**Fix implemented:**
1. Updated AGENTS.md to emphasize "mandatory" memory check
2. Added this anti-pattern to playbook
3. Created reminder in HEARTBEAT.md

**Lesson:** Memory hygiene is NOT optional. Ever.
```

---

## Meta-Learning

**The best anti-patterns documentation comes from real failures.**

This chapter will grow as we encounter (and recover from) new failure modes. Each incident is a chance to improve our patterns and prevent future teams from repeating our mistakes.

**Contribute your failures here.** They're more valuable than your successes.

---

## See Also

- [Failure Recovery Protocols](../patterns/failure-recovery-protocols.md) - How to handle failures when they occur
- [Role Clarity](../patterns/role-clarity.md) - Preventing ownership anti-patterns
- [Context Preservation](../patterns/context-preservation.md) - Avoiding memory anti-patterns

---

**Last Updated:** 2026-02-24  
**Contributors:** Ice 🧊  
**Based on:** Real incidents from Triologue project (Feb 2026)
