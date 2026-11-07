# Hermes delegation dispatch

Applies when the session exposes `delegate_task` and no Agent Teams tools.

## Facts

- A child starts with no knowledge of the parent conversation. It receives only `goal` and `context`, so the dispatch message must be self-contained.
- Children cannot call `clarify`, `memory`, `send_message` or `cronjob`. A child cannot ask the user a question mid-run; it can only return text or a schema-validated object.
- Children cannot delegate further unless the task uses `role="orchestrator"` and `delegation.max_spawn_depth` is at least 2. Keep the default: role work happens in one level.
- A child loads its role skill with `skill_view` as its first action.
- Results arrive as a completion message after the Lead ends its turn. The Lead must finish independent work and end the turn instead of polling.
- `output_schema` makes the child return JSON: a valid object on success, and a schema error when the child cannot fill it. When `schema_valid` is false, read the raw text for the deliverable paths.
- Project context files (`AGENTS.md` and similar) are injected automatically; still name the exact paths in `context`.
- The user can run a single role with `/role-<slug>`, with `/personality <slug>` when a persona is configured, or with `hermes chat -s role-<slug>`.
- `/review` dispatches a built-in reviewer that is independent of `role-reviewer`. It may be used in addition to the gate, never instead of it.

## Procedure per stage

1. Build the dispatch message with the nine items of the Dispatch contract. Because the child cannot ask questions, item 9 reads: `If a required input is missing, stop and return the question as your final answer; do not guess.`
2. Call `delegate_task` with `goal` = `Stage <n>: <stage name> as role-<slug>` and `context` = the dispatch message. Add `output_schema` from the template below.
3. End the turn. When the completion arrives, read `summary`. If `schema_valid` is false, extract the paths from the raw text.
4. Open the deliverables and check the Definition of Done yourself.
5. For the gate call `delegate_task` for `role-reviewer` with the deliverable paths, the DoD of the stage and the revision under review; parse the `verdict`.
6. On `request-changes`, dispatch the author role again with the review path in `Inputs`.
7. Stages 5 and 5p: pass both tasks in one `delegate_task(tasks=[...])` call. The tester task keeps its write scope free of source directories.
8. After every stage, update `docs/delivery/plan.md` with the status, the deliverable paths and the verdict.

## Output schema

```json
{
  "type": "object",
  "properties": {
    "deliverables": {"type": "array", "items": {"type": "string"}},
    "dod": {"type": "array", "items": {"type": "object", "properties": {"item": {"type": "string"}, "status": {"type": "string", "enum": ["pass", "fail"]}}, "required": ["item", "status"]}},
    "open_questions": {"type": "array", "items": {"type": "string"}},
    "risks": {"type": "array", "items": {"type": "string"}},
    "verdict": {"type": "string", "enum": ["approve", "request-changes", "n/a"]},
    "summary_ru": {"type": "string"}
  },
  "required": ["deliverables", "dod", "open_questions", "summary_ru"]
}
```

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
Missing input rule: If a required input is missing, stop and return the question as your final answer; do not guess.
```

## Configuration hints

- `skills.external_dirs` must include the directory that holds the role skill folders, otherwise `skill_view` cannot load `role-<slug>`.
- `agent.personalities` may define one persona per role, for example `developer` or `reviewer`; see `hermes/config.snippet.yaml` in this repository for the snippet.
- `delegation.max_concurrent_children` default 10 is enough for the eight roles plus a reviewer; no change is required.
- `delegation.worktree_isolation: true` is recommended when stage 5 and stage 5p run in parallel inside a git repository.
- The Lead is the only orchestrator: it keeps the plan, the gates and the final Russian summary, and it never edits stage artifacts itself.
