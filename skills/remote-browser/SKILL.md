---
name: remote-browser
description: Automates web browsing via a cloud browser for sandboxed or headless environments. Use when the user needs to open URLs, click buttons, fill forms, take screenshots, scrape pages, or expose local dev servers — and no local GUI or Chrome is available.
allowed-tools: Bash(browser-use:*)
---

# Remote Browser Automation for Sandboxed Agents

This skill is for agents running on **sandboxed remote machines** (cloud VMs, CI, coding agents) that need to control a browser. Install `browser-use` and drive a cloud browser — no local Chrome needed.

## Prerequisites

Before using this skill, `browser-use` must be installed and configured. Run diagnostics to verify:

```bash
browser-use doctor
```

For more information, see https://github.com/browser-use/browser-use/blob/main/browser_use/skill_cli/README.md

## Core Workflow

Commands use the cloud browser:

```bash
# Step 1: Start session (automatically uses remote mode)
browser-use open https://example.com
# Returns: url, live_url (view the browser in real-time)

# Step 2+: All subsequent commands use the existing session
browser-use state                   # Get page elements with indices
browser-use click 5                 # Click element by index
browser-use type "Hello World"      # Type into focused element
browser-use input 3 "text"          # Click element, then type
browser-use screenshot              # Take screenshot (base64)
browser-use screenshot page.png     # Save screenshot to file

# Done: Close the session
browser-use close                   # Close browser and release resources
```

See [references/REFERENCE.md](references/REFERENCE.md) for the complete command reference (interactions, cookies, JavaScript, Python, agent tasks, profiles, tunnels, and more).

## Common Workflows

### Exposing Local Dev Servers

Use when you have a dev server on the remote machine and need the cloud browser to reach it.

**Core workflow:** Start dev server → create tunnel → browse the tunnel URL.

```bash
# 1. Start your dev server
python -m http.server 3000 &

# 2. Expose it via Cloudflare tunnel
browser-use tunnel 3000
# → url: https://abc.trycloudflare.com

# 3. Now the cloud browser can reach your local server
browser-use open https://abc.trycloudflare.com
browser-use state
browser-use screenshot
```

**Note:** Tunnels are independent of browser sessions. They persist across `browser-use close` and can be managed separately. Cloudflared must be installed — run `browser-use doctor` to check.

### Running Subagents

Use cloud sessions to run autonomous browser agents in parallel.

**Core workflow:** Launch task(s) with `run` → poll with `task status` → collect results → clean up sessions.

- **Session = Agent**: Each cloud session is a browser agent with its own state
- **Task = Work**: Jobs given to an agent; an agent can run multiple tasks sequentially
- **Session lifecycle**: Once stopped, a session cannot be revived — start a new one

#### Launching Tasks

```bash
# Single task (async by default — returns immediately)
browser-use run "Search for AI news and summarize top 3 articles"
# → task_id: task-abc, session_id: sess-123

# Parallel tasks — each gets its own session
browser-use run "Research competitor A pricing"
# → task_id: task-1, session_id: sess-a
browser-use run "Research competitor B pricing"
# → task_id: task-2, session_id: sess-b
browser-use run "Research competitor C pricing"
# → task_id: task-3, session_id: sess-c

# Sequential tasks in same session (reuses cookies, login state, etc.)
browser-use run "Log into example.com" --keep-alive
# → task_id: task-1, session_id: sess-123
browser-use task status task-1  # Wait for completion
browser-use run "Export settings" --session-id sess-123
# → task_id: task-2, session_id: sess-123 (same session)
```

#### Managing & Stopping

```bash
browser-use task list --status finished      # See completed tasks
browser-use task stop task-abc               # Stop a task (session may continue if --keep-alive)
browser-use session stop sess-123            # Stop an entire session (terminates its tasks)
browser-use session stop --all               # Stop all sessions
```

#### Monitoring

**Task status is designed for token efficiency.** Default output is minimal — only expand when needed:

| Mode | Flag | Tokens | Use When |
|------|------|--------|----------|
| Default | (none) | Low | Polling progress |
| Compact | `-c` | Medium | Need full reasoning |
| Verbose | `-v` | High | Debugging actions |

```bash
# For long tasks (50+ steps)
browser-use task status <id> -c --last 5   # Last 5 steps only
browser-use task status <id> -v --step 10  # Inspect specific step
```

**Live view**: `browser-use session get <session-id>` returns a live URL to watch the agent.

**Detect stuck tasks**: If cost/duration in `task status` stops increasing, the task is stuck — stop it and start a new agent.

**Logs**: `browser-use task logs <task-id>` — only available after task completes.

## Global Options

| Option | Description |
|--------|-------------|
| `--session NAME` | Named session (default: "default") |
| `--browser MODE` | Browser mode (only if multiple modes installed) |
| `--profile ID` | Cloud profile ID for persistent cookies. Works with `open`, `session create`, etc. — does NOT work with `run` (use `--session-id` instead) |
| `--json` | Output as JSON |

## Tips

1. **Run `browser-use doctor`** to verify installation before starting
2. **Always run `state` first** to see available elements and their indices
3. **Sessions persist** across commands — the browser stays open until you close it
4. **Tunnels are independent** — they persist across `browser-use close`
5. **Use `--json`** for programmatic parsing
6. **`tunnel` is idempotent** — calling it again for the same port returns the existing URL

## Troubleshooting

**"Browser mode 'chromium' not installed"?**
- Expected for sandboxed agents — remote mode only supports cloud browsers
- Run `browser-use doctor` to verify configuration

**Cloud browser won't start?**
- Run `browser-use doctor` to check configuration

**Tunnel not working?**
- Verify cloudflared is installed: `which cloudflared`
- `browser-use tunnel list` to check active tunnels
- `browser-use tunnel stop <port>` and retry

**Element not found?**
- Run `browser-use state` to see current elements
- `browser-use scroll down` then `browser-use state` — element might be below fold

**Session reuse fails after `task stop`**:
Create a new session instead:
```bash
browser-use session create --profile <profile-id> --keep-alive
browser-use run "new task" --session-id <new-session-id>
```

**Task stuck at "started"**: Check cost with `task status` — if not increasing, the task is stuck. View live URL with `session get`, then stop and start a new agent.

**Sessions persist after tasks complete**: Run `browser-use session stop --all` to clean up.

## Cleanup

**Always close resources when done:**

```bash
browser-use close                     # Close browser session
browser-use session stop --all        # Stop cloud sessions (if any)
browser-use tunnel stop --all         # Stop tunnels (if any)
```
