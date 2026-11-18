# pstack in Codex

pstack skills use Cursor names. In Codex, read this file once per session, then read `~/.agents/pstack-models.md` before the first spawn, then map each Cursor name in a skill with the tables below. `<pstack>` means the clone that holds `skills/` and `docs/`, the folder two levels above the target of `readlink ~/.agents/skills/poteto-mode`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | Codex |
| --- | --- |
| `Task` | `spawn_agent`. Current models get the `collaboration` tools: `spawn_agent`, `wait_agent`, `send_message`, `followup_task`, `interrupt_agent`, `list_agents`. A model on the older set gets `spawn_agent`, `wait_agent`, `send_input`, `resume_agent`, `close_agent`. Use the set the session lists. |
| `AskQuestion` | `request_user_input`, which Codex offers only in Plan mode. Otherwise ask in plain text. In `codex exec` nobody answers, so take the recommended or current option, say which, and go on. |
| Open a todolist, or copy steps into the todolist tool | Use `update_plan` if the session offers it. Codex hides it unless `[tools.update_plan]` has `enabled = true`. Otherwise keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | `Read` `<pstack>/skills/<name>/SKILL.md` and follow it. When you tell the user to type it, write `$<name>`. Prompts start pstack with `$poteto-mode`. |
| `Grep`, `Glob` | `rg` in the shell. |
| Web search | `web_search`. It reads a cached index by default. Live results need `web_search = "live"`, or `-c web_search="live"` on `codex exec`, plus network access. |
| A step that runs `git commit`, `git push`, or `gh pr create` | Under `workspace-write`, `.git` and `.agents` stay read-only and network is off unless enabled, so these fail. Leave the changes in the tree and put the exact commands in your reply for the operator. |

## Subagent fields

| Cursor | Codex |
| --- | --- |
| Every `Task` call | Pass `task_name` (lowercase letters, digits, underscores), the brief as `message`, and `fork_turns: "none"` so the agent starts from the brief alone. The default `all` copies the whole parent history. On the older set, omit `task_name` and `fork_context`. |
| `subagent_type: generalPurpose` | Omit `agent_type`. The built-in `default` agent runs. |
| `poteto-agent`, `comment-sicko` | Codex has no such agents. Omit `agent_type` and make the brief's first line `Read <pstack>/agents/<name>.md and follow it as your role.` For `poteto-agent`, also give the absolute path `<pstack>/skills/poteto-mode/SKILL.md`, because its fallback search of `~/.cursor/plugins/` finds nothing. |
| The `model:` line in `poteto-agent`'s file | It holds a Cursor slug. Pass `model` and `reasoning_effort` on every `spawn_agent` call for a `poteto-agent` brief, mapped with the Models table, so that slug never reaches the API. |
| `readonly: true` | `agent_type: "explorer"`. Subagents share the parent's sandbox, so it can still edit. The brief also says "do not edit files", and you check `git status` afterward. |
| `readonly: false` | Omit `agent_type`. Subagents inherit the parent's sandbox, approvals, and MCP servers. |
| `environment: "cloud"` | Omit. Workers share the parent's checkout, so give each one disjoint files and say it is not alone in the tree. |
| `environment: "local"` | Omit. |
| `cloud_base_branch: <b>` | Check out `<b>` before the run. `git checkout` writes `.git`, which the sandbox keeps read-only. |
| `run_in_background: true` | Omit. Spawned agents run in parallel and report back as messages. Call `wait_agent` with a long `timeout_ms` when the next step needs them. |
| `resume` | `followup_task` with `target` and `message` while `list_agents` still shows the agent. Otherwise spawn a new agent and paste the earlier result into its brief. |
| `file_attachments` | List the paths in the brief. |

## Models

| Cursor | Codex |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh` | Use the role's line in `~/.agents/pstack-models.md`. With no line, omit `model`, pass `reasoning_effort: "<effort>"`, and say so. Drop `-fast`. |
| A `gemini-*`, `grok-*`, or other non-OpenAI slug | Omit `model` and `reasoning_effort` so the agent runs on the session model, and say so. |
| `auto` or `inherit-parent` | Omit `model` and `reasoning_effort`. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | `~/.agents/pstack-models.md`. Codex never loads it, so read it at session start. If the file or the role's line is missing, map the skill's default slug with the rows above. |
| A role value `<model> <effort>`, such as `gpt-6.2-sol high` | `model: "gpt-6.2-sol"` and `reasoning_effort: "high"`. |
| A model or effort that `spawn_agent` rejects | Never pass it again. Follow poteto-mode's fallback order, then omit both fields, and say so. |

## setup-pstack

Codex has no always-applied rule, so `setup-pstack` writes a plain file that this doc tells every session to read. Follow its steps with these changes:

- Detect. The detected models are the entries in `spawn_agent`'s "Available model overrides" list, each with its listed reasoning efforts. If the list is absent, run `codex debug models` and use the entries with `"visibility": "list"`. Write a real model as `<model> <effort>`, such as `gpt-6.2-sol xhigh`. Use only `low`, `medium`, `high`, and `xhigh`, never `max` or `ultra`. The effort is the last token, so the budget step works as written.
- Defaults. Map each default slug: `claude-opus-5-5-xhigh-fast` is `gpt-6.2-sol xhigh`, `claude-opus-5-5-high-fast` is `gpt-6.2-sol high`, `claude-fable-5-2-high` is `gpt-6.2-astra high`, and `gemini-4-carbon-high` is `inherit-parent`. Codex has no `-fast` slugs, so drop that rule.
- Vendors. Every Codex model is OpenAI. No `arena cross-judge pool` entry passes Arena's vendor check, so Phase C skips the judge. Say so when you show the roles.
- Write. Write `~/.agents/pstack-models.md` with no frontmatter, then the comment lines, the `# defaults: 3` line, the `# budget` line, and one line per role from step 5. In the second comment line, write "omit `spawn_agent` `model` and `reasoning_effort`" for the Task wording. Overwrite the whole file. Never put the role lines in `AGENTS.md`. `workspace-write` blocks writes outside the workspace, so run with `--add-dir ~/.agents`. If the write still fails, print the file and ask the user to save it.
- Confirm. Say the file applies to every session that reads this doc.

## Paths

| Cursor | Codex |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.agents/skills/`, `~/.agents/skills/`. Codex follows symlinked skill folders. |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | None. pstack is linked in as plain skill folders, one per name in `~/.agents/skills/`. Find files through `<pstack>`. |
| `.cursor/rules/*.mdc`, always-applied rules | `~/.codex/AGENTS.md`, then each `AGENTS.md` from the repo root down to the working folder. Codex never reads `.mdc` files. The model rule is the exception. See Models. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when that file exists. Read the local copy and never fetch it. Give the user the URL when they need a link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.codex/sessions/YYYY/MM/DD/rollout-<timestamp>-<id>.jsonl`. Older files may be compressed to `.jsonl.zst`. The first line is `session_meta`, whose `cwd` names the workspace. Read only files whose `cwd` is this workspace. The newest is the current session. Each subagent thread has its own file. `codex exec --ephemeral` writes none. |

## Cursor-only features

Codex has no Cursor Automations, Bugbot, Cursor cloud agents, or built-in PR tool. When a step needs one, put one line in your reply, `Skipped in Codex: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | Codex |
| --- | --- |
| Cursor Automations: routines, `update_state` routine calls, `api2.cursor.sh/automations` webhooks, routine wakes, and `SendToUser` secret requests | Skip with the line. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads, so triage them with the forge command that the step gives. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. |
| Cursor cloud agents | `environment: "cloud"` runs in the shared checkout, as in Subagent fields. Skip a step that needs the cloud agent itself, such as its dashboard status or its own machine. |
| A built-in PR tool | Use `gh pr create`, or the forge command that the step gives. That is not a skip, so write no line. When the sandbox blocks it, hand the command over as in Tools. |
