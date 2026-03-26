# Dual Backlog: GitHub Issues + Jira — Options

**Date:** 2026-03-25
**Author:** Ed-Fi Product Management

## Background

Ed-Fi Alliance manages software development for several enterprise-grade open source tools serving the U.S. K-12 education community. The primary tool is the **Ed-Fi API**, a central interoperability hub for collecting and sharing data across disparate education systems.

The software community includes three groups:
- **System administrators** — run the Ed-Fi API and related applications
- **System integrators** — contracted to install or operate software on behalf of education organizations
- **API clients** — push or pull data via the Ed-Fi API

When migrating from Jira Data Center (self-hosted) to Jira Cloud, it became cost-prohibitive to give all community members access. Development work continues in Jira, but there is no community visibility into bugs or feature requests.

## Proposed Solution

Use **two backlogs** with distinct purposes:

| Backlog | Tool | Audience | Granularity |
|---|---|---|---|
| **Product backlog** | GitHub Issues | Staff, contractors, and eventually the broader community | High-level: bugs and feature requests |
| **Release backlog** | Jira | Staff and contractors only | Fine-grained: tasks, technical debt, research spikes |

### What goes in GitHub Issues

- Bugs reported by community members
- Bugs discovered internally that need community feedback
- New feature requests (product-level, not implementation detail)

### What stays in Jira only

- Technical tasks and subtasks
- Technical debt
- Research spikes
- Implementation-level detail

### Where GitHub Issues will live

To be decided. Options include:
- **Centralized** — all issues in this `Technology-Roadmap` repository (current pattern for features and releases)
- **Distributed** — bugs in each product's own repository, features in `Technology-Roadmap`

## Automation Approaches

The key challenge is keeping both backlogs in sync without duplicating effort. Three options are proposed, from most to least automated.

---

### Option A: Label-triggered full automation

When a specific label (e.g., `scheduled`) is applied to a GitHub Issue, a GitHub Action automatically:
1. Creates a corresponding Jira ticket
2. Posts the Jira ticket link as a comment on the GitHub Issue
3. Applies an `in-jira` label to the issue

When the Jira ticket is resolved, a Jira webhook triggers a GitHub Action that closes the GitHub Issue.

**Pros:**
- Minimal manual steps end-to-end
- Clear, visible state machine for community members

**Cons:**
- Requires strict label discipline — accidental label application creates unwanted Jira tickets
- More complex to build and maintain
- Better suited after workflow norms are established

---

### Option B: Manual Jira creation, automated sync back *(recommended starting point)*

Staff manually create the Jira ticket when a GitHub Issue is picked up for a release, and paste the Jira link into the GitHub Issue. From there, automation takes over:

- A GitHub Action monitors a Jira webhook
- When the Jira ticket resolves, the GitHub Issue is automatically closed (optionally with a closing comment)

**Pros:**
- PM retains full control over what enters the Jira release backlog
- Automation is simpler and one-directional
- No risk of accidental ticket creation
- Easier to establish good norms before opening issues to the broader community

**Cons:**
- One manual step per issue: creating the Jira ticket and pasting the link

---

### Option C: Manual everything, Jira native integration only

Rely on Jira's native GitHub integration for linking PRs and commits. Staff manually update GitHub Issue labels to communicate status and manually close issues when work ships.

**Pros:**
- Nothing to build or break

**Cons:**
- High manual toil
- High risk of GitHub Issues going stale
- Does not scale as community grows

---

## Recommendation

Start with **Option B**. It delivers most of the automation value with a fraction of the complexity. Once label taxonomy and workflow norms are established — and especially before opening issue creation to the broader community — the team can evolve toward Option A's label-triggered Jira ticket creation.

## Open Questions

- Where will GitHub Issues live? (centralized vs. distributed by product)
- What labels are needed to represent issue lifecycle states?
- Should GitHub Issues eventually be open to community members to file directly?
- What comment or notification should be posted when a GitHub Issue is auto-closed?
