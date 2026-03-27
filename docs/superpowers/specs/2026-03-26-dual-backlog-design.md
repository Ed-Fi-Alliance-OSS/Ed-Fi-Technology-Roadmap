# Dual Backlog Design: GitHub Issues + Jira (Option A)

**Date:** 2026-03-26
**Author:** Ed-Fi Product Management
**Status:** Draft — pending implementation planning

## Overview

Ed-Fi Alliance manages software development for several enterprise-grade open source tools serving the U.S. K-12 education community. Following a migration from self-hosted Jira Data Center to Jira Cloud, the broader software community (system administrators, system integrators, API clients) lost visibility into bugs and feature requests.

This design establishes a dual-backlog system:

| Backlog | Tool | Audience | Purpose |
|---|---|---|---|
| **Product backlog** | GitHub Issues | Staff, contractors (community eventually) | High-level bugs and feature requests |
| **Release backlog** | Jira Cloud | Staff and contractors only | Fine-grained tasks, technical debt, spikes |

## Scope

### What belongs in GitHub Issues

- Bugs reported by community members
- Bugs discovered internally that need community feedback or visibility
- New feature requests at the product level

### What stays in Jira only

- Technical subtasks and implementation detail
- Technical debt
- Research spikes
- Release planning artifacts

### Out of scope

**Planned Release issues** (created via the `PLANNED-RELEASE.yml` template) are not part of the dual-backlog workflow. They are not promoted to Jira and are not affected by any automation described in this design. The `PLANNED-RELEASE.yml` template currently instructs staff to apply product area labels; that instruction will be removed as part of this work to keep templates consistent.

### Where GitHub Issues will live

All GitHub Issues are centralized in the `Ed-Fi-Technology-Roadmap` repository. This provides a single collection point for community-facing visibility and, when the time comes, for community-submitted issues.

**Future consideration:** After triage, staff may copy an issue from `Ed-Fi-Technology-Roadmap` into the relevant product-specific repository (e.g., `Ed-Fi-ODS`) to keep it close to the code. This option is not exercised at launch but is available without changes to the automation design.

**Future consideration:** Direct issue creation by the broader community is planned but has no fixed timeline. The label schema and workflow described here are designed to accommodate community contributors without structural changes.

### Implementation prerequisites

The following items must be completed before the automation can be used:

- **New `BUG.yml` issue template** — does not currently exist; must be created with fields: Steps to Reproduce, Expected Behavior, Actual Behavior, and Environment. It should auto-apply the `bug` label via the template `labels:` field.
- **Update `PROPOSED-FEATURE.yml`** — add a `markdown` informational element prompting staff to apply the appropriate `jira-*` routing label after creation; remove the existing instruction referencing product area labels. The `jira-*` prompt is informational text, not a required form field — enforcement is handled by Action 1. The existing `labels: [feature]` auto-assignment is preserved.
- **Update `PLANNED-RELEASE.yml`** — remove the instruction to apply product area labels.
- **Create `GitHub Issue URL` custom field in Jira** — a one-time admin task; a plain text field used by Action 1 to store the GitHub Issue URL and by the Jira Automation rule to retrieve it. Must be created before any Jira tickets are promoted via the automation.

---

## Label Schema

Every GitHub Issue is labeled across three dimensions.

### Type labels
Determine the Jira issue type created during promotion.

| Label | Jira Issue Type |
|---|---|
| `bug` | Bug |
| `feature` | Story |

**Conflict and missing-label rules (enforced by Action 1):**
- If both `bug` and `feature` are present: treat as `feature` (Story)
- If neither is present: post an error comment, remove `scheduled`, and exit; the PM must add a type label before re-applying `scheduled`

### Lifecycle labels
Reflect where the issue is in the pipeline. `triaged` and `scheduled` are applied by humans; `in-jira` is applied by automation but may also be applied manually (see below).

| Label | Meaning |
|---|---|
| `triaged` | Staff has reviewed and confirmed the issue is valid |
| `scheduled` | PM has approved this for Jira promotion — trigger label |
| `in-jira` | A Jira ticket exists for this issue |

**`triaged` is a convention, not enforced by automation.** Because only staff and contractors can currently create issues, the automation does not require `triaged` to be present before promotion. Staff are expected to apply it as part of their review workflow.

**Manual use of `in-jira`:** Staff may apply this label directly when linking a GitHub Issue to a pre-existing Jira ticket, skipping the automated promotion flow. In this case, staff must also:
- Post a comment on the GitHub Issue with the Jira ticket URL
- Create a Remote Link on the Jira ticket pointing to the GitHub Issue URL (required for the Jira Automation rule to include the GitHub Issue URL in the close payload; see Action 2)

### Jira routing labels
Determine which Jira project the ticket is created in. Exactly one per issue.

| Label | Jira Project |
|---|---|
| `jira-ods` | ODS |
| `jira-ac` | AC |
| *(additional labels added as Jira projects are onboarded)* | |

**Routing label rules (enforced by Action 1):**
- If no `jira-*` label is present when `scheduled` is applied: post an error comment, remove `scheduled`, and exit; the PM must add a routing label before re-applying `scheduled`
- If more than one `jira-*` label is present: post an error comment, remove `scheduled`, and exit; the PM must remove the extra routing labels before re-applying `scheduled`

> **Note:** The existing product area labels (`ods-api-platform`, `dms-platform`, `data-standard`, `data-mgmt-tools`) are not used for automation routing. The issue templates will be updated as part of implementation to prompt for `jira-*` routing labels and remove instructions referencing the product area labels.

---

## Workflow

### 1. Creation

Staff opens a GitHub Issue using the `BUG.yml` or `PROPOSED-FEATURE.yml` template and applies:
- The appropriate type label (`bug` or `feature`) — auto-applied by the template
- The appropriate Jira routing label (`jira-*`) — prompted by the template

### 2. Triage

Staff reviews the issue and applies `triaged` if it is valid and worth tracking. Issues may remain in this state indefinitely — the GitHub product backlog has no time pressure. `triaged` is a human convention; the automation does not depend on it.

### 3. Scheduling → Jira promotion (automated)

When the PM is ready to schedule work for a release, they apply the `scheduled` label. The `jira-*` routing label may be applied before or after `scheduled` — the guard is evaluated against the full label set at event time, so both orderings trigger the action.

The "Promote to Jira" GitHub Action fires when an `issues` `labeled` event occurs and the issue has **`scheduled`** present and does **not** already have `in-jira`. The action validates routing and type labels internally and posts error comments if anything is missing. The action:
1. Creates a Jira ticket in the correct project with the correct issue type
2. Posts a comment on the GitHub Issue with the new Jira ticket URL
3. Applies `in-jira` to the GitHub Issue
4. Removes `scheduled` from the GitHub Issue (prevents re-triggering)

### 4. Manual back-linking (alternative to step 3)

If a Jira ticket already exists for the GitHub Issue, staff skip step 3 and instead:
- Apply `in-jira` manually
- Post a comment on the GitHub Issue with the Jira ticket URL
- Create a Remote Link on the Jira ticket pointing to the GitHub Issue URL

### 5. Active work

The Jira ticket drives release backlog work. The GitHub Issue remains open and publicly visible, with the Jira link in the comments.

### 6. Resolution (automated)

When the Jira ticket is marked Done, a Jira Automation rule triggers the "Close from Jira" GitHub Action, which:
1. Posts a comment on the GitHub Issue referencing the release version (see Action 2 below)
2. Closes the GitHub Issue as completed

---

## Automation Architecture

### Action 1: Promote to Jira

| Property | Value |
|---|---|
| **Trigger** | `issues` event, `labeled` action |
| **Guard** | Issue must have `scheduled` AND must NOT have `in-jira` at time of event |

The guard omits the `jira-*` requirement so that applying `scheduled` before a routing label is present will still trigger the action and surface a user-facing error (Step 2 below), rather than silently doing nothing.

**Steps:**
1. Confirm the issue does not have `in-jira`. If it does, exit without action.
2. Validate routing labels: if no `jira-*` label is present, post error comment *"This issue cannot be promoted to Jira because it has no routing label (`jira-*`). Please add one and re-apply `scheduled`."*, remove `scheduled`, and exit. If more than one `jira-*` label is present, post error comment *"This issue cannot be promoted to Jira because it has more than one routing label. Please remove the extra `jira-*` label and re-apply `scheduled`."*, remove `scheduled`, and exit.
3. Validate type labels: if neither `bug` nor `feature` is present, post error comment *"This issue cannot be promoted to Jira because it has no type label (`bug` or `feature`). Please add one and re-apply `scheduled`."*, remove `scheduled`, and exit.
4. Resolve Jira project from the single `jira-*` label
5. Resolve Jira issue type: `feature` → Story, `bug` → Bug; if both type labels are present, use Story
6. Call Jira REST API v3 to create ticket:
   - Summary ← GitHub Issue title
   - Description ← two ADF `paragraph` nodes: the first containing a `text` node with the raw GitHub Issue body (Markdown passed through as-is; no conversion), the second containing a `text` node with `GitHub Issue: <URL>`
   - `GitHub Issue URL` custom field ← GitHub Issue URL (see note below)
   - If the Jira API call fails: post error comment *"Jira ticket creation failed. Please try again or create the Jira ticket manually."*, remove `scheduled`, and exit.
7. Call Jira REST API v3 to create a Remote Link on the new ticket (`POST /rest/api/3/issue/{key}/remotelink`) with title "GitHub Issue" and the GitHub Issue URL. If this call fails, log the error but continue — the ticket was already created. Staff will need to add the Remote Link manually.

> **Note — `GitHub Issue URL` custom field:** The Jira Automation rule in Action 2 needs to read the GitHub Issue URL from the Jira ticket. Jira Automation smart value support for Remote Link URLs is not guaranteed on all Jira Cloud tiers. To ensure reliability, a custom text field named `GitHub Issue URL` must be created in Jira (a one-time admin task) and populated by Action 1 at ticket creation time. The Jira Automation rule reads from this field. If the custom field is not created, verify during implementation that `{{issue.remoteLinks}}` smart value filtering by title is available on the target Jira Cloud instance before falling back to the Remote Link approach.
8. Post comment on GitHub Issue with the new Jira ticket URL
9. Apply `in-jira` label; remove `scheduled` label

### Action 2: Close from Jira

| Property | Value |
|---|---|
| **Trigger** | `repository_dispatch` event with `event_type: jira-issue-done`, fired by a Jira Automation rule on ticket transition to Done |

**Jira Automation rule configuration:** A Jira Automation rule (not a native webhook) must be configured to fire when an issue transitions to the Done status. The rule sends an HTTP POST request to the GitHub API `repository_dispatch` endpoint with a JSON body constructed from the Jira issue's fields. Native Jira webhooks do not support custom payload construction and cannot be used here.

The rule must extract:
- `github_issue_url` from the `GitHub Issue URL` custom field on the Jira ticket (populated by Action 1; also manually populated during back-linking)
- `fix_version` from the Jira ticket's Fix Version field (may be empty)

**Expected payload structure:**

```json
{
  "event_type": "jira-issue-done",
  "client_payload": {
    "github_issue_url": "https://github.com/Ed-Fi-Alliance-OSS/Ed-Fi-Technology-Roadmap/issues/123",
    "fix_version": "v4.2"
  }
}
```

`fix_version` may be `null` or absent if no Fix Version is set on the Jira ticket.

**Steps:**
1. Parse `github_issue_url` from the `client_payload` in the `repository_dispatch` event
2. If `github_issue_url` is absent or unparseable: log the error and exit without taking action
3. Check whether the GitHub Issue is already closed. If it is, exit without action.
4. Parse `fix_version` from `client_payload`
5. Post comment:
   - If `fix_version` is present: *"This has been resolved and will be included in release \<version\>."*
   - If `fix_version` is absent or null: *"This has been resolved and will be included in an upcoming release."*
6. Close the GitHub Issue as completed (GitHub API `state_reason: completed`)

### Required secrets

Since all issues live in `Ed-Fi-Technology-Roadmap` and both actions run in the same repository, the built-in `GITHUB_TOKEN` is sufficient for GitHub API calls at launch.

| Secret | Purpose |
|---|---|
| `JIRA_API_TOKEN` | Jira Cloud access token |
| `JIRA_BASE_URL` | Jira Cloud instance URL |
| `JIRA_USER_EMAIL` | Account used for Jira API calls |
| `GITHUB_TOKEN` | Write access to GitHub Issues (built-in) |

**Future consideration:** If issues are ever copied to product-specific repositories and the actions need to close issues in those repos, a PAT with cross-repo write access will be required in place of the built-in token.

---

## Data Mapping: GitHub Issue → Jira Ticket

| GitHub Issue field | Jira Ticket field |
|---|---|
| Title | Summary |
| Body | Description — ADF paragraph node, `text` node containing raw Markdown (not converted) |
| — | Description second paragraph: `GitHub Issue: <URL>` |
| Issue URL | `GitHub Issue URL` custom field |
| `bug` label (sole type label) | Issue Type: Bug |
| `feature` label (or both type labels present) | Issue Type: Story |
| `jira-ods` label | Project: ODS |
| `jira-ac` label | Project: AC |
| Issue URL | Remote Link (title: "GitHub Issue") |

Jira fields not sourced from GitHub (assignee, sprint, priority, story points) are set by the team inside Jira during release planning.

## Data Mapping: Jira Ticket → GitHub Issue (on close)

| Jira Ticket field | GitHub Issue |
|---|---|
| `GitHub Issue URL` custom field | Source of `github_issue_url` in Jira Automation rule payload |
| Fix Version (if set) | Source of `fix_version` in Jira Automation rule payload; referenced in closing comment |

---

## Open Questions

- When community issue creation is enabled, should it be gated on a label-based permission check or simply opened to all GitHub users?
- When community issue creation is enabled, should `triaged` become a required guard condition before `scheduled` can trigger Jira promotion?
