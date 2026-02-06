---
name: team
description: Work as a agile team to complete tasks assigned.
argument-hint: task description
allowed-tools: Read, Grep, Glob, Bash(git status:*), Bash(git log:*), Bash(git diff:*)
---

## Rule

The `<execute name="main">ARGUMENTS</execute>` is entry point for this command.

## Expierimental

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

## Skills

Before starting the task, review available skills for current environment and assign correct skills to each team member to maximize efficiency.

**IMPORTANT:** Always use skills before starting the task to ensure the best possible outcome if the skill is available.

## Task Breakdown

Each team member should focus on their own area of expertise, but collaboration and communication are key to ensure the task is completed successfully.

When breaking down the task, consider the following aspects:

- Each subtask should be clearly defined with deliverable outcomes.
- Prefer keeping original assignments, but when a member is blocked and reasonable attempts have failed, re-assign or spawn a new agent.
- Delivery early and often, with regular check-ins to ensure progress is being made.
- Minimize dependencies between subtasks to avoid bottlenecks, work as Kanban style.

For example, create "RD: Feature X", "QA: Feature X", "Doc: Feature X" for feature X, don't create generic tasks like "Implement Feature X".

## Task Assignment

For each feature, assign tasks to team members with following guidelines:

- Co-work with other members, including pair programming, joint reviews, and collaborative testing.
- Members within the same feature may communicate directly (QA↔Developer, Designer↔Developer, Reviewer↔Developer). Team lead maintains visibility through idle notification peer DM summaries.
- Escalate to team lead only for cross-feature impacts or major blockers.
- All delegation and coordination uses SendMessage only. Do NOT write `.autonoe-note.md` — that file is for the single-agent `autonoe-plan` workflow, not for teams.
- After delegating, wait for members to report via message. Do NOT poll — member idle is normal.

### Communication Routing

| Situation | Action |
|-----------|--------|
| Intra-feature collaboration (bugs, interface, details) | Members communicate directly |
| Cross-feature impact or major blocker | Escalate to team lead |
| Gate completion (Review/QA result) | Notify next gate role + team lead |

### Delegation Checklist

Verify before each assignment:

| Check | Question |
|-------|----------|
| Deliverable | Expected deliverable is clear? |
| Context | Sufficient context provided (file paths, prior summaries)? |
| DoD | Includes Definition of Done checklist? |

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

## Quality Gates

Each feature must pass through Reviewer and QA gates before completion. The gate order is: Developer completes → Reviewer reviews → QA verifies.

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

## Definition

<function name="overview">
    <description>Use lightweight sub-agents to get an overview of the task.</description>
    <parameter name="task_description" type="string" required="true">The description of the task to be completed by the team.</parameter>
    <loop for="aspect in ['Requirements', 'Design', 'Implementation', 'Testing', 'Documentation']" parallel="true">
        <step>1. use `git log` to gather relevant commit history related to the task.</step>
        <step>2. use `git diff` to identify recent changes that may impact the task.</step>
        <step>3. explode the codebase to understand the current structure and components.</step>
        <step>4. summarize findings related to the aspect</step>
    </loop>
    <return>Summary of findings for each aspect of the task.</return>
</function>

<function name="task_breakdown">
    <description>Break down the main task into smaller, manageable subtasks.</description>
    <parameter name="overview" type="string" required="true">The overview of the task obtained from the overview function.</parameter>
    <step>1. analyze the the overview to identify key components and requirements.</step>
    <step>2. create a list of tasks needed to complete the overall task.</step>
    <step>3. review with breakdown rules to ensure all aspects are covered.</step>
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

<procedure name="main">
    <parameter name="task_description" type="string" required="true">The description of the task to be completed by the team.</parameter>
    <step>1. <execute name="overview">$task_description</execute> to gather initial insights about the task.</step>
    <step>2. <execute name="task_breakdown">$overview</execute> to create a detailed plan of action.</step>
    <step>3. <execute name="active-skills">$overview</execute> to identify and activate necessary skills for the task.</step>
    <step>4. create a team to working on the subtasks</step>
    <step>5. aggregate subtasks by feature, set $features</step>
    <step>6. prioritize $features by value, risk, and dependencies — process highest priority first</step>
    <loop for="feature in $features">
        <step>7. check member feature counts, rotate members at limit per Member Lifecycle rules</step>
        <step>8. assign $feature to team members per Delegation Checklist</step>
        <condition if="skill available for assigned member">
            <step>9. use Skill($skill) before starting the task</step>
        </condition>
        <step>10. delegate to members via SendMessage — members may communicate peer-to-peer within the feature</step>
        <step>11. wait for members to report back via message (do NOT poll or check status)</step>
        <condition if="member reports blocker">
            <step>12. coordinate resolution — if member remains blocked after reasonable attempts, may re-assign or spawn new agent</step>
        </condition>
        <step>13. run Quality Gates: Developer → Reviewer → QA, handle results per Gate Result Handling table</step>
        <step>14. confirm $feature against Feature Completion Rubric, all checks must pass</step>
        <step>15. update member feature counts</step>
    </loop>
    <step>16. compile final results and deliverables.</step>
    <return>Final deliverables and report on the completed task.</return>
</procedure>

## Task

<execute name="main">$ARGUMENTS</execute>
