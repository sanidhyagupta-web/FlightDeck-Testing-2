---
name: create-prs
description: Push feature branches to remote and create pull requests for each affected repository.
disable-model-invocation: true
---

# Push & Create Pull Requests

Push feature branches and create PRs for each affected repository.

## Prerequisites
- All tasks complete, checks pass, user approved, CLAUDE.md updated

## Steps

### 1. Read State
Read `execution-state.md` for branch names and task list.

### 2. Push Branches
For each affected repository:
```bash
cd {repo-path}
git push -u origin {branch-name}
```

### 3. Create Pull Requests
For each affected repository:

<!-- CUSTOMIZE: Add `--draft` if an automated review pass runs before humans see the PR — see
     "Draft PRs in automated flows" below. Leave it off for human-driven work, since nothing
     marks the PR ready again on its own. -->
```bash
gh pr create --title "{TICKET-ID}: {Short summary}" --body "$(cat <<'EOF'
## Summary
- {2-4 bullet points}

## Changes
### {Service/App name}
- {Key files and what changed}

## Testing
- {Tests run and results}
- {Manual verification steps}

## References
- **Ticket**: {ticket ID}
- **Requirements**: docs/execution/{ticket}/requirements.md
- **Implementation Plan**: docs/execution/{ticket}/implementation-plan.md
- **Execution State**: docs/execution/{ticket}/execution-state.md

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 4. Link to Ticket Tracker (if available)
If a ticket tracking MCP tool is available (e.g., Linear, Jira, GitHub Issues), attach the PR URLs to the ticket:
- For Linear: use `mcp__linear__save_issue` to add the PR URL as an attachment or comment
- For Jira: use the Jira MCP to post a comment with the PR URLs
- If no MCP is available, remind the user to manually link the PR to the ticket

### 5. Update Execution State
Set status to `PR_CREATED`. Record PR URLs.

### 6. Report
Provide PR links, summary of each PR, and next steps.

## Draft PRs in automated flows

Opening the PR as a draft is the right default **only** when something later marks it ready — an
automated review pass, a CI gate, a headless execution loop. A draft signals "not yet worth a
human's attention", and a PR nobody un-drafts is a PR nobody reviews.

**Human-driven work:** leave `--draft` off. That is the default above.

**Automated flows:** add `--draft` to the `gh pr create` call, and have the automation flip the PR
when its loop finishes:

```bash
gh pr ready {PR_NUMBER}
```

Flip on **every** terminal state, not just the clean one. A run that timed out, exhausted its
budget, or escalated is equally done and equally needs human eyes — restricting the flip to the
success path leaves failed runs stuck in draft indefinitely, which is the exact outcome draft
status is supposed to prevent.

`gh pr ready` is idempotent, so a retried step will not fail on an already-ready PR. Note that the
REST API has no "undraft" endpoint; `gh` uses the GraphQL `markPullRequestReadyForReview` mutation
underneath, which matters if you are calling the API directly rather than shelling out to `gh`.

## Rules
- Always push with `-u` flag
- Do NOT force push
- One PR per repository with changes
- PR title follows the repo's commit format
- Do not merge — that's a manual step
