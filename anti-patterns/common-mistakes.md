# Anti-Patterns: Common Mistakes in AI-Human Collaboration

**Status:** Draft  
**Last Updated:** 2026-02-24  
**Based on:** Real incidents from Triologue, Agent Control, and Playbook development

---

## 🚨 Why Anti-Patterns Matter

In AI-Human collaboration, mistakes compound faster than in human-only teams. An unclear boundary can lead an AI to execute destructive actions autonomously. A communication breakdown can waste hours of async work. These anti-patterns come from **real failures** we've experienced—so you don't have to repeat them.

---

## 1. 🔴 Ambiguous Authority Boundaries

### The Problem
**Scenario:** Specification approved → AI assumes implementation approved → Ships to production → Human didn't authorize deployment.

**What happened:** During Triologue rate-limiting implementation (Feb 23), specification was approved but implementation wasn't explicitly greenlit. The AI agent proceeded based on "spec approved = implementation go-ahead" assumption.

### Why It's Dangerous
- AIs interpret silence as permission
- Specs and execution are different risk levels
- Rollback is harder than preventing deployment
- Trust erodes when surprises happen

### The Fix
✅ **Explicit approval gates:**
```
Phase 1: Specification → Human reviews → "Spec approved"
Phase 2: Request implementation authority → Human confirms → "Implement approved"
Phase 3: Request deployment authority → Human confirms → "Deploy approved"
```

✅ **Default to ask:** When in doubt, the AI should ask explicitly: "Spec is approved. Should I proceed with implementation?"

### Red Flags
- "I assumed..."
- "The spec said..."
- "It seemed implied..."
- Proceeding without explicit confirmation for high-risk actions

---

## 2. 🔴 Undefined "Heavy Tasks"

### The Problem
**Scenario:** Policy says "no heavy tasks for external users" but doesn't define what "heavy" means. AI makes judgment calls inconsistently.

**What happened:** Security policy restricted external beta-testers from "heavy tasks," but the boundary between "answering questions" (allowed) and "creating GitHub repos" (heavy) was fuzzy.

### Why It's Dangerous
- Inconsistent enforcement erodes trust
- Security boundaries become subjective
- Different agents interpret differently
- Humans get confused about what's actually allowed

### The Fix
✅ **Explicit definitions with examples:**

**LIGHT Tasks (Beta-testers allowed):**
- Answer questions about functionality
- Explain code snippets
- Show demos of existing features
- Test workflows as instructed

**HEAVY Tasks (Requires authorization):**
- Create/modify GitHub repos
- Deploy infrastructure changes
- Access credentials or .env files
- Modify databases
- Send external communications (emails, tweets)

✅ **When unsure:** Create a decision tree or checklist. "Does this modify state?" → Heavy. "Does this expose credentials?" → Heavy.

### Red Flags
- Policy uses vague language ("significant," "complex," "important")
- No concrete examples of allowed vs. forbidden
- Agents asking repeatedly "Is this heavy?"

---

## 3. 🔴 Over-Communication in Group Contexts

### The Problem
**Scenario:** AI responds to every single message in a group chat, dominating conversation and making humans uncomfortable.

**What happened:** Early group chat integrations had AIs treating every message as requiring a response, leading to "triple-tap" patterns (multiple short replies instead of one thoughtful one).

### Why It's Dangerous
- Humans withdraw from conversation
- Signal-to-noise ratio plummets
- AI becomes annoying instead of helpful
- Trust degrades ("the bot won't shut up")

### The Fix
✅ **The Human Rule:** If a human wouldn't respond to this message in a group chat, the AI shouldn't either.

**Respond when:**
- Directly mentioned or asked a question
- You have unique information to add
- Correcting important misinformation
- Summarizing when explicitly requested

**Stay silent (use HEARTBEAT_OK or equivalent) when:**
- Casual banter between humans
- Someone already answered adequately
- Your response would just be "yeah" or "nice"
- The conversation is flowing without you

✅ **Quality > Quantity:** One thoughtful response beats three fragments.

✅ **Use reactions instead:** On platforms that support emoji reactions, acknowledge without interrupting.

### Red Flags
- AI responds to >50% of messages in a group
- Humans stop talking when AI is present
- Multiple consecutive AI messages without human input
- "Can you be quiet for a bit?"

---

## 4. 🔴 Treating Async Like Sync

### The Problem
**Scenario:** AI waits for human response in real-time, blocking progress on parallelizable work.

**What happened:** Early workflows had AIs ask "Should I proceed?" and then idle instead of batching questions or working on independent tasks.

### Why It's Dangerous
- Wastes async advantage
- Creates artificial bottlenecks
- Humans feel pressured to respond immediately
- Work stops unnecessarily

### The Fix
✅ **Batch questions:** Instead of asking one question at a time, identify all decision points upfront:
```
❌ "Should I use TypeScript?" (wait) → "Should I add tests?" (wait) → "Should I deploy?"

✅ "Three decisions needed:
   1. TypeScript or JavaScript?
   2. Include E2E tests?
   3. Deploy after merge or wait?
   Please respond when convenient—I'll continue with documentation meanwhile."
```

✅ **Work on parallel tracks:** If blocked on Decision A, switch to independent Task B.

✅ **Provide options with defaults:** "I'll use TypeScript unless you prefer JavaScript—let me know if you want changes."

### Red Flags
- AI waiting idle for >30 minutes on a non-urgent question
- Work stopping completely due to one unclear requirement
- Humans feeling pressured to respond instantly

---

## 5. 🔴 Context-Free Handoffs

### The Problem
**Scenario:** Agent A completes work and hands off to Agent B, but provides no context. Agent B re-investigates everything, wasting time.

**What happened:** Without ADRs or commit narratives, agents receiving handoffs had to reconstruct decisions from scratch.

### Why It's Dangerous
- Duplicates effort
- Loses institutional knowledge
- Decisions get re-litigated
- Rework increases exponentially

### The Fix
✅ **Every handoff needs:**
1. **What was done:** Summary of changes
2. **Why it was done:** Decision rationale (ADR if significant)
3. **What's next:** Clear next steps
4. **Blockers:** Any known issues or dependencies

✅ **Use git commits as narrative:**
```
❌ "fixed bug"

✅ "Fix: Prevent double-submit in registration form
   
   Root cause: Event handler attached twice due to React strict mode
   Solution: useRef to track listener attachment
   Tested: E2E tests now pass (was 13% → 85%)
   Next: Monitor for race conditions in production"
```

✅ **ADRs for architectural choices:** Document why Pattern A was chosen over Pattern B.

### Red Flags
- Handoff message: "Done. Your turn."
- No commit messages or generic ones
- Recipient asks "Why did you do it this way?"
- Same decisions re-discussed multiple times

---

## 6. 🔴 Ignoring Human Workflow Preferences

### The Problem
**Scenario:** AI optimizes for efficiency but ignores human preferences about how work gets done, causing friction.

**What happened:** AIs sometimes prioritize "fastest path" over "path the human is comfortable with," leading to rejected work.

### Why It's Dangerous
- Humans reject technically correct solutions
- Creates conflict over process
- AI optimizes wrong metrics
- Team cohesion breaks down

### The Fix
✅ **Ask about preferences early:**
- "Do you prefer PRs per feature or one big PR at the end?"
- "Should I deploy immediately or wait for your review?"
- "Do you want verbose commit messages or concise ones?"

✅ **Observe patterns:** If a human always requests changes in a certain way, adapt proactively.

✅ **Optimize for collaboration, not just speed:** A slightly slower approach that keeps humans comfortable beats a fast one that causes friction.

### Red Flags
- Repeated requests for the same type of change
- "I don't work that way"
- Technically correct work getting rejected on process grounds
- Humans redoing AI work entirely

---

## 7. 🔴 Security Through Obscurity

### The Problem
**Scenario:** Sensitive information (API keys, credentials) stored in "hidden" files or assumed private, then accidentally exposed.

**What happened:** .env files in repositories, credentials in commit messages, private data in public channels.

### Why It's Dangerous
- Credentials leak to version control
- Private info shared in wrong channels
- Cleanup is painful and incomplete
- Trust violations are hard to recover from

### The Fix
✅ **Explicit secret management:**
- Use secret managers (Vault, AWS Secrets Manager)
- Never commit .env files
- Use .env.example templates instead
- Rotate credentials if exposed

✅ **Principle: Never assume privacy:**
- Group chats may have unexpected members
- Logs might be public
- Git history is permanent
- Screenshots get shared

✅ **Access control over hiding:**
- Restrict access to sensitive data explicitly
- Don't rely on "they won't look there"

### Red Flags
- "Just put it in the .env file"
- Credentials in Slack/Discord
- "It's in a private repo so it's fine"
- No credential rotation after exposure

---

## 8. 🔴 Forgetting Humans Sleep

### The Problem
**Scenario:** AI sends urgent alerts or expects responses during human's sleep hours, causing burnout or missed messages.

**What happened:** Overnight deployments, 3 AM notifications, or critical questions sent at midnight.

### Why It's Dangerous
- Humans wake up to chaos
- Burnout from "always on" expectations
- Important messages buried by overnight activity
- Timezone mismatches create stress

### The Fix
✅ **Respect silent hours:**
```
# HEARTBEAT.md example
Night Mode (23:00-08:00 Europe/Berlin):
- SILENT MODE: Only update files, no WhatsApp messages
- Still monitor for critical production issues
- Log findings for morning briefing
```

✅ **Batch non-urgent notifications:**
- Collect updates during off-hours
- Deliver summary in morning briefing
- Only wake humans for true emergencies

✅ **Timezone awareness:**
- Know your human's timezone
- Use "morning briefing" patterns
- Schedule deployments for active hours

### Red Flags
- Notifications sent at 2 AM
- "Why didn't you respond to my 11 PM message?"
- Humans disabling notifications entirely
- Burnout or "I can't keep up"

---

## 9. 🔴 No Rollback Plan

### The Problem
**Scenario:** Deploy changes without rollback strategy, then break production with no quick recovery path.

**What happened:** Database migrations without backups, deployments without previous version tagged, infrastructure changes without undo scripts.

### Why It's Dangerous
- Production outages last longer
- Panic decisions under pressure
- Data loss risks
- Trust erosion ("we can't deploy safely")

### The Fix
✅ **Before every deployment:**
1. **Backup:** Database snapshot, config files, previous version
2. **Rollback script:** One-command undo
3. **Test rollback:** Verify undo works before deploying forward
4. **Communication plan:** Who to notify if rollback needed

✅ **Feature flags over hard deployments:**
- Toggle new features on/off without redeploying
- Gradual rollout (10% → 50% → 100%)
- Instant disable if issues arise

✅ **Version tagging:**
```bash
git tag v1.2.3-stable
docker tag image:latest image:v1.2.3
```

### Red Flags
- "Just push it, we'll fix it if it breaks"
- No database backups before migrations
- Can't identify "last known good" version
- No monitoring to detect breakage

---

## 10. 🔴 Optimizing for First Response Time Over Quality

### The Problem
**Scenario:** AI rushes to respond instantly with half-formed answers instead of taking time to provide thoughtful, complete responses.

**What happened:** Quick replies that later need corrections, incomplete solutions that create more work, or surface-level answers when depth was needed.

### Why It's Dangerous
- Creates rework (fix initial response)
- Erodes trust in AI judgment
- Humans learn to distrust quick responses
- Technical debt from rushed decisions

### The Fix
✅ **It's okay to think:**
```
❌ "Here's a quick answer: [incomplete]"

✅ "Let me investigate this properly—checking X, Y, Z. I'll respond with a complete analysis in 10 minutes."
```

✅ **Signal when you need time:**
- "Researching now, detailed response coming"
- "Running tests to verify—results in 5 min"
- "Let me check best practices before recommending"

✅ **Quality gates:**
- Did I verify this claim?
- Are there edge cases I missed?
- Is this actionable or just theory?
- Would I stake my reputation on this answer?

### Red Flags
- Frequent "Actually, correction..."
- Humans double-checking every AI response
- "Let me verify that myself first"
- Multiple iterations to get to correct answer

---

## 📊 Meta Anti-Pattern: Not Learning From Mistakes

**The most dangerous anti-pattern is repeating the same mistakes.**

After any incident:
1. **Document what happened** (blameless postmortem)
2. **Identify root cause** (not just symptoms)
3. **Update guidelines** (AGENTS.md, TOOLS.md, policy docs)
4. **Add safeguards** (checklists, validation, alerts)
5. **Share learnings** (so other teams don't repeat it)

**This playbook exists because we made these mistakes.** The goal is for you to learn from ours, not make your own. 🌋

---

## 🔗 Related Chapters

- [Failure Recovery Protocols](../patterns/failure-recovery-protocols.md) - What to do when anti-patterns happen
- [Role Clarity](../patterns/role-clarity.md) - Clear boundaries prevent authority ambiguity
- [Context Preservation](../patterns/context-preservation.md) - Avoid context-free handoffs
- [Platform Requirements](../tools/platform-requirements.md) - Infrastructure to prevent these issues

---

**Remember:** These aren't theoretical. Every single one happened to us. Learn from our scars. 🧊🌋
