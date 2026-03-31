---
name: todoist
description: >
  Invoke this skill for any request involving the user's Todoist tasks, to-do list,
  or projects. Handles: adding tasks with due dates/priorities/projects, completing
  or checking off tasks, viewing today's or upcoming tasks, reviewing recently
  completed work, moving tasks to different projects, creating new projects, and
  setting priorities (p1-p4). Also invoke for natural-language task management
  queries ("what's on my plate today?", "what did I finish this week?", "mark that
  done") -- these require the td CLI to pull live Todoist data and cannot be
  answered without it.
---

# Todoist CLI (td)

## Behavioral guidelines

- Default to human-readable output for single-item views; use `--json` when
  you need to process or filter results programmatically (e.g., counting, sorting)
- Before deleting or completing a task matched by fuzzy name, show what was matched
  and confirm if the match seems ambiguous
- Prefer `id:xxx` references when operating on a specific item found via a
  previous list command, to avoid accidental fuzzy-match errors
- For multi-step workflows (e.g., "move all p1 tasks from Inbox to Work"),
  list first, confirm the set, then act

## Quick Reference

- `td today` - Tasks due today and overdue
- `td inbox` - Inbox tasks
- `td upcoming` - Tasks due in next N days
- `td completed` - Recently completed tasks
- `td task add "content"` - Add a task
- `td task list` - List tasks with filters
- `td task complete <ref>` - Complete a task
- `td project list` - List projects
- `td label list` - List labels
- `td filter list/view` - Manage and use saved filters
- `td workspace list` - List workspaces
- `td activity` - Activity logs
- `td notification list` - Notifications
- `td reminder add` - Task reminders
- `td stats` - Productivity stats
- `td settings view` - User settings
- `td completion install` - Install shell completions
- `td view <url>` - View supported Todoist entities/pages by URL
- `td update` - Self-update the CLI to the latest version
- `td changelog` - Show recent changelog entries
- `td attachment view <url>` - View/download a file attachment by URL
- `td auth login/token/status/logout` - Manage authentication
- `td skill install/update/uninstall/list` - Manage coding agent skills
- `td template export-file/export-url/create/import-file/import-id` - Manage project templates

## Output Formats

All list commands support:
- `--json` - JSON output (essential fields)
- `--ndjson` - Newline-delimited JSON (streaming)
- `--full` - Include all fields in JSON
- `--raw` - Disable markdown rendering

## Shared List Options

Most list commands also support:
- `--limit <n>` - Limit number of results
- `--all` - Fetch all results (no limit)
- `--cursor <cursor>` - Continue from pagination cursor
- `--show-urls` - Show web app URLs for each item

## Global Options

- `--no-spinner` - Disable loading animations
- `--progress-jsonl` - Machine-readable progress events (JSONL to stderr)
- `-v, --verbose` - Verbose output to stderr (repeat: -v info, -vv detail, -vvv debug, -vvvv trace)
- `--accessible` - Add text labels to color-coded output (due:/deadline:/~ prefixes, ★ for favorites). Also: `TD_ACCESSIBLE=1`

## References

Tasks, projects, labels, and filters can be referenced by:
- Name (fuzzy matched within context)
- `id:xxx` - Explicit ID
- Todoist URL - Paste directly from the web app (e.g., `https://app.todoist.com/app/task/buy-milk-8Jx4mVr72kPn3QwB` or `https://app.todoist.com/app/project/work-2pN7vKx49mRq6YhT`)

## Priority Mapping

- p1 = Highest priority (API value 4)
- p2 = High priority (API value 3)
- p3 = Medium priority (API value 2)
- p4 = Lowest priority (API value 1, default)

## Commands

### Today
```bash
td today                             # Due today + overdue
td today --json                      # JSON output
td today --workspace "Work"          # Filter to workspace
td today --personal                  # Personal projects only
td today --any-assignee              # Include tasks assigned to others
```

### Inbox
```bash
td inbox                             # Inbox tasks
td inbox --priority p1               # Filter by priority
td inbox --due today                 # Filter by due date
```

### Upcoming
```bash
td upcoming                          # Next 7 days
td upcoming 14                       # Next 14 days
td upcoming --workspace "Work"       # Filter to workspace
td upcoming --personal               # Personal projects only
td upcoming --any-assignee           # Include tasks assigned to others
```

### Completed
```bash
td completed                         # Completed today
td completed --since 2024-01-01 --until 2024-01-31
td completed --project "Work"        # Filter by project
```

### Quick Add (human shorthand)
```bash
td add "Buy milk tomorrow p1 #Shopping"   # Natural language quick-add
td add "Call dentist next Monday #Health" # Date, project, priority parsed automatically
```
Use `td add` when the user provides a natural-language task string. Use `td task add` (below) when constructing tasks programmatically with explicit flags — it's more reliable for agents.

### Task Management
```bash
# List with filters
td task list --project "Work"
td task list --label "urgent" --priority p1
td task list --due today
td task list --filter "today | overdue"
td task list --assignee me              # me or id:xxx (email not supported here)
td task list --unassigned
td task list --workspace "Work"
td task list --personal
td task list --parent "Parent task"

# View, complete, uncomplete
td task view "task name"
td task complete "task name"
td task complete id:123456
td task complete "task name" --forever  # Stop recurrence
td task uncomplete id:123456            # Reopen completed task

# Add tasks
td task add "New task" --due "tomorrow" --priority p2
td task add "Task" --deadline "2024-03-01" --project "Work"
td task add "Task" --duration 1h --section "Planning" --project "Work"
td task add "Task" --labels "urgent,review" --parent "Parent task"
td task add "Task" --description "Details here" --assignee me

# Update
td task update "task name" --content "New title"
td task update "task name" --due "next week"
td task update "task name" --priority p1
td task update "task name" --labels "urgent,review"   # replaces existing labels
td task update "task name" --description "More details"
td task update "task name" --deadline "2024-06-01"
td task update "task name" --no-deadline              # remove deadline
td task update "task name" --duration 2h
td task update "task name" --assignee "john@example.com"  # name, email, id:xxx, or "me"
td task update "task name" --unassign

# Move
td task move "task name" --project "Personal"
td task move "task name" --section "In Progress"
td task move "task name" --parent "Parent task"
td task move "task name" --no-parent          # Move to project root
td task move "task name" --no-section         # Remove from section

# Delete and browse
td task delete "task name" --yes
td task browse "task name"                    # Open in browser
```

### Projects
```bash
td project list
td project list --personal                    # Personal projects only
td project view "Project Name"
td project collaborators "Project Name"
td project create --name "New Project" --color "blue"
td project update "Project Name" --favorite
td project archive "Project Name"
td project unarchive "Project Name"
td project delete "Project Name" --yes
td project browse "Project Name"              # Open in browser
td project move "Project Name" --to-workspace "Acme"
td project move "Project Name" --to-workspace "Acme" --folder "Engineering"
td project move "Project Name" --to-workspace "Acme" --visibility team
td project move "Project Name" --to-personal
# move requires --yes to confirm (without it, shows a dry-run preview)
```

### Labels
```bash
td label list                                 # Lists personal + shared labels
td label view "urgent"                        # View label details and tasks
td label view "team-review"                   # Works for shared labels too
td label create --name "urgent" --color "red"
td label update "urgent" --color "orange"
td label delete "urgent" --yes
td label browse "urgent"                      # Open in browser
```

Note: Shared labels (from collaborative projects) appear in `list` and can be viewed, but cannot be deleted/updated via the standard label commands since they have no ID.

### Comments
```bash
td comment list --task "task name"
td comment list --project "Project Name" -P   # Project comments
td comment add --task "task name" --content "Comment text"
td comment add --task "task name" --content "See attached" --file ./report.pdf
td comment view id:123                        # View full comment
td comment update id:123 --content "Updated text"
td comment delete id:123 --yes
td comment browse id:123                      # Open in browser
```

### Sections
```bash
td section list "Work"                        # List sections in project (or --project "Work")
td section list --project "Work"              # Same, using named flag
td section create --project "Work" --name "In Progress"
td section update id:123 --name "Done"
td section delete id:123 --yes
td section browse id:123                      # Open in browser
```

### Filters
```bash
td filter list
td filter create --name "Urgent work" --query "p1 & #Work"
td filter view "Urgent work"                  # Show tasks matching filter (alias: show)
td filter update "Urgent work" --query "p1 & #Work & today"
td filter delete "Urgent work" --yes
td filter browse "Urgent work"                # Open in browser
```

### Workspaces
```bash
td workspace list
td workspace view "Workspace Name"
td workspace projects "Workspace Name"        # or --workspace "Workspace Name"
td workspace users "Workspace Name" --role ADMIN,MEMBER  # or --workspace "..."
```

### Activity
```bash
td activity                                   # Recent activity
td activity --since 2024-01-01 --until 2024-01-31
td activity --type task --event completed
td activity --project "Work"
td activity --by me
```

### Notifications
```bash
td notification list
td notification list --unread
td notification list --type "item_assign"
td notification view id:123
td notification read --all --yes              # Mark all as read
td notification accept id:123                 # Accept share invitation
td notification reject id:123                 # Reject share invitation
```

### Reminders
```bash
td reminder list "task name"                  # or --task "task name"
td reminder add "task name" --before 30m      # or --task "task name" --before 30m
td reminder add "task name" --at "2024-01-15 10:00"
td reminder update id:123 --before 1h
td reminder delete id:123 --yes
```

### Stats
```bash
td stats                                      # View karma and productivity
td stats --json
td stats goals --daily 10 --weekly 50
td stats vacation --on                        # Enable vacation mode
td stats vacation --off                       # Disable vacation mode
```

### Settings
```bash
td settings view
td settings view --json
td settings update --timezone "America/New_York"
td settings update --time-format 24 --date-format intl
td settings themes                            # List available themes
```

### Shell Completions
```bash
td completion install                         # Install tab completions (prompts for shell)
td completion install bash                    # Install for specific shell
td completion install zsh
td completion install fish
td completion uninstall                       # Remove completions
```

### View (URL Router)
```bash
td view <todoist-url>                          # Auto-route to appropriate view by URL type
td view https://app.todoist.com/app/task/buy-milk-abc123
td view https://app.todoist.com/app/project/work-def456
td view https://app.todoist.com/app/label/urgent-ghi789
td view https://app.todoist.com/app/filter/work-tasks-jkl012
td view https://app.todoist.com/app/today
td view https://app.todoist.com/app/upcoming
td view <url> --json                           # JSON output for entity views
td view <url> --limit 25 --ndjson              # Passthrough list options where supported
```

### Update
```bash
td update                                    # Update CLI to latest version
td update --check                            # Check for updates without installing
```

### Changelog
```bash
td changelog                                 # Show last 5 versions
td changelog -n 10                           # Show last 10 versions
```

### Attachments
```bash
td attachment view <url>                     # View/download a file attachment by URL
td attachment view <url> --json              # JSON output with metadata and content
```

### Auth
```bash
td auth login                                # Authenticate via OAuth
td auth token <token>                        # Save API token directly
td auth status                               # Show current auth status
td auth logout                               # Remove saved token
```

### Skill (coding agent integrations)
```bash
td skill list                                # List supported agents and install status
td skill install                             # Install skill for a coding agent
td skill update                              # Update installed skill to latest version
td skill uninstall                           # Uninstall skill for a coding agent
```

### Templates
```bash
td template export-file "Project Name"       # Export project as CSV template file
td template export-url "Project Name"        # Export project as template URL
td template create                           # Create new project from a template file
td template import-file "Project Name"       # Import a template file into existing project
td template import-id "Project Name"         # Import a template by ID into existing project
```

## Examples

### Daily workflow
```bash
td today --json | jq '.results | length'      # Count today's tasks
td inbox --limit 5                             # Quick inbox check
td upcoming                                    # What's coming this week
td completed                                   # What I finished today
```

### Filter by multiple criteria
```bash
td task list --project "Work" --label "urgent" --priority p1
td task list --filter "today & #Work"
td task list --workspace "Work" --due today
```

### Complete tasks efficiently
```bash
td task complete "Review PR"
td task complete id:123456789
td task uncomplete id:123456789                # Reopen if needed
```
