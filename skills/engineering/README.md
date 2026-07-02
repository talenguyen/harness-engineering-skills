# Engineering Skills

Small Harness Engineering skills for AI coding agents. Install the whole set or
only the station your workflow currently needs.

## User-invoked

| Skill | Role |
|---|---|
| [`setup-harness-engineering`](./setup-harness-engineering/SKILL.md) | Prepare a repo: rules, docs location, gate map |
| [`running-the-line`](./running-the-line/SKILL.md) | Orchestrate one task through all five stations |

## Model-invoked

| Skill | Role |
|---|---|
| [`work-order`](./work-order/SKILL.md) | Define correctness before implementation |
| [`tdd-plan-gate`](./tdd-plan-gate/SKILL.md) | Turn the work order into tests and stop before coding |
| [`andon-cord`](./andon-cord/SKILL.md) | Add hard-failing gates and failure recovery rules |
| [`inspection`](./inspection/SKILL.md) | Review a diff against the work order and judgment calls |
| [`field-return`](./field-return/SKILL.md) | Convert escaped defects into upstream checks |

Start with `running-the-line` for a full task or `work-order` if your current
agent workflow keeps building the wrong thing correctly.

