# remote-browser Command Reference

Complete command documentation for cloud browser automation.

## Navigation & Tabs
```bash
browser-use open <url>              # Navigate to URL
browser-use back                    # Go back in history
browser-use scroll down             # Scroll down
browser-use scroll up               # Scroll up
browser-use scroll down --amount 1000  # Scroll by specific pixels (default: 500)
browser-use switch <tab>            # Switch tab by index
browser-use close-tab               # Close current tab
browser-use close-tab <tab>         # Close specific tab
```

## Page State
```bash
browser-use state                   # Get URL, title, and clickable elements
browser-use screenshot              # Take screenshot (base64)
browser-use screenshot path.png     # Save screenshot to file
browser-use screenshot --full p.png # Full page screenshot
```

## Interactions
```bash
browser-use click <index>           # Click element
browser-use type "text"             # Type into focused element
browser-use input <index> "text"    # Click element, then type
browser-use keys "Enter"            # Send keyboard keys
browser-use keys "Control+a"        # Key combination
browser-use select <index> "option" # Select dropdown option
browser-use hover <index>           # Hover over element
browser-use dblclick <index>        # Double-click
browser-use rightclick <index>      # Right-click
```

Use indices from `browser-use state`.

## JavaScript & Data
```bash
browser-use eval "document.title"   # Execute JavaScript
browser-use get title               # Get page title
browser-use get html                # Get page HTML
browser-use get html --selector "h1"  # Scoped HTML
browser-use get text <index>        # Get element text
browser-use get value <index>       # Get input value
browser-use get attributes <index>  # Get element attributes
browser-use get bbox <index>        # Get bounding box (x, y, width, height)
```

## Cookies
```bash
browser-use cookies get             # Get all cookies
browser-use cookies get --url <url> # Get cookies for specific URL
browser-use cookies set <name> <val>  # Set a cookie
browser-use cookies set name val --domain .example.com --secure
browser-use cookies set name val --same-site Strict  # SameSite: Strict, Lax, None
browser-use cookies set name val --expires 1735689600  # Expiration timestamp
browser-use cookies clear           # Clear all cookies
browser-use cookies clear --url <url>  # Clear cookies for specific URL
browser-use cookies export <file>   # Export to JSON
browser-use cookies import <file>   # Import from JSON
```

## Wait Conditions
```bash
browser-use wait selector "h1"                         # Wait for element
browser-use wait selector ".loading" --state hidden    # Wait for element to disappear
browser-use wait text "Success"                        # Wait for text
browser-use wait selector "#btn" --timeout 5000        # Custom timeout (ms)
```

## Python Execution
```bash
browser-use python "x = 42"           # Set variable
browser-use python "print(x)"         # Access variable (prints: 42)
browser-use python "print(browser.url)"  # Access browser object
browser-use python --vars             # Show defined variables
browser-use python --reset            # Clear namespace
browser-use python --file script.py   # Run Python file
```

The Python session maintains state across commands. The `browser` object provides:
- `browser.url`, `browser.title`, `browser.html` — page info
- `browser.goto(url)`, `browser.back()` — navigation
- `browser.click(index)`, `browser.type(text)`, `browser.input(index, text)`, `browser.keys(keys)` — interactions
- `browser.screenshot(path)`, `browser.scroll(direction, amount)` — visual
- `browser.wait(seconds)`, `browser.extract(query)` — utilities

## Agent Tasks
```bash
browser-use run "Fill the contact form with test data"   # AI agent
browser-use run "Extract all product prices" --max-steps 50

# Specify LLM model
browser-use run "task" --llm gpt-4o
browser-use run "task" --llm claude-sonnet-4-20250514

# Proxy configuration (default: us)
browser-use run "task" --proxy-country uk

# Session reuse
browser-use run "task 1" --keep-alive        # Keep session alive after task
browser-use run "task 2" --session-id abc-123 # Reuse existing session

# Execution modes
browser-use run "task" --flash       # Fast execution mode
browser-use run "task" --wait        # Wait for completion (default: async)

# Advanced options
browser-use run "task" --thinking    # Extended reasoning mode
browser-use run "task" --no-vision   # Disable vision (enabled by default)

# Using a cloud profile (create session first, then run with --session-id)
browser-use session create --profile <cloud-profile-id> --keep-alive
# → returns session_id
browser-use run "task" --session-id <session-id>

# Task configuration
browser-use run "task" --start-url https://example.com  # Start from specific URL
browser-use run "task" --allowed-domain example.com     # Restrict navigation (repeatable)
browser-use run "task" --metadata key=value             # Task metadata (repeatable)
browser-use run "task" --skill-id skill-123             # Enable skills (repeatable)
browser-use run "task" --secret key=value               # Secret metadata (repeatable)

# Structured output and evaluation
browser-use run "task" --structured-output '{"type":"object"}'  # JSON schema for output
browser-use run "task" --judge                          # Enable judge mode
browser-use run "task" --judge-ground-truth "answer"
```

## Task Management
```bash
browser-use task list                     # List recent tasks
browser-use task list --limit 20          # Show more tasks
browser-use task list --status finished   # Filter by status (finished, stopped)
browser-use task list --session <id>      # Filter by session ID
browser-use task list --json              # JSON output

browser-use task status <task-id>         # Get task status (latest step only)
browser-use task status <task-id> -c      # All steps with reasoning
browser-use task status <task-id> -v      # All steps with URLs + actions
browser-use task status <task-id> --last 5  # Last N steps only
browser-use task status <task-id> --step 3  # Specific step number
browser-use task status <task-id> --reverse # Newest first

browser-use task stop <task-id>           # Stop a running task
browser-use task logs <task-id>           # Get task execution logs
```

## Cloud Session Management
```bash
browser-use session list                  # List cloud sessions
browser-use session list --limit 20       # Show more sessions
browser-use session list --status active  # Filter by status
browser-use session list --json           # JSON output

browser-use session get <session-id>      # Get session details + live URL
browser-use session get <session-id> --json

browser-use session stop <session-id>     # Stop a session
browser-use session stop --all            # Stop all active sessions

browser-use session create                          # Create with defaults
browser-use session create --profile <id>           # With cloud profile
browser-use session create --proxy-country uk       # With geographic proxy
browser-use session create --start-url https://example.com
browser-use session create --screen-size 1920x1080
browser-use session create --keep-alive
browser-use session create --persist-memory

browser-use session share <session-id>              # Create public share URL
browser-use session share <session-id> --delete     # Delete public share
```

## Cloud Profile Management
```bash
browser-use profile list                  # List cloud profiles
browser-use profile list --page 2 --page-size 50
browser-use profile get <id>              # Get profile details
browser-use profile create                # Create new profile
browser-use profile create --name "My Profile"
browser-use profile update <id> --name "New Name"
browser-use profile delete <id>
```

## Tunnels
```bash
browser-use tunnel <port>           # Start tunnel (returns URL)
browser-use tunnel <port>           # Idempotent - returns existing URL
browser-use tunnel list             # Show active tunnels
browser-use tunnel stop <port>      # Stop tunnel
browser-use tunnel stop --all       # Stop all tunnels
```

## Session Management
```bash
browser-use sessions                # List active sessions
browser-use close                   # Close current session
browser-use close --all             # Close all sessions
```
