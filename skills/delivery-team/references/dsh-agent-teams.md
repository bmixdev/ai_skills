# DSH Agent Teams dispatch

Applies when the session exposes `spawn_teammate`, `send_message`, `list_agents`, `wait_agent`, `interrupt_agent`, `team_task_create`, `team_task_get`, `team_task_list` and `team_task_update`.

## Facts

- All teammates share the Lead's working directory and filesystem. There is no per-teammate checkout unless the Lead creates one.
- `spawn_teammate` with `context: "fresh"` starts without the Lead's history; `"fork"` inherits completed Lead turns. Use `fresh` by default so that the role reads only its dispatch message and its skill.
- Teammate names are permanent, unique and lowercase kebab-case: `business`, `enterprise`, `solution`, `analyst`, `developer`, `tester`, `devops`, `reviewer-stage-1`.
- Only the Lead creates, interrupts or reassigns teammates. A teammate cannot spawn another teammate.
- `write_scopes` on a shared task produce a warning on overlap; they are not a lock. The Lead enforces one writer per scope by ordering tasks.
- `wait_agent` returns immediately with `noProgress` when no teammate is running. Call `list_agents` first; if the teammate is `inactive` and its task is not completed, `send_message` it before waiting again.
- A teammate loads its role skill with the `skill` tool as its first action.
- A single role without a team is run with the `subagent` tool, or by the user typing `/role-<slug>` in the chat.
- The shared board is the source of stage status: `pending`, `in_progress`, `completed`. The status column of `docs/delivery/plan.md` mirrors the board.

## Procedure per stage

1. `team_task_create` with `subject` = stage name and role, `description` = the full dispatch message (nine items of the Dispatch contract), `write_scopes` = the row of `handoff-contracts.md` for the role, and `blocked_by` = ids of the prerequisite stage tasks.
2. `spawn_teammate` with `name` = the slug, `description` = the role title, `context` = `fresh`, and `prompt` = the same dispatch message plus `Claim task <id> on the shared board, complete it when done.`
3. Bounded wait: call `wait_agent` once with `timeout_ms` 600000. If it returns `timedOut: false`, the state changed: call `list_agents` once and act on the result. If it returns `timedOut: true`, call `wait_agent` a second time, once. If that second wait also times out, stop waiting: report the blocker to the user and end the turn. A teammate that is still running never justifies a third wait in the same turn. When the teammate is `inactive` and the task is `completed`, read the deliverables.
4. Never poll. Do not call `list_agents`, `team_task_list` or `get_goal` repeatedly while waiting; each call resends the whole context and costs a full request. Call them only when `wait_agent` reports a state change. A wait that returns `timedOut: false` is not progress by itself: if the only change is that the same teammate is still running, end the turn instead of waiting again.
5. If the teammate is `inactive` and the task is still `in_progress`, `send_message` asking for the status report, then use one bounded wait from step 3.
6. For the review gate spawn `reviewer-stage-<n>`. Reuse the same reviewer teammate for later stages by `send_message` with a new dispatch message.
7. On `request-changes`, `send_message` the author teammate with the review path and `Fix the findings, re-run your Quality checklist, report again.`
8. Call `interrupt_agent` only when a teammate ignores its write scope or loops. Then re-dispatch with a narrower prompt.
9. After a stage closes, mark the task `completed` and update `docs/delivery/plan.md` before starting the next stage.

## Dispatch message template

```text
Role: <role title>
First action: load the skill "role-<slug>" and follow it exactly.
Project root: <absolute path of the target project>
Stage and size: stage <n> (<stage name>), size <S|M|L>
Inputs:
- <label>: <path>
Deliverables:
- <path>
Definition of Done:
- [ ] <DoD item copied from references/handoff-contracts.md>
Write scope: <allowed directories from the Write scopes table>; everything else is read-only.
Report: final message in Russian with deliverable paths, DoD status per item, open questions, risks.
Missing input rule: stop and return one precise question instead of guessing.
```

One role per subagent; the subagent name equals the slug. Two subagents never share a write scope at the same time.

## Parallel stages

Stage 5 and stage 5p run together: create both tasks, keep `blocked_by` on the same stage 4 task, spawn `developer` and `tester` in the same message, and confirm the write scopes do not overlap. The tester writes only test directories, `docs/test/` and the `TC` column of `docs/requirements/traceability.md`; the developer writes source, tests and `docs/dev/`. Wait for both, then open the review gate for stage 5 and merge the stage 5p review into the stage 6 gate.

## Limits

- A team holds at most 16 members. Reuse the `reviewer` and `tester` teammates across stages instead of spawning new ones; only the first reviewer gets a stage suffix.
- Keep every dispatch message under 8000 characters by passing paths, never document contents.
- The Lead opens the deliverable files itself; a teammate summary is not evidence.
- The board accumulates tasks; archive closed stages in the plan instead of deleting tasks.
- Token budget per turn: at most two `wait_agent` calls per teammate per turn, and at most one `list_agents` call after each state change. Every extra status call resends the entire context as a new request. Polling was measured at 84.5 percent of the input tokens of one real orchestration session; a turn that only waits and rereads status is waste, not diligence.
- End the turn instead of waiting. If nothing can advance the work right now, report the current state and stop. The harness wakes the Lead when a teammate reports; it does not require an open wait.
