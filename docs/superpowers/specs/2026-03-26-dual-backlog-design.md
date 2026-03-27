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

### Where GitHub Issues will live

To be decided: either centralized in this `Technology-Roadmap` repository or distributed across individual product repositories. The design works in either location.

---

## Label Schema

Every GitHub Issue is labeled across three dimensions.

### Type labels
Determine the Jira issue type created during promotion.

| Label | Jira Issue Type |
|---|---|
| `bug` | Bug |
| `feature` | Story |

### Lifecycle labels
Reflect where the issue is in the pipeline. `triaged` and `scheduled` are applied by humans; `in-jira` is applied by automation but may also be applied manually (see below).

| Label | Meaning |
|---|---|
| `triaged` | Staff has reviewed and confirmed the issue is valid |
| `scheduled` | PM has approved this for Jira promotion — trigger label |
| `in-jira` | A Jira ticket exists for this issue |

**Manual use of `in-jira`:** Staff may apply this label directly when linking a GitHub Issue to a pre-existing Jira ticket, skipping the automated promotion flow. In this case, staff should also post a comment with the Jira ticket URL.

### Jira routing labels
Determine which Jira project the ticket is created in. One per issue.

| Label | Jira Project |
|---|---|
| `jira-ods` | ODS |
| `jira-ac` | AC |
| *(additional labels added as Jira projects are onboarded)* | |

> **Note:** The existing product area labels (`ods-api-platform`, `dms-platform`, `data-standard`, `data-mgmt-tools`) are not used for automation routing and should be evaluated for retirement or repurposed for community-facing categorization only.

---

## Workflow

### 1. Creation

Staff opens a GitHub Issue using the `bug` or `feature` template and applies:
- The appropriate type label (`bug` or `feature`)
- The appropriate Jira routing label (`jira-*`)

### 2. Triage

Staff reviews the issue and applies `triaged` if it is valid and worth tracking. Issues may remain in this state indefinitely — the GitHub product backlog has no time pressure.

### 3. Scheduling → Jira promotion (automated)

When the PM is ready to schedule work for a release, they apply the `scheduled` label.

The "Promote to Jira" GitHub Action fires when it detects that an issue has **both** `scheduled` and a `jira-*` label. The action:
1. Creates a Jira ticket in the correct project with the correct issue type
2. Posts a comment on the GitHub Issue with the new Jira ticket URL
3. Applies `in-jira` to the GitHub Issue
4. Removes `scheduled` from the GitHub Issue (prevents re-triggering)

### 4. Manual back-linking (alternative to step 3)

If a Jira ticket already exists for the GitHub Issue, staff skip step 3 and instead:
- Apply `in-jira` manually
- Post a comment on the GitHub Issue with the Jira ticket URL

### 5. Active work

The Jira ticket drives release backlog work. The GitHub Issue remains open and publicly visible, with the Jira link in the comments.

### 6. Resolution (automated)

When the Jira ticket is marked Done, a Jira webhook triggers the "Close from Jira" GitHub Action, which:
1. Posts a comment on the GitHub Issue: *"This has been resolved and will be included in an upcoming release."*
2. Closes the GitHub Issue

---

## Automation Architecture

### Action 1: Promote to Jira

| Property | Value |
|---|---|
| **Trigger** | `issues` event, `labeled` action |
| **Guard** | Issue must have both `scheduled` AND a `jira-*` label at time of event |

**Steps:**
1. Read issue title, body, type label, and `jira-*` label
2. Resolve Jira project from `jira-*` label
3. Resolve Jira issue type: `bug` → Bug, `feature` → Story
4. Call Jira REST API to create ticket:
   - Summary ← GitHub Issue title
   - Description ← GitHub Issue body + GitHub Issue URL in footer
   - Create a Jira Remote Link pointing to the GitHub Issue URL
5. Post comment on GitHub Issue with the new Jira ticket URL
6. Apply `in-jira` label; remove `scheduled` label

### Action 2: Close from Jira

| Property | Value |
|---|---|
| **Trigger** | `repository_dispatch` event, fired by Jira webhook on ticket transition to Done |

**Steps:**
1. Parse the GitHub Issue URL from the Jira webhook payload (sourced from the Remote Link stored during creation)
2. Post comment: *"This has been resolved and will be included in an upcoming release."*
3. Close the GitHub Issue

### Required secrets

| Secret | Purpose |
|---|---|
| `JIRA_API_TOKEN` | Jira Cloud access token |
| `JIRA_BASE_URL` | Jira Cloud instance URL |
| `JIRA_USER_EMAIL` | Account used for Jira API calls |
| `GITHUB_TOKEN` | Write access to GitHub Issues (built-in for same-repo actions; PAT if cross-repo) |

---

## Data Mapping: GitHub Issue → Jira Ticket

| GitHub Issue field | Jira Ticket field |
|---|---|
| Title | Summary |
| Body | Description |
| `bug` label | Issue Type: Bug |
| `feature` label | Issue Type: Story |
| `jira-ods` label | Project: ODS |
| `jira-ac` label | Project: AC |
| Issue URL | Remote Link + Description footer |

Jira fields not sourced from GitHub (assignee, sprint, priority, story points) are set by the team inside Jira during release planning.

---

## Open Questions

- Where will GitHub Issues live? (centralized in `Technology-Roadmap` vs. distributed per product repo)
- Should the closing comment reference the specific release version once known?
- Should the `triaged` step be enforced (i.e., block `scheduled` from being applied unless `triaged` is present), or left as a convention?
- What should happen if both `bug` and `feature` labels are applied? (Guard or default behavior needed)
- When will issue creation be opened to the broader community?
