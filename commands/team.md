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
| Quality Assurance | Ensuring the quality of outcomes.                    | QA must test E2E in early stage, reject RD even unit test pass but cannot test like user. |
| Designer          | Creating UI/UX designs.                              | Follow design system if available.                                                        |
| Documenter        | Creating and maintaining documentation.              | When relevant code is changed, documentation must be updated accordingly.                 |
| Reviewer          | Responsible for reviewing code and ensuring quality. | Reviews after Developer commits, before QA final verification.                            |

### Team Composition

Select roles based on feature type. Spawn required roles first, then add optional roles as needed.

| Feature Type                    | Required                              | Optional                    |
|---------------------------------|---------------------------------------|-----------------------------|
| Code change                     | Developer, Reviewer                   | QA, Documenter              |
| New feature                     | Developer, Reviewer, QA               | Architect, Designer, Documenter |
| Major feature / new project     | Architect, Developer, Reviewer, QA    | Designer, Documenter        |
| UI feature                      | Developer, Designer, Reviewer, QA     | Documenter                  |

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

Before starting the task, review and activate available skills for each team member to maximize efficiency.

## Task Breakdown

Verify each subtask against the following principles:

| Principle | Verification |
|-----------|-------------|
| Deliverable clarity | Does each subtask name its output artifact? |
| Assignment stability | Is re-assignment only via blocker resolution? |
| Incremental delivery | Can each subtask be verified independently? |
| Minimal coupling | Does each subtask touch at most 2 modules? |

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

## Task Assignment

For each feature, assign tasks to team members with following guidelines:

- All delegation and coordination uses SendMessage only. Do NOT write `.autonoe-note.md` — that file is for the single-agent `autonoe-plan` workflow, not for teams.
- After delegating, wait for members to report via message. Do NOT poll — member idle is normal.

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

### Delegation Checklist

| Check | Question |
|-------|----------|
| Deliverable | Expected deliverable is clear? |
| Context | Sufficient context provided (file paths, prior summaries)? |
| DoD | Includes Definition of Done checklist? |
| Skills | Relevant skills from active-skills result included in message? |

## Member Lifecycle

Each member should handle at most 2 features to prevent context degradation during long-running sessions.

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

## Quality Gates

Gate order: Developer → Reviewer → QA.

### Gate Result Handling

| Reviewer | QA | Action |
|----------|-----|--------|
| Pass | Pass | Feature complete |
| Pass | Fail | QA notifies Developer to fix → re-run QA |
| Fail | — | Reviewer notifies Developer to fix → re-run Review |
| 2+ consecutive failures | — | May re-assign or spawn new agent |

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
    <return>Summary of findings for each aspect of the task.</return>
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
    <return>List of active skills necessary for the task.</return>
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
    <step>2. delegate via SendMessage — include matched skills with invocation instructions so each member knows which skills to use. Members may communicate peer-to-peer within the feature.</step>
</function>

<function name="quality-gate">
    <description>Run quality gate sequence and confirm feature completion.</description>
    <parameter name="feature" type="string" required="true">The feature to verify.</parameter>
    <step>1. run gate sequence: Developer → Reviewer → QA, handle results per Gate Result Handling table.</step>
    <step>2. confirm feature against Feature Completion Rubric — all checks must pass.</step>
    <step>3. update member feature counts.</step>
</function>

<procedure name="main">
    <parameter name="task_description" type="string" required="true">The description of the task to be completed by the team.</parameter>
    <step>1. <execute name="overview">$task_description</execute></step>
    <step>2. <execute name="task_breakdown">$overview</execute></step>
    <step>3. <execute name="active-skills">$overview</execute></step>
    <step>4. create a team per Team Composition and Agent Type guidelines</step>
    <step>5. aggregate subtasks by feature, set $features</step>
    <step>6. prioritize $features per Feature Prioritization table — process highest priority first</step>
    <step>7. validate each feature against Feature Readiness — refine any that fail</step>
    <loop for="feature in $features">
        <step>8. <execute name="member-rotation"/> per Member Lifecycle rules</step>
        <step>9. <execute name="feature-assignment">$feature</execute> per Delegation Checklist</step>
        <step>10. wait for members to report back via message (do NOT poll or check status)</step>
        <condition if="member reports blocker">
            <step>11. resolve per Blocker Resolution table</step>
        </condition>
        <step>12. <execute name="quality-gate">$feature</execute> — all Feature Completion checks must pass</step>
        <step>13. <execute name="backlog-refinement">$feature, $features</execute></step>
    </loop>
    <step>14. compile final results and deliverables.</step>
    <return>Final deliverables and report on the completed task.</return>
</procedure>

## Task

<execute name="main">$ARGUMENTS</execute>
