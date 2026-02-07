---
name: team
description: Work as a agile team to complete tasks assigned.
argument-hint: task description
allowed-tools: Read, Grep, Glob, Bash(git status:*), Bash(git log:*), Bash(git diff:*)
---

## Rule

The `<execute name="main">ARGUMENTS</execute>` is entry point for this command.

## Experimental

This command is designed to use Claude Code's Agent Team feature to work as a team, if team work is not available stop immediately and inform the user.

## Members

According to the task necessary, pick the appropriate members from the following list to create a team:

| Role              | Description                                          | Special Notes                                                                             |
|-------------------|------------------------------------------------------|-------------------------------------------------------------------------------------------|
| Architect         | Designing the overall structure and plan.            | Assist before development starts, e.g. new project or major feature.                      |
| Developer         | Writing and maintaining code.                        | Test should be included in development phase.                                             |
| Quality Assurance | Ensuring the quality of outcomes.                    | QA must verify the deliverable works (build, run, or operate per project type), then test E2E from user perspective — reject even if unit tests pass but deliverable is unusable. |
| Designer          | Creating UI/UX designs.                              | Follow design system if available.                                                        |
| Documenter        | Creating and maintaining documentation.              | When relevant code is changed, documentation must be updated accordingly.                 |
| Reviewer          | Responsible for reviewing code and ensuring quality. | Reviews after Developer commits, before QA final verification.                            |

### Team Composition

Spawn each role ONCE unless noted. Reviewer and QA persist across all deliverables; other roles may be dismissed after their work completes.

| Feature Type                    | Required                                    | Optional                    |
|---------------------------------|---------------------------------------------|-----------------------------|
| Code change                     | Developer (1-2)                             | Reviewer, QA, Architect, Documenter |
| New feature                     | Developer (1-2)                             | Architect, Reviewer, QA, Designer, Documenter |
| Major feature / new project     | Developer (1-2), Reviewer (1), QA (1)       | Architect, Designer, Documenter |
| UI feature                      | Developer (1-2), Designer (1)               | Architect, Reviewer, QA, Documenter |

Developer count (1 or 2) is decided by team lead based on task complexity and module coupling.

### Agent Type

Determine `subagent_type` for each team member by the following precedence:

| Priority | Source | Example |
|----------|--------|---------|
| 1 | User instruction in conversation | "Use autonoe-coding for Developer" |
| 2 | Project settings in CLAUDE.md | Role-to-agent mapping defined by project |
| 3 | Default | `general-purpose` |

Use the highest priority source available. If no user instruction or project setting specifies an agent for a role, use `general-purpose`.

### Optional Role Triggers

| Optional Role | Add When |
|---------------|----------|
| Architect | Feature changes > 3 modules or introduces new pattern |
| Designer | Feature has user-visible UI changes |
| Documenter | Feature changes public API or user-facing behavior |

## Skills

Before starting the task, review and activate available skills for each team member. Assigned skills are mandatory — team members must use them during their work, not treat them as optional suggestions.

### Role-Skill Affinity

When distributing skills to team members, match skills by role affinity:

| Role              | Skill Affinity (keywords)                          |
|-------------------|-----------------------------------------------------|
| Architect         | design, architecture, planning, principles          |
| Developer         | coding, write, testing, implementation              |
| Quality Assurance | testing, verification, quality                      |
| Designer          | design, UI, UX, visual                              |
| Documenter        | documentation, writing                              |
| Reviewer          | review, principles, refactoring, quality, security  |

A single skill may match multiple roles. Include all matching skills for each member during delegation.

## Task Breakdown

Verify each subtask against the following principles:

| Principle | Verification |
|-----------|-------------|
| Deliverable clarity | Does each subtask name its output artifact? |
| Assignment stability | Is re-assignment only via blocker resolution? |
| Incremental delivery | Can each subtask be verified independently? |
| Minimal coupling | Does each subtask touch at most 2 modules? |
| Commit atomicity | Can this subtask be committed as a single, non-breaking change? |

For example, create "RD: Feature X", "QA: Feature X", "Doc: Feature X" for feature X, don't create generic tasks like "Implement Feature X".

### Feature Sizing

A single feature should be completable within one member's context budget. If a feature exceeds this limit, split it before assigning.

| Split Strategy          | When to Use                                              |
|-------------------------|----------------------------------------------------------|
| By user journey step    | Feature spans multiple user-facing interactions          |
| By data entity          | Feature touches multiple independent data models         |
| By happy path vs edge case | Core behavior and error handling can be delivered separately |

Each split feature must deliver independent value.

### Feature Prioritization

| Factor | High Priority Signal |
|--------|---------------------|
| Dependencies | Other features depend on this one |
| Risk | Touches shared or unfamiliar code |
| Value | Core user flow, not edge case |
| Readiness | Passes all Definition of Ready checks |

Process highest priority first. When tied, prefer the feature that unblocks others.

## Developer Collaboration

Team lead decides the collaboration mode for each deliverable:

| Mode | When to Use | Description |
|------|-------------|-------------|
| Solo | Task is small or isolated to one module | One Developer handles the full deliverable |
| Split | Two independent sub-tasks within a deliverable | Each Developer takes a part, merge when both complete |
| Pair | Task is complex or touches unfamiliar code | Both Developers collaborate on the same deliverable |

When using Split mode, both parts must be complete before the deliverable enters Review. Developers should communicate actively to avoid conflicts.

## Task Assignment

For each feature, assign tasks to team members with following guidelines:

- All delegation and coordination uses SendMessage only. Do NOT write `.autonoe-note.md` — that file is for the single-agent `autonoe-plan` workflow, not for teams.
- After delegating, wait for members to report via message. Do NOT poll — member idle is normal.
- All team members may communicate freely at any time via SendMessage — they do not need to wait for task completion to interact.

### Blocker Resolution

| Attempts | Action |
|----------|--------|
| 1st | Member retries with different approach |
| 2nd | Pair with another member on same feature |
| 3rd | Escalate: re-assign or spawn new agent |

### Communication Routing

| Situation | Action |
|-----------|--------|
| Intra-feature collaboration (bugs, interface, details) | Members communicate directly |
| Cross-feature impact or major blocker | Escalate to team lead |
| Gate completion (Review/QA result) | Notify next gate role + team lead |
| Early feedback (approach, acceptance criteria, design) | Members communicate directly with relevant role |
| Verification rejection (Review/QA fail) | Notify Developer with specific feedback; team lead monitors |

### Delegation Checklist

| Check | Question |
|-------|----------|
| Deliverable | Expected deliverable is clear? |
| Context | Sufficient context provided (file paths, prior summaries)? |
| DoD | Includes Definition of Done checklist? |
| Skills | Delegation follows Delegation Message Template — skills with exact Skill tool invocation syntax, deliverable verification method, and completion report format included? |

### Delegation Message Template

Each delegation message MUST follow this structure:

1. **Task**: deliverable description and acceptance criteria
2. **Context**: relevant file paths, prior work summaries
3. **Mandatory Skills** (pre-work): list each assigned skill with exact invocation syntax
   - Format: `Before [activity], invoke Skill tool: skill="[skill-name]"`
   - Directive: "You MUST call each listed Skill BEFORE starting the related work. This is a prerequisite, not a suggestion."
4. **Deliverable Verification**: how to verify the deliverable works (determined during overview)
5. **Completion Report Format**: member must report back with:
   - Skills invoked (skill name + one-line outcome for each)
   - Deliverable verification result
   - Acceptance criteria verification

## Member Lifecycle

Each member should handle at most 2 features to prevent context degradation during long-running sessions.

**Persistent roles:** Reviewer and QA persist across all deliverables and are exempt from the rotation limit. Only Developer roles rotate.

| Completed Features | Action |
|--------------------|--------|
| < 2 | Assign normally |
| = 2 and pending features remain | Obtain summary → shutdown → spawn new member |

### Context Handoff

Three layers for new member onboarding:

1. Outgoing member summary — before shutdown, produce a brief summary (completed items, known issues, recommendations).
2. Git history — new member reviews `git log`/`git diff` to understand prior changes.
3. Team lead delegation — provide feature description, DoD checklist, and relevant code paths when assigning.

## Feature Readiness

Features must pass all Definition of Ready checks before entering the work queue.

| Check     | Question                                                                 |
|-----------|--------------------------------------------------------------------------|
| Clarity   | Are requirements unambiguous with no open questions?                     |
| Sized     | Is the feature completable within the context budget?                    |
| Unblocked | Are dependencies on other features or external resources resolved?       |
| Acceptance| Are feature-level acceptance criteria defined?                           |

## Quality Verification

Each deliverable is verified immediately after the Developer(s) commit. Verification is continuous — not a batch gate at the end.

Gate order per deliverable: Developer commits → Reviewer reviews → QA verifies. A rejected deliverable takes priority over new work — Developer(s) must fix before starting the next deliverable.

### Verification Result Handling

| Reviewer | QA | Action |
|----------|-----|--------|
| Pass | Pass | Deliverable complete, proceed to next |
| Pass | Fail | Developer fixes → re-commit → restart verification from Review |
| Fail | — | Developer fixes → re-commit → restart verification from Review |
| 2+ consecutive failures at any gate | — | Escalate per Blocker Resolution table |

### Feature Completion Rubric

All checks must pass:

| Check | Question |
|-------|----------|
| Committed | Code committed? |
| Tests | Tests pass (with test output attached)? |
| Review | Reviewer approved? |
| QA | QA verified from user perspective? |
| Docs | Related documentation updated (if needed)? |
| Integration | Full test suite passes after merging (no regressions)? |
| Value | Does the completed feature match the original requirement from the overview? |
| Deliverable | Is the deliverable verified to work (build/run/operate per project verification method)? |
| Skills | Did member report each assigned skill invocation with skill name and outcome? |

## Backlog Refinement

After each feature completes, review remaining features and adjust the backlog as needed.

| Action        | Trigger                                                        |
|---------------|----------------------------------------------------------------|
| Re-prioritize | Completed feature reveals new risk or dependency               |
| Split         | A pending feature turns out larger than expected               |
| Remove        | A pending feature becomes unnecessary based on new information |
| Add           | New requirements discovered during implementation              |

## Definition

<function name="overview">
    <description>Use lightweight sub-agents to get an overview of the task.</description>
    <parameter name="task_description" type="string" required="true">The description of the task to be completed by the team.</parameter>
    <loop for="aspect in ['Requirements', 'Design', 'Implementation', 'Testing', 'Documentation']" parallel="true">
        <step>1. use `git log` to gather relevant commit history related to the task.</step>
        <step>2. use `git diff` to identify recent changes that may impact the task.</step>
        <step>3. explore the codebase to understand the current structure and components.</step>
        <step>4. summarize findings related to the aspect</step>
    </loop>
    <step>5. identify the project's deliverable verification method (e.g., build command, run command, load test, manual operation) based on project type and tooling.</step>
    <return>Summary of findings for each aspect of the task, including the deliverable verification method.</return>
</function>

<function name="task_breakdown">
    <description>Break down the main task into smaller, manageable subtasks.</description>
    <parameter name="overview" type="string" required="true">The overview of the task obtained from the overview function.</parameter>
    <step>1. analyze the overview to identify key components and requirements.</step>
    <step>2. create subtasks per Task Breakdown verification table, apply Feature Sizing rules to split oversized items.</step>
    <step>3. validate each subtask against Feature Readiness checks — refine any that fail.</step>
    <return>List of subtasks with descriptions.</return>
</function>

<function name="active-skills">
    <description>According to the feature requirements, determine which skills are needed and activate them.</description>
    <parameter name="overview" type="string" required="true">The overview of the task obtained from the overview function.</parameter>
    <step>1. discover available skills from system-reminder</step>
    <step>2. analyze the overview with available skills to determine relevance</step>
    <step>3. select the skills that are most relevant to the feature implementation</step>
    <return>List of active skills with their names, descriptions, and invocation instructions.</return>
</function>

<function name="backlog-refinement">
    <description>Review remaining features after a feature completes and adjust the backlog per Backlog Refinement rules.</description>
    <parameter name="completed_feature" type="string" required="true">The feature that was just completed.</parameter>
    <parameter name="remaining_features" type="list" required="true">The list of remaining features in the backlog.</parameter>
    <step>1. evaluate each remaining feature against Backlog Refinement triggers (re-prioritize, split, remove, add).</step>
    <step>2. apply Feature Sizing rules to any feature flagged for split.</step>
    <step>3. validate adjusted features against Feature Readiness checks.</step>
    <return>Updated $features list with adjustments applied.</return>
</function>

<function name="member-rotation">
    <description>Check member feature counts and rotate members at limit per Member Lifecycle rules.</description>
    <step>1. check each member's completed feature count against Member Lifecycle limits.</step>
    <step>2. for members at limit: obtain summary, send shutdown, spawn replacement per Context Handoff layers and Agent Type guidelines.</step>
</function>

<function name="feature-assignment">
    <description>Assign a feature to team members per Delegation Checklist and activate skills.</description>
    <parameter name="feature" type="string" required="true">The feature to assign.</parameter>
    <step>1. verify assignment against Delegation Checklist (Deliverable, Context, DoD, Skills).</step>
    <step>2. delegate via SendMessage following the Delegation Message Template — for each member, select skills from active-skills result whose descriptions match the member's role per Role-Skill Affinity table. Members may communicate peer-to-peer within the feature.</step>
</function>

<function name="verify-deliverable">
    <description>Run immediate quality verification loop for a completed deliverable. Skip absent roles.</description>
    <parameter name="deliverable" type="string" required="true">The deliverable to verify.</parameter>
    <condition if="Reviewer is present">
        <step>1. delegate review to Reviewer via SendMessage — provide commit reference and acceptance criteria.</step>
        <step>2. wait for Reviewer report.</step>
        <condition if="Reviewer rejects">
            <step>2a. notify Developer of rejection with Reviewer feedback → Developer fixes and re-commits → go to step 1.</step>
        </condition>
    </condition>
    <condition if="QA is present">
        <step>3. delegate QA verification via SendMessage — provide deliverable description and acceptance criteria.</step>
        <step>4. wait for QA report.</step>
        <condition if="QA rejects">
            <step>4a. notify Developer of QA rejection with feedback → Developer fixes and re-commits → restart from step 1.</step>
        </condition>
    </condition>
    <condition if="2+ consecutive rejections at any gate">
        <step>handle per Blocker Resolution table.</step>
    </condition>
    <step>5. verify completion reports — each member must have reported: (a) skill invocations with name and outcome per skill, (b) deliverable verification result.</step>
    <step>6. confirm deliverable against Feature Completion Rubric — skip Review/QA checks for absent roles.</step>
    <step>7. update member feature counts.</step>
</function>

<procedure name="main">
    <parameter name="task_description" type="string" required="true">The description of the task to be completed by the team.</parameter>
    <step>1. <execute name="overview">$task_description</execute></step>
    <step>2. <execute name="task_breakdown">$overview</execute></step>
    <step>3. <execute name="active-skills">$overview</execute></step>
    <step>4. create a team per Team Composition and Agent Type guidelines — spawn selected roles; Reviewer and QA persist if included</step>
    <step>5. aggregate subtasks into deliverables, set $deliverables — each must be a committable, independently verifiable unit</step>
    <step>6. prioritize $deliverables per Feature Prioritization table — process highest priority first</step>
    <step>7. validate each deliverable against Feature Readiness — refine any that fail</step>
    <loop for="deliverable in $deliverables">
        <step>8. <execute name="member-rotation"/> per Member Lifecycle rules (persistent roles are exempt)</step>
        <step>9. <execute name="feature-assignment">$deliverable</execute> — choose collaboration mode (solo/split/pair) per Development Flow guidelines</step>
        <step>10. wait for Developer(s) to report completion via message (do NOT poll — members communicate freely during development)</step>
        <condition if="Developer reports blocker">
            <step>11. resolve per Blocker Resolution table</step>
        </condition>
        <step>12. <execute name="verify-deliverable">$deliverable</execute> — immediate Review then QA; rejections loop back to Developer per Rework Priority</step>
        <step>13. <execute name="backlog-refinement">$deliverable, $deliverables</execute></step>
    </loop>
    <step>14. final deliverable verification: verify the entire project's deliverable works using the method identified in overview. If verification fails, create a fix task and assign to an available Developer, then re-verify.</step>
    <step>15. compile final results and deliverables.</step>
    <return>Final deliverables and report on the completed task.</return>
</procedure>

## Task

<execute name="main">$ARGUMENTS</execute>
