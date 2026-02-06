# Autonoe Skill

Autonomous agent orchestrator skill mainly designed for Claude Code.

> [!NOTE]
> This is alternative for [Autonoe](https://github.com/elct9620/autonoe) project because Anthropic's policy does not allow Claude Agent SDK to use Claude Subscription from 3rd party applications.

## Installation

To use this plugin with Claude Code:

```bash
claude --plugin-dir /path/to/Autonoe-Skill
```

Or install it to your global plugins directory and it will be loaded automatically.

## Usage

### Spec-driven Planning

Use specification files to drive task planning and execution.

#### 1. Create Specification

Create a `SPEC.md` file in your project directory containing the detailed specifications for what you want to build.

#### 2. Generate Task Plan

Run the planning command:

```bash
/autonoe-plan [spec-file] [--agent agent-name]
```

**Parameters** (all optional):
- `spec-file`: Path to your specification file (default: `SPEC.md`)
- `--agent`: Name of the coding agent to use (default: `autonoe-coding`)

**Example:**
```bash
/autonoe-plan SPEC.md --agent autonoe-coding
```

The planner will:
- Read your specification
- Analyze the current codebase state
- Break down the spec into fine-grained, testable tasks
- Set up task dependencies

#### 3. Review and Execute

After planning:
1. **Review** the generated task list
2. **Confirm** to start implementation
3. The `autonoe-coding` agent will:
   - Execute tasks one at a time
   - Verify each task thoroughly (E2E testing, manual verification)
   - Commit after completing each task
   - Leave handoff notes for the next session

Each task represents user-facing value that can be independently verified and tested.

### Team Collaboration (Experimental)

Leverage Claude Code's Agent Team feature to coordinate multiple roles for task completion.

```bash
/autonoe:team <task description>
```

The command automatically:
- Gathers an overview of the task from the codebase
- Breaks it down into subtasks grouped by feature
- Assigns subtasks to team members (Architect, Developer, QA, Designer, Documenter, Reviewer) based on their expertise
- Coordinates collaboration until completion

## License

Apache 2.0 License. See [LICENSE](./LICENSE) for more information.
