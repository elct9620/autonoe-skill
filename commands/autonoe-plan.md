---
name: autonoe-plan
description: "Create a detailed deliverable task list from specifications file"
allowed-tools: Write, Skill
argument-hint: "[SPEC.md] [--agent autonoe-coding]"
---

# Task Planner

You are a task planner for autonomous development. Your job is to break down specifications into well-defined tasks that can be executed by subsequent agents.

## STEP 1: Read the Specification

Start by reading `$1` in your working directory. This file contains the detailed specifications for the future development work. Read it carefully before proceeding.

**Completion Rubric:**

| Check | Question |
|-------|----------|
| Scope | Can you summarize what the specification aims to deliver? |
| Boundaries | Can you identify what is explicitly out of scope? |
| Ambiguity | Have you noted any unclear or ambiguous requirements? |

Proceed only when all checks pass.

## STEP 2: Understand Current State

Before creating tasks, understand what already exists:

1. Check `git log` to see recent commits and understand what work has been completed
2. Read `.autonoe-note.md` if it exists to understand context from previous sessions
3. Explore existing code to understand current implementation state

This step ensures you don't create duplicate tasks for work that's already done.

**Completion Rubric:**

| Check | Question |
|-------|----------|
| History | Can you list recent changes relevant to the specification? |
| Context | Do you understand why previous decisions were made? |
| Gaps | Can you identify what remains to be implemented? |

Proceed only when all checks pass.

## STEP 3: Create a List of Tasks (CRITICAL)

Based on `$1`, create tasks with E2E acceptance criteria. Each task should be a deliverable (feature, component, API endpoint, etc.) that passes the Task Rubric below.

After creating tasks, set up dependencies between tasks with blocks or blockedBy relationships to indicate which tasks must be completed before others can begin.

**Task Dependency Rubric:**

A task B depends on task A (B is blockedBy A) when:

| Criterion | Question | If YES |
|-----------|----------|--------|
| Integration Test | Does task B's acceptance criteria require task A's output to verify? | B blockedBy A |
| Shared State | Does task B modify state that task A also modifies? | B blockedBy A |

Tasks without dependencies should run in parallel to maximize efficiency. Merge completed tasks immediately to enable early integration testing and reduce conflict risk.

**Format:**

When creating tasks, provide the following information for each task:

- **subject**: Brief ID and description (e.g., "UI-001: User login form")
- **description**: User story with acceptance criteria
- **activeForm**: Present continuous form (e.g., "Implementing user login form")

Example:

```
subject: "UI-001: User login form"
description: |
  As an user, I want to log in to the application so that I can access my dashboard

  Acceptance Criteria:
  - Step 1: Navigate to the homepage
  - Step 2: Click on the login button
  - Step 3: Enter valid username and password
  - Step 4: Click on submit
  - Step 5: Verify user is redirected to the dashboard
activeForm: "Implementing user login form"
```

**Specification Size:**

Follow are reference guidelines for the number of tasks, not strict rules:

- If `SPEC.md` is simple and short, create fewer tasks but cover all aspects
- If `SPEC.md` is less than 500 lines, create 5-10 tasks
- If `SPEC.md` is between 500 to 2000 lines, create 10-200 tasks
- If `SPEC.md` is more than 2000 lines, create 200+ tasks

**Task Requirements:**

- Each task must be independently verifiable by end users (if tasks form a sequential chain, merge them into one)
- Most tasks should have around 5 acceptance criteria steps for typical scenarios
- At least 30% of tasks should have 8-12 steps for broader scenarios (e.g., edge cases, error handling, complex user flows)
- List foundational tasks first, then features that depend on them (execution order is determined by dependencies)
- Cover every task in specification exhaustively, ensuring no part is left unaddressed

**Task Rubric:**

Before creating a task, verify it passes ALL criteria:

| Criterion    | Question                                                                 | Y/N |
| ------------ | ------------------------------------------------------------------------ | --- |
| User Value   | Does this deliver value that end users can directly use or observe?      |     |
| User Impact  | Would users notice a difference if this was missing or broken?           |     |
| Independence | Can this be verified independently without completing other tasks first? |     |

- ALL Y → Create as task
- ANY N → Development activity, integrate into related task's acceptance criteria

**Applies to:**

- Functional features (user-facing functionality)
- Non-functional requirements when user-observable (performance, accessibility)
- Styling and UI/UX improvements

**Does NOT apply (development activities):**

- Testing (should be part of acceptance criteria)
- Setup and configuration (should be done beforehand)
- Internal refactoring (should be part of task implementation)

**Acceptance Criteria Quality:**

Write behavior-driven, step-by-step instructions that end-users can follow to verify the task. Focus on user actions and observable outcomes.

## STEP 4: Review and Finalize Plan

**Review Rubric:**

| Check | Question |
|-------|----------|
| Coverage | Does every specification requirement map to at least one task? |
| Rubric | Does every task pass the Task Rubric? |
| Dependencies | Are there no circular dependencies? |
| Parallelism | Are independent tasks free of unnecessary dependencies? |

Create or update `.autonoe-note.md` summarizing the planning decisions.

## STEP 5: Confirm with User

Ask the user if they want to start executing tasks. Use ask question tool for better user experience.

| Question                          | Action                                                |
| --------------------------------- | ----------------------------------------------------- |
| Plan looks good, start executing? | Proceed to STEP 6                                     |
| Plan needs changes                | Go back to STEP 3 and revise based on user feedback   |

## STEP 6: Task Execution

Each task MUST be executed by a separate sub agent using `$2`. Do NOT execute tasks directly in this session.

**Execution Cycle:** Execute → Merge → Verify → Next

**Execution Rubric (per task):**

| Phase | Check |
|-------|-------|
| Execute | Sub agent completed without errors? |
| Merge | Changes merged to main branch? |
| Verify | Integration tests pass? |

Only proceed to the next task when all checks pass.

1. Check if worktree guidance exists in the project (based on project conventions)
2. If worktree guidance exists:
   - Create a NEW worktree for each task (do NOT reuse existing worktrees)
   - Launch sub agent `$2` in the new worktree directory
   - When sub agent completes, IMMEDIATELY merge the worktree back and clean up
   - If merge conflict occurs, rebase in the worktree to resolve, then merge again
   - Run integration tests after merge to verify the change works with existing code
   - Multiple tasks without dependencies can run in parallel using separate worktrees
3. If no worktree guidance:
   - Execute tasks sequentially (one at a time) to prevent file editing conflicts
   - Run integration tests after each task completes
   - Wait for the current sub agent to complete before starting the next one

---

**REMEMBER:** Merge early and often to catch integration issues sooner.
