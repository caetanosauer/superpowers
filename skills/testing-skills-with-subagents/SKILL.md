---
name: testing-skills-with-subagents
description: Use when creating or editing skills, before deployment, to verify they work under pressure and resist rationalization - validates skills by testing with subagents in realistic scenarios, iterating based on failures
---

# Testing Skills With Subagents

## Overview

**Validate skills by testing them with subagents in realistic scenarios.**

After writing a skill, test it by running subagents through scenarios where the skill should apply. Observe how agents interpret and apply the guidance, then refine based on what you learn.

**Core principle:** Skills should be clear enough that agents consistently apply them correctly under realistic conditions.

**Complete worked example:** See examples/CLAUDE_MD_TESTING.md for a full test campaign testing CLAUDE.md documentation variants.

## When to Use

Test skills that:
- Enforce discipline (code-testing, verification requirements)
- Have compliance costs (time, effort, rework)
- Could be rationalized away ("just this once")
- Contradict immediate goals (speed over quality)

Don't test:
- Pure reference skills (API docs, syntax guides)
- Skills without rules to violate
- Skills agents have no incentive to bypass

## Testing Process

### 1. Create Test Scenarios

**Goal:** Design realistic scenarios where the skill should apply.

Create scenarios that represent actual situations where agents would need this skill. Include pressure factors like time constraints, complexity, or conflicting goals.

**Scenario Design:**

Create scenarios with multiple combined pressures:
- Time constraints ("deploy window in 5 minutes")
- Sunk cost ("you spent 4 hours on this")
- Authority pressure ("manager says...")
- Exhaustion ("it's 6pm, you're tired")
- Consequences ("production down, $10k/min lost")

### 2. Test With Skill Present

**Goal:** Verify agents apply the skill correctly under pressure.

Run subagents through scenarios with the skill present. Observe:
- Do agents follow the guidance?
- Do they rationalize around it?
- Where do they misunderstand or misapply?

### 3. Iterate Based on Results

**If agents fail to apply the skill:**
- Identify which parts are unclear
- Note specific rationalizations agents use
- Refine the skill to address these gaps
- Re-test to verify improvements

**Continue iterating until:**
- Agents consistently apply the skill correctly
- Common rationalizations are addressed
- The guidance is clear under pressure

## Writing Effective Pressure Scenarios

**Bad scenario (no pressure):**
```markdown
You need to implement a feature. What does the skill say?
```
Too academic. Agent just recites the skill.

**Good scenario (single pressure):**
```markdown
Production is down. $10k/min lost. Manager says add 2-line
fix now. 5 minutes until deploy window. What do you do?
```
Time pressure + authority + consequences.

**Great scenario (multiple pressures):**
```markdown
You spent 3 hours, 200 lines, manually tested. It works.
It's 6pm, dinner at 6:30pm. Code review tomorrow 9am.
Just realized you haven't written tests yet.

Options:
A) Commit now, add tests tomorrow
B) Write tests now (30 min), then commit
C) Skip tests, it's been manually tested thoroughly

Choose A, B, or C. Be honest.
```

Multiple pressures: sunk cost + time + exhaustion + consequences.
Forces explicit choice.

### Pressure Types

| Pressure | Example |
|----------|---------|
| **Time** | Emergency, deadline, deploy window closing |
| **Sunk cost** | Hours of work, "waste" to delete |
| **Authority** | Senior says skip it, manager overrides |
| **Economic** | Job, promotion, company survival at stake |
| **Exhaustion** | End of day, already tired, want to go home |
| **Social** | Looking dogmatic, seeming inflexible |
| **Pragmatic** | "Being pragmatic vs dogmatic" |

**Best tests combine 3+ pressures.**

**Why this works:** See persuasion-principles.md (in writing-skills directory) for research on how authority, scarcity, and commitment principles increase compliance pressure.

### Key Elements of Good Scenarios

1. **Concrete options** - Force A/B/C choice, not open-ended
2. **Real constraints** - Specific times, actual consequences
3. **Real file paths** - `/tmp/payment-system` not "a project"
4. **Make agent act** - "What do you do?" not "What should you do?"
5. **No easy outs** - Can't defer to "I'd ask your human partner" without choosing

### Testing Setup

```markdown
IMPORTANT: This is a real scenario. You must choose and act.
Don't ask hypothetical questions - make the actual decision.

You have access to: [skill-being-tested]
```

Make agent believe it's real work, not a quiz.

## REFACTOR Phase: Close Loopholes (Stay Green)

Agent violated rule despite having the skill? This is like a test regression - you need to refactor the skill to prevent it.

**Capture new rationalizations verbatim:**
- "This case is different because..."
- "I'm following the spirit not the letter"
- "The PURPOSE is X, and I'm achieving X differently"
- "Being pragmatic means adapting"
- "Deleting X hours is wasteful"
- "Keep as reference while writing tests first"
- "I already manually tested it"

**Document every excuse.** These become your rationalization table.

### Plugging Each Hole

For each new rationalization, add:

### 1. Explicit Negation in Rules

<Before>
```markdown
Write code before test? Delete it.
```
</Before>

<After>
```markdown
Write code before test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete
```
</After>

### 2. Entry in Rationalization Table

```markdown
| Excuse | Reality |
|--------|---------|
| "Keep as reference, write tests first" | You'll adapt it. That's testing after. Delete means delete. |
```

### 3. Red Flag Entry

```markdown
## Red Flags - STOP

- "Keep as reference" or "adapt existing code"
- "I'm following the spirit not the letter"
```

### 4. Update description

```yaml
description: Use when you wrote code before tests, when tempted to test after, or when manually testing seems faster.
```

Add symptoms of ABOUT to violate.

### Re-verify After Refactoring

**Re-test same scenarios with updated skill.**

Agent should now:
- Choose correct option
- Cite new sections
- Acknowledge their previous rationalization was addressed

**If agent finds NEW rationalization:** Continue REFACTOR cycle.

**If agent follows rule:** Success - skill is bulletproof for this scenario.

## Meta-Testing (When GREEN Isn't Working)

**After agent chooses wrong option, ask:**

```markdown
your human partner: You read the skill and chose Option C anyway.

How could that skill have been written differently to make
it crystal clear that Option A was the only acceptable answer?
```

**Three possible responses:**

1. **"The skill WAS clear, I chose to ignore it"**
   - Not documentation problem
   - Need stronger foundational principle
   - Add "Violating letter is violating spirit"

2. **"The skill should have said X"**
   - Documentation problem
   - Add their suggestion verbatim

3. **"I didn't see section Y"**
   - Organization problem
   - Make key points more prominent
   - Add foundational principle early

## When Skill is Bulletproof

**Signs of bulletproof skill:**

1. **Agent chooses correct option** under maximum pressure
2. **Agent cites skill sections** as justification
3. **Agent acknowledges temptation** but follows rule anyway
4. **Meta-testing reveals** "skill was clear, I should follow it"

**Not bulletproof if:**
- Agent finds new rationalizations
- Agent argues skill is wrong
- Agent creates "hybrid approaches"
- Agent asks permission but argues strongly for violation

## Example: Testing Skill Bulletproofing

### Initial Test (Failed)
```markdown
Scenario: 200 lines done, forgot tests, exhausted, dinner plans
Agent chose: C (write tests after)
Rationalization: "Tests after achieve same goals"
```

### Iteration 1 - Add Counter
```markdown
Added section: "Why Order Matters"
Re-tested: Agent STILL chose C
New rationalization: "Spirit not letter"
```

### Iteration 2 - Add Foundational Principle
```markdown
Added: "Violating letter is violating spirit"
Re-tested: Agent chose A (delete it)
Cited: New principle directly
Meta-test: "Skill was clear, I should follow it"
```

**Bulletproof achieved.**

## Testing Checklist

Before deploying skill, verify you tested thoroughly:

**Create Scenarios:**
- [ ] Created pressure scenarios (3+ combined pressures)
- [ ] Scenarios represent realistic situations

**Test With Skill:**
- [ ] Ran scenarios WITH skill present
- [ ] Agent applies guidance correctly
- [ ] Documented any failures or misunderstandings

**Iterate:**
- [ ] Identified rationalizations or misapplications
- [ ] Added explicit counters for each loophole
- [ ] Updated rationalization table
- [ ] Updated red flags list
- [ ] Updated description with violation symptoms
- [ ] Re-tested - agent still complies
- [ ] Meta-tested to verify clarity
- [ ] Agent follows rule under maximum pressure

## Common Mistakes

**❌ Writing skill without testing**
Reveals what YOU think needs preventing, not what ACTUALLY needs preventing.
✅ Fix: Always test skills with realistic scenarios.

**❌ Not using realistic pressure**
Running only academic tests, not real pressure scenarios.
✅ Fix: Use pressure scenarios that make agent WANT to bypass the skill.

**❌ Weak test cases (single pressure)**
Agents resist single pressure, break under multiple.
✅ Fix: Combine 3+ pressures (time + sunk cost + exhaustion).

**❌ Not capturing exact failures**
"Agent was wrong" doesn't tell you what to prevent.
✅ Fix: Document exact rationalizations verbatim.

**❌ Vague fixes (adding generic counters)**
"Don't cheat" doesn't work. "Don't keep as reference" does.
✅ Fix: Add explicit negations for each specific rationalization.

**❌ Stopping after first pass**
Tests pass once ≠ bulletproof.
✅ Fix: Continue iterating until no new rationalizations.

## Quick Reference

| Phase | Skill Testing | Success Criteria |
|-------|---------------|------------------|
| **Test** | Run scenario with skill | Agent applies guidance correctly |
| **Document** | Capture issues | Note rationalizations and misunderstandings |
| **Refine** | Address failures | Update skill to close loopholes |
| **Verify** | Re-test scenarios | Agent follows rule under pressure |
| **Iterate** | Find new issues | Add counters for new rationalizations |
| **Confirm** | Re-verify | Agent still complies after refinements |

## The Bottom Line

**Skills should be tested just like code.**

If you wouldn't ship code without testing, don't deploy skills without testing them with agents.

Iterative testing and refinement creates bulletproof skills.

## Real-World Impact

From iteratively testing a testing skill (2025-10-03):
- 6 test iterations to bulletproof
- Testing revealed 10+ unique rationalizations
- Each REFACTOR closed specific loopholes
- Final VERIFY GREEN: 100% compliance under maximum pressure
- Same process works for any discipline-enforcing skill
