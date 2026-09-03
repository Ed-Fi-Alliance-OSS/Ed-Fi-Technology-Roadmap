# PRD: Community Input and Product Management System

> **Status:** Draft for review \
> **Date:** 2026-09-03 \
> **Owner:** Ed-Fi Product Management \
> **Audience:** layered — Ed-Fi leadership (direction and investment), MSDF IT (platform dependencies
> and asks), Ed-Fi technical staff (implementable requirements)

> [!WARNING]
> **Note on preparation:** this document was drafted with AI assistance from a
> staff brain dump (Stephen Fuqua) and the design documents in the Roadmap repository.
> Requirements, options, and open questions were shaped by author decisions
> recorded in Section 8. It has not been fact-checked against licensing quotes,
> case volumes, or platform capability documentation; readers should treat
> Section 6 cost and capability statements as unverified until the open
> questions are closed.

## 1. Product overview

The Ed-Fi Alliance maintains an open standard for K-12 operational data exchange and the open
source Ed-Fi Technology Suite that operationalizes it. Community members — platform hosts, system
integrators, and edtech vendors — report defects, request capabilities, and need to know what is
coming and when. Ed-Fi staff and contractors turn that input into released software.

Between those two facts sits a chain of systems. Salesforce holds CRM records and support cases and
presents the Community Hub. Jira Cloud holds the engineering backlog across multiple development teams.
GitHub Issues, adopted in early 2026, holds the public product backlog. Slack carries a large volume
of informal community conversation. Each was adopted for a defensible reason. Together they form a
pipeline whose handoffs may be more than a small staff can reliably service and more front doors than a
community member can reasonably be expected to learn.

This document defines what a coherent system must do, expressed as system characteristics rather
than as a tool selection. Section 6 sketches candidate solutions against those requirements.

### Problem statement

Three failure modes recur today:

1. **Input arrives through channels with unequal guarantees.** A Community Hub case has an owner
   and a queue. A Slack message, a governance forum comment, and a hallway conversation at an event
   do not. Whether an idea survives depends on which door it came through and whether a staff member
   happened to transcribe it.
2. **The community's window into what happens next is partial.** Roadmap viewing is anonymous and
   public, which is right. But participation — commenting, upvoting, following — requires a GitHub
   account that some members do not have, and members cannot file anything into the public backlog
   themselves. The Jira engineering backlog, where the work actually happens, is closed.
3. **Nothing synchronizes except item creation.** The summer 2026 automations copy items between
   systems at a point in time. Changes to the core problem definition do not cascade. Comments never
   cross. Closing the loop back to the person who first raised the problem is entirely manual, and
   therefore inconsistent.

The cost of these failure modes is asymmetric. The addressable market is small — state education
agencies, regional service agencies, and a defined set of vendors — so each member relationship
carries disproportionate weight, and a dropped report is disproportionately expensive.

### 1.1 Strategic alignment

- **Community trust is the program's operating capital.** An open standard governed by an alliance
  is credible to the degree its members can see how their input is handled. A backlog they can
  neither see into nor contribute to undercuts the governance story the Alliance tells.
- **Open source norms set the expectation.** Technically inclined members arrive expecting a public
  issue tracker and are surprised to find they cannot file one.
- **Small teams need leverage.** Three development teams plus a small Customer Success group of
  contract engineers and analysts cannot absorb per-item manual re-entry across four systems as
  community participation grows. The point of fixing the seams is to make growth in participation
  survivable rather than threatening.
- **Excessive tooling in the product management path is the stated primary driver** for this work.
  Lead capture is acknowledged as an adjacent concern but is explicitly out of scope (Section 7).
- **Release objective:** converge stakeholders on one target state, and get a phase-1 increment into
  production that holds regardless of which end state is eventually chosen.

### 1.2 Target users and personas

**Community member — roadmap watcher.** Wants to know whether a capability is planned and roughly
when, in order to plan their own procurement or development. Never intends to comment. Estimated at
ten or more times the population of members who do want to comment. Success looks like: got the
answer without creating an account and without asking a human.

**Community member — platform host / system administrator.** Runs the Ed-Fi Technology Suite for an
SEA, ESA, or LEA. Files defect reports containing operational detail, sometimes including deployment
specifics that must not become public. Moderately technical; holds a Salesforce Community account,
often a Slack account, rarely a GitHub account. Success looks like: reported once, knew it was
received, learned when it was fixed.

**Community member — vendor / integration developer.** Builds SIS, assessment, or special education
integrations against the Ed-Fi API. The most technically inclined persona and the most likely to
already hold a GitHub account. Wants specification-level precision and advance warning of breaking
changes. Success looks like: could see and influence the shape of a change before it shipped.

**Ed-Fi Customer Success.** A Technical Program Manager overseeing a small group of contract
engineers and analysts. Works a daily Kanban board. Triages, resolves what they can, escalates what
they cannot. Success looks like: one queue, no case answered twice, no case silently orphaned when
escalated.

**Ed-Fi Product Manager, program managers, and team leads.** Own the product backlog and the public
roadmap. Decide what is proposed, reviewed, accepted, rejected, and scheduled. Success looks like:
one place to curate the community-facing narrative, with the engineering reality attached rather
than retyped.

**Ed-Fi development contractors.** Work Jira sprints and Kanban boards day to day. Are not expected
to work in the product backlog. Success looks like: no change to their daily tooling.

**MSDF IT.** Owns and administers Salesforce and Jira Cloud as enterprise services, and owns the
Entra directory that fronts them. Not a user of the workflow, but a gatekeeper on identity,
licensing, and configuration. Success looks like: no unbounded per-person onboarding burden, and no
request to run a second identity provider for staff.

### 1.3 Jobs to be done

- **When** I hit a defect in production, **I want** to report it once through whichever channel I am
  already in, **so that** I do not have to learn a new system while I am mid-incident.
- **When** I have reported something, **I want** to be told what happened to it, **so that** I do not
  have to re-ask, and so that I can tell my own stakeholders.
- **When** I am evaluating whether to adopt or upgrade, **I want** to see what is planned and roughly
  when, **so that** I can sequence my own work — without creating an account to do it.
- **When** I see someone else's request that matters to me too, **I want** to say so in a way that
  counts, **so that** prioritization reflects actual demand rather than who complained loudest.
- **When** I am on Customer Success and cannot resolve a case, **I want** to escalate it without
  retyping it, **so that** the reporter's words survive the handoff intact.
- **When** I am the Product Manager, **I want** one curated public backlog whose items are linked to
  real engineering work, **so that** the roadmap stays honest without me maintaining it by hand.
- **When** engineering finishes the work, **I want** the person who reported it to find out,
  **so that** the loop closes without anyone remembering to close it.
- **When** a member asks the AI assistant whether something is on the roadmap, **I want** it to
  answer correctly, **so that** the roadmap does the work instead of a staff member.

### 1.4 Success measures

Three outcome families define success. Baselines are not currently instrumented; establishing them
is itself a phase-1 requirement (FR-MEAS-5).

**Community participation.** Distinct community members who comment, upvote, or file an item in a
quarter, trending up. Ratio of community-originated to staff-originated published backlog items,
trending up from a near-zero baseline — today the community cannot file at all.

**Roadmap transparency and findability.** Share of "is this planned, and when?" questions answerable
without human involvement, including through the AI assistant. Time between a roadmap decision and
that decision being visible to the public and retrievable by the assistant.

**Nothing falls through the cracks.** Time to first substantive response on community-originated
input, by channel. Count of items past a defined age with no owner or no response. Count of
community comments on public items with no staff reply after a defined interval. Count of escalated
cases whose reporter was never told the outcome.

Staff effort per item is a design driver — it is the reason this work is being done — but is not
being adopted as a headline success measure.

> [!INFO]
> Stephen: I like these... will we be able to prioritize instrumenting for baseline and growth
> measurement?

## 2. Current state and system context

This section records observed behavior. It is not a statement of intended requirements; those are in
Section 3.

### 2.1 Systems inventory

| System | Owner | Role today | Community access |
|---|---|---|---|
| Salesforce (CRM) | MSDF IT | Organization and contact records, Chatter | Members hold accounts |
| Community Hub (`community.ed-fi.org`) | Ed-Fi Community team, on Salesforce | Case submission, deflection, member content | Authenticated members |
| Jira Cloud — `EDFI` space | Ed-Fi staff, MSDF IT admin | Customer Success daily Kanban | None |
| Jira Cloud — team spaces | Ed-Fi staff, MSDF IT admin | Engineering backlog, four teams | None; Data Standard space is public read-only |
| GitHub Issues — `Ed-Fi-Technology-Roadmap` | Ed-Fi technical staff | Public product backlog, releases | Read and comment anonymously/with GitHub account; cannot create |
| GitHub Projects (boards 1 and 2) | Ed-Fi technical staff | Public roadmap views by product and by quarter | Anonymous read |
| Slack | Ed-Fi technical staff | Informal community conversation | Separate accounts, unlinked to Salesforce |
| `docs.ed-fi.org` | Ed-Fi | Technical documentation; the bulk of public search traffic | Anonymous |
| Fiona (Perplexity-based assistant) | Ed-Fi | Answers member questions; sources restricted by domain | Public |

### 2.2 What already works

Four automations were put in place in summer 2026 and should be either preserved / improved upon, or
discarded if a new solution obviates one or more component the _status quo_ system:

1. Salesforce cases clone automatically into the Jira `EDFI` space, which Customer Success works as
   their Kanban board. Member-facing communication continues through Salesforce.
2. Escalating an `EDFI` item to a development team's space is a single Jira clone action.
3. A custom Atlassian Rovo agent copies a Jira work item out to the `Ed-Fi-Technology-Roadmap`
   repository as a GitHub Issue, writing bidirectional links in both places.
4. A GitHub Actions workflow watches for a `jira-<space-key>` label on an Issue and copies that
   Issue into the indicated Jira space, recording the Jira link in a custom field. The workflow gates
   on GitHub team membership, so only authorized staff can trigger promotion.

The public GitHub Project boards satisfy anonymous roadmap viewing today, which is the single
hardest requirement that prior alternatives failed.

### 2.3 Where the seams leak

- **Creation-time only.** All four automations copy at a moment in time. Later edits to the problem
  definition do not cascade; a reader of the derived item may be reading a stale description with no
  indication that it is stale.
- **Comments never cross.** A community member commenting on a GitHub Issue is not visible to the
  team working the Jira ticket, and a Salesforce case conversation is not visible to anyone outside
  Salesforce. This is the most direct cause of the "falls through the cracks" failure mode.
- **The community cannot originate anything.** Issue creation is staff-only and blank issues are
  disabled; the templates route the public back to the Community Hub. Every community idea reaches
  the public backlog only by staff transcription.
- **Participation requires the least-held account.** Commenting and reacting on GitHub requires a
  GitHub account, which the vendor persona usually has and the platform-host persona usually does
  not.
- **The roadmap is invisible to the assistant.** Fiona restricts sources by domain to `www.ed-fi.org`
  and `docs.ed-fi.org`. Allowing `github.com` wholesale is too broad a permission, so the public
  roadmap — though genuinely public — cannot be reached by the tool members are increasingly pointed
  at. Blog posts about roadmap updates provide sporadic exposure without filling the gap.
- **Slack capture is built but undeployed.** A Slack-to-GitHub-Issue integration exists and is
  withheld over the risk that members will paste production deployment detail — server names, IP
  addresses — into a public issue. Slack-to-Salesforce has not been approved or keyed by MSDF IT, and
  Slack identities are entirely separate from Salesforce identities.
- **Fragility is unmonitored.** The number of hops creates many places to fail quietly; there is no
  described mechanism that alerts an owner when a hop fails.

## 3. Functional requirements

Requirements are tool-neutral. `SHALL` is mandatory, `SHOULD` is intended but negotiable, `MAY` is
optional.

### 3.1 Intake (FR-IN)

- **FR-IN-1** — The system SHALL accept community-originated input through, at minimum: the
  Community Hub, the community Slack workspace, and direct staff entry on behalf of a member
  (covering email, meetings, governance forums such as the TAG and SIGs, and events).
- **FR-IN-2** — A community member SHALL be able to submit a problem report or improvement idea
  without obtaining any account beyond the one they already hold for the channel they are using.
- **FR-IN-3** — Every submitted item SHALL land in exactly one queue that has a named owning team and
  a defined response expectation. No channel may be a write-only destination.
- **FR-IN-4** — Staff SHALL be able to convert a Slack conversation into a tracked item without
  retyping it. Community members SHOULD be able to initiate that conversion themselves. Items
  created this way SHALL pass through the same publication gate as any other item (FR-SAFE-1).
- **FR-IN-5** — The system SHALL support linking or merging a new submission into an existing tracked
  item rather than creating a duplicate, and SHOULD surface likely duplicates at submission time.
- **FR-IN-6** — Staff SHOULD be able to record the originating channel and the requesting
  organization on an item, for later analysis of demand by member.
- **FR-IN-7** — Submitting a report SHALL produce an immediate acknowledgement to the submitter that
  includes a reference they can use to follow up.

### 3.2 Public visibility (FR-PUB)

- **FR-PUB-1** — The roadmap and the published product backlog SHALL be viewable with no account and
  no sign-in.
- **FR-PUB-2** — Each published item SHALL display a lifecycle status drawn from a small, publicly
  documented set: Proposed, Reviewing, Accepted, Rejected, Done.
- **FR-PUB-3** — Accepted items SHALL show a target release or quarter when one is known, and the
  presentation SHALL make clear that these are plans rather than commitments.
- **FR-PUB-4** — Both defects and feature requests SHALL be publishable. Public visibility SHALL NOT
  be restricted to features.
- **FR-PUB-5** — The roadmap SHALL be viewable along at least two axes: by product or portfolio, and
  by schedule.
- **FR-PUB-6** — Rejected items SHALL remain publicly visible with a recorded rationale rather than
  disappearing.
- **FR-PUB-7** — Engineering-internal work items — subtasks, technical debt, research spikes,
  documentation tickets — SHALL NOT be published. Public visibility is a curated act, not a mirror.
- **FR-PUB-8** — Publication SHALL be reversible: staff SHALL be able to unpublish or withdraw an
  item, with the action recorded.

### 3.3 Participation (FR-PART)

- **FR-PART-1** — An authenticated community member SHALL be able to comment on any published item.
- **FR-PART-2** — A member SHALL be able to register support for an item in a way that can be
  counted and used in prioritization.
- **FR-PART-3** — A member SHOULD be able to follow an item and be notified when its status changes.
- **FR-PART-4** — The member who originated an item SHALL be notified when the resulting published
  item is Accepted, Rejected, or Done, without a staff member having to remember to tell them.
- **FR-PART-5** — Participation SHOULD be achievable with a credential the member already holds. The
  acceptable credential is an unresolved decision; see OQ-1.
- **FR-PART-6** — Public discussion SHALL be moderatable: staff SHALL be able to hide or remove
  content and, where the platform allows, restrict a participant, in support of the Ed-Fi Contributor
  Code of Conduct.
- **FR-PART-7** — Read access SHALL never be gated behind the participation credential. The
  read-only majority must not be made to pay the participation tax.

### 3.4 Lifecycle and system of record (FR-LIFE)

- **FR-LIFE-1** — Exactly one system SHALL be the system of record for the community-facing
  definition of a problem or idea. Every other representation of that item is derived.
- **FR-LIFE-2** — Exactly one system SHALL be the system of record for engineering execution. Jira
  Cloud fills this role and is treated as fixed (Section 8, D-7).
- **FR-LIFE-3** — Promotion from the product backlog to the engineering backlog SHALL be a
  deliberate, permissioned staff action. It SHALL NOT happen automatically on submission.
- **FR-LIFE-4** — A bidirectional link SHALL be stored in both the product-backlog and
  engineering-backlog representations, and SHALL be traversable by a human in one click from either
  side.
- **FR-LIFE-5** — Completion of the engineering work SHALL propagate to the published item's status
  without manual re-entry, including the release version when one is known.
- **FR-LIFE-6** — Rejection SHALL be recordable with a rationale, and SHALL propagate to the
  originator and to the public item.
- **FR-LIFE-7** — The model SHALL support fan-out and fan-in: one published item may map to several
  engineering items, and several reports may be merged into one published item.
- **FR-LIFE-8** — A staff member SHALL be able to carry an item from intake through publication to
  the engineering backlog while working in no more than two systems.
- **FR-LIFE-9** — An item SHALL carry a durable identifier that survives promotion, merging, and any
  future platform migration.

### 3.5 Synchronization (FR-SYNC)

- **FR-SYNC-1** — When the core definition of an item changes in the system of record, derived
  representations SHALL be updated, or SHALL visibly indicate that they may be stale and link to the
  authoritative version.
- **FR-SYNC-2** — A community comment on a published item SHALL reach the staff or team responsible
  for it without requiring that person to monitor the public system.
- **FR-SYNC-3** — A staff response SHOULD be publishable to the community-visible item without
  copy-and-paste re-entry.
- **FR-SYNC-4** — Synchronization failures SHALL be detected and surfaced to a named owner.
  Silent failure is not acceptable.
- **FR-SYNC-5** — Synchronization SHALL be loop-safe: an automated update SHALL NOT trigger a
  cascade of further automated updates.
- **FR-SYNC-6** — Status propagation SHALL be idempotent, so that a repeated or replayed event does
  not duplicate items or comments.

### 3.6 Sensitive content control (FR-SAFE)

- **FR-SAFE-1** — No community-submitted content SHALL become publicly visible except through an
  explicit staff action. There is no path from private submission to public visibility that a
  non-staff actor can trigger unaided.
- **FR-SAFE-2** — Staff SHALL be able to edit, redact, or rewrite an item before publishing it.
- **FR-SAFE-3** — The system SHOULD warn staff when content proposed for publication appears to
  contain sensitive deployment detail — host names, IP addresses, connection strings, credentials,
  tokens — or personal or student-level data.
- **FR-SAFE-4** — A reporter SHALL retain a private channel for detail that supports diagnosis but is
  never published.
- **FR-SAFE-5** — Publication actions SHALL be attributable and auditable: who published what, when,
  and from which source item.

### 3.7 Discoverability (FR-FIND)

- **FR-FIND-1** — Roadmap content SHALL be retrievable by the Ed-Fi AI assistant's knowledge base.
- **FR-FIND-2** — Because the assistant restricts sources by domain, publicly visible roadmap content
  SHALL be reachable under an `ed-fi.org` domain, either natively or by scheduled export, and SHALL
  be refreshed at least weekly and after any material roadmap change.
- **FR-FIND-3** — Every published item SHALL have a stable, shareable URL that staff can paste into
  Slack, email, governance decks, and support responses.
- **FR-FIND-4** — Published items SHOULD be indexable by public web search engines.
- **FR-FIND-5** — A member SHOULD be able to determine whether a problem is already known, planned,
  or released without contacting a staff member.

### 3.8 Identity and onboarding (FR-ID)

- **FR-ID-1** — Onboarding a community member to the participation surface SHALL NOT require a
  per-person MSDF IT help desk ticket.
- **FR-ID-2** — Onboarding an Ed-Fi contractor SHALL follow a documented, repeatable path with a
  stated turnaround expectation.
- **FR-ID-3** — Participation entitlement SHOULD derive from existing Ed-Fi Community membership
  records rather than from a separately maintained list.
- **FR-ID-4** — Read access SHALL require no account, no registration, and no cookie wall
  (restates FR-PUB-1 from the identity angle because it is the requirement most often lost).
- **FR-ID-5** — Staff and contractor access to internal systems SHALL continue to work through the
  MSDF-managed Entra directory, and SHALL NOT require MSDF to operate a second identity provider for
  staff.

### 3.9 Measurement (FR-MEAS)

- **FR-MEAS-1** — The system SHALL report, per quarter, the number of distinct community members who
  filed, commented on, or registered support for an item.
- **FR-MEAS-2** — The system SHALL report the ratio of community-originated to staff-originated
  published items.
- **FR-MEAS-3** — The system SHALL report time-to-first-substantive-response on community-originated
  input, segmented by intake channel.
- **FR-MEAS-4** — The system SHALL identify items past a configurable age with no owner or no
  response, and community comments with no staff reply past a configurable interval, and SHALL
  surface them to an owner.
- **FR-MEAS-5** — Instrumentation SHALL be in place before or at the launch of the first increment,
  so that a baseline exists against which later change can be judged.
- **FR-MEAS-6** — The system SHOULD report how often the AI assistant can answer a roadmap question
  from indexed content.

## 4. Non-functional requirements

### 4.1 Security and privacy

- **NFR-SEC-1** — Integration credentials SHALL be stored as managed secrets, scoped to the minimum
  privilege needed, and rotatable without code changes.
- **NFR-SEC-2** — No public actor SHALL be able to write to the engineering backlog, directly or by
  triggering an automation.
- **NFR-SEC-3** — Automation triggers that create or publish records SHALL verify the actor's
  authorization, as the current promotion workflow does through GitHub team membership.
- **NFR-PRIV-1** — Support submissions may contain personal details and, inadvertently, student-level
  data. The system SHALL minimize propagation of such content and SHALL NOT copy it into a public
  surface without the staff gate in FR-SAFE-1.
- **NFR-PRIV-2** — Where a member's identity appears on a public item, it SHALL be by their own
  action or with their consent; staff transcription SHALL NOT attribute a member publicly by default.

### 4.2 Reliability and observability

- **NFR-REL-1** — Every automated hop SHALL be retryable and recoverable without data loss; a failed
  hop SHALL leave the source item unchanged and re-runnable.
- **NFR-REL-2** — The system SHALL tolerate the temporary unavailability of any one integrated
  platform without losing submissions.
- **NFR-OBS-1** — Every automated hop SHALL emit a log record sufficient to reconstruct what moved
  where, and SHALL be monitorable by Ed-Fi technical staff.
- **NFR-OBS-2** — A named owner SHALL be alerted on repeated or systemic integration failure.

### 4.3 Compatibility and continuity

- **NFR-COMPAT-1** — The solution SHALL work with the MSDF-managed Entra directory as the staff
  identity provider and SHALL NOT depend on a capability that Jira Cloud does not offer, such as a
  second concurrent SSO provider.
- **NFR-CONT-1** — Existing public GitHub Issues and their comment history SHALL survive any platform
  change, either in place or by migration with links preserved.
- **NFR-CONT-2** — Existing stable URLs to roadmap items SHOULD continue to resolve, or SHALL
  redirect, after any platform change.
- **NFR-PORT-1** — Community-contributed discussion SHALL be exportable in a documented format. The
  Alliance SHALL NOT accept a solution that traps member contributions in a vendor's store.

### 4.4 Operations, cost, and delivery

- **NFR-OPS-1** — Day-to-day operation, including label and workflow changes and automation
  maintenance, SHALL be performable by Ed-Fi technical staff without an MSDF IT ticket.
- **NFR-COST-1** — Total cost of ownership SHALL be stated for any proposed solution, including
  per-seat licensing, marketplace apps, and staff maintenance effort.
- **NFR-COST-2** — Pricing models whose cost scales with the size of the community are strongly
  disfavored, because participation growth is the goal and a per-member cost makes success expensive.
  Net-new spend is permitted where justified; unbounded per-member spend is not.
- **NFR-PERF-1** — Status propagation between systems SHALL complete within minutes, not days.
- **NFR-A11Y-1** — Public roadmap and item views SHOULD meet WCAG 2.1 AA.
- **NFR-SDLC-1** — Automations SHALL be version-controlled, peer-reviewed, and documented in this
  repository, with configuration expressed as data rather than hard-coded, following the pattern
  established by the existing promotion workflow.
- **NFR-SDLC-2** — Each automation SHALL have a documented failure mode and a manual fallback
  procedure, so that a broken integration degrades to manual work rather than to lost work.

## 5. System architecture and platform dependencies

### 5.1 Component model

Roles are expressed as capabilities so that the model survives a change of tool. "Today" names the
system currently filling the role.

| Capability | Role | Today | Owner | Fixed? |
|---|---|---|---|---|
| Member identity | Authenticates community members | Salesforce Community accounts | MSDF IT | No |
| Staff identity | Authenticates staff and contractors | Entra via MSDF | MSDF IT | Yes |
| Case intake | Receives and tracks private support requests | Community Hub (Salesforce) | Ed-Fi Community team / MSDF IT | No |
| Conversational intake | Captures informal reports and ideas | Slack | Ed-Fi technical staff | No |
| Support queue | Working queue for Customer Success | Jira `EDFI` space | Ed-Fi staff | No |
| Product backlog | Curated, community-facing item set | GitHub Issues | Ed-Fi technical staff | No |
| Public roadmap | Anonymous status and schedule views | GitHub Projects 1 and 2 | Ed-Fi technical staff | No |
| Engineering backlog | Sprint and Kanban execution | Jira Cloud team spaces | Ed-Fi staff / MSDF IT | **Yes** |
| CRM | Organization and contact records | Salesforce | MSDF IT | Yes for CRM role |
| Documentation | Public technical content and search surface | `docs.ed-fi.org` | Ed-Fi | Out of scope |
| AI assistant | Answers member questions from indexed sources | Fiona (Perplexity) | Ed-Fi | No |
| Integration layer | Moves items and state between the above | GitHub Actions, Jira Automation, Rovo agent | Ed-Fi technical staff | No |

### 5.2 MSDF IT dependencies and asks

This subsection is addressed to MSDF IT. Each item is a dependency the Ed-Fi program cannot resolve
on its own.

- **Identity federation.** The central constraint on community participation is that Jira Cloud
  supports a single SSO provider, which is Entra. Jira Data Center previously allowed Salesforce as a
  second provider, and that capability is what the community lost. Any solution that puts community
  members into an Atlassian product depends on MSDF IT establishing a supportable answer to this.
- **Bounded onboarding.** The Alliance needs a path to entitle community members to a participation
  surface that does not generate one help desk ticket per person. A group-based or automated
  provisioning path derived from Community membership records would satisfy FR-ID-1 and FR-ID-3.
- **Contractor provisioning reliability.** Adding Ed-Fi contractors to Atlassian products has
  historically required rework — wrong directory group assigned, or Entra-to-Atlassian directory sync
  not completing. A documented path with a stated turnaround would satisfy FR-ID-2.
- **Licensing figures.** Evaluating the Atlassian-centric option in Section 6 requires current
  Jira Service Management and Jira Product Discovery pricing at the relevant seat counts, and
  clarity on whether any Salesforce licensing cost would actually fall if case intake moved.
- **Slack integration approval.** The Slack-to-Salesforce connection has not received an API key or
  approval. A decision either way unblocks or closes off FR-IN-4.
- **Salesforce configuration latitude.** If the Salesforce-centric option in Section 6 is pursued,
  the Alliance needs to know what anonymous public viewing and public commenting Salesforce
  Experience Cloud can support, and who would configure it.

### 5.3 Data ownership

- The community-facing definition of an item, its public status, and its public discussion belong to
  the Alliance and must be exportable (NFR-PORT-1).
- Private case content and the member relationship record belong in the CRM and stay there.
- Engineering execution detail belongs in Jira and is not published (FR-PUB-7).

## 6. Potential solutions

These are sketches for discussion, not recommendations, and they are deliberately sequenced: a
coherence increment that is worth doing under any end state, followed by three candidate end states
for the consolidation decision.

### 6.1 Phase 1 — close the seams in place

Keep the current topology. Fix the specific leaks identified in Section 2.3. Nothing here is wasted
under any of the end states in 6.2, because each item is either an integration behavior that must
exist regardless or a piece of instrumentation needed to make the end-state decision on evidence.

- **Comment relay** (FR-SYNC-2, FR-SYNC-3). Route community comments on published items to the
  responsible team, and allow a staff reply to be posted back. Even a one-way notification into the
  team's working system would remove the largest current failure mode.
- **Staleness indication** (FR-SYNC-1). Where full definition sync is impractical, mark derived items
  with the authoritative source and a last-synced timestamp.
- **Integration monitoring** (FR-SYNC-4, NFR-OBS-2). Alert a named owner on hop failure. Cheap, and
  it converts an unknown fragility into a known one.
- **Roadmap export to an `ed-fi.org` domain** (FR-FIND-1, FR-FIND-2). Publish a generated roadmap
  summary under `www.ed-fi.org` or `docs.ed-fi.org` on a schedule, so the assistant can index it
  without opening `github.com` as a source. This is a standalone gap that needs filling even if
  nothing else changes.
- **Slack capture with a redaction gate** (FR-IN-4, FR-SAFE-1 through FR-SAFE-3). Deploy the built
  integration behind a staff review step, so the oversharing risk that has kept it shelved is
  handled by the gate rather than by non-deployment.
- **Instrumentation** (FR-MEAS-5). Establish baselines now. Without them, no later claim of
  improvement is defensible.

Phase 1 does not resolve the identity question, does not enable community self-service filing, and
does not reduce the number of systems. It makes the current system honest and measurable.

### 6.2 Candidate end states

**Option A — Salesforce-centric.** The Community Hub becomes both the intake and the participation
surface. Members comment and upvote with the account they already have, which is the single largest
friction removal available. A read-only roadmap projection is published to an `ed-fi.org` domain to
satisfy anonymous viewing and assistant indexing.

- *Resolves:* the identity problem outright (FR-PART-5, FR-ID-1, FR-ID-3); a single member-facing
  front door; FR-IN-2 for the largest persona.
- *Risks:* it is not established that Salesforce can present the required anonymous public views
  (OQ-2); staff dislike of Salesforce is documented and would now extend to product management;
  moving the participation surface onto `community.ed-fi.org` deepens the Community team's
  one-stop-shop strategy in a way that must be discussed with them (Section 7); the vendor persona
  may resist a CRM-based surface.

**Option B — GitHub-centric.** GitHub Issues becomes the intake surface for public-safe reports as
well as the participation surface. The Community Hub retains deflection and private, sensitive cases.
This is the current direction carried to its conclusion.

- *Resolves:* anonymous viewing already works; the promotion automation already exists and is
  team-gated; cheapest path; strongest fit for the vendor persona; no new licensing.
- *Risks:* it accepts that participation requires a GitHub account most platform-host members do not
  have, which is precisely the tradeoff in OQ-1; the domain problem for the assistant persists and
  must be solved by export regardless; opening issue creation to the public introduces a moderation
  and duplicate-management load the team has not yet carried.

**Option C — Atlassian-centric.** Jira Service Management for intake and Jira Product Discovery for
the product backlog, with a public roadmap published through a marketplace app such as Released Hub.
One vendor, one fewer hop, and native linkage to the engineering backlog.

- *Resolves:* the product-to-engineering seam nearly disappears; a single support-to-delivery data
  model.
- *Risks:* every reason JPD was rejected in October 2024 still needs to be retested — no anonymous
  roadmap viewing natively, per-person onboarding friction, and per-seat cost at community scale,
  which collides directly with NFR-COST-2. JPD is oriented to features rather than defects, though
  that is likely workable. This option depends most heavily on MSDF IT (Section 5.2) and should not
  be considered without the licensing figures and an answer on identity federation.

### 6.3 Suggested decision sequence

1. Execute Phase 1, which is independent of the end-state choice.
2. Close OQ-1 (participation credential) and OQ-2 (Salesforce anonymous viewing capability). These
   two answers eliminate at least one option.
3. Obtain the licensing figures needed to keep or discard Option C.
4. Use the Phase 1 baselines to judge whether the residual friction justifies a platform move at all
   — the honest fourth outcome is that a well-instrumented Phase 1 proves sufficient.

## 7. Out of scope and known limitations

### Out of scope

- **Lead generation and prospective adopters.** The Intercom Fin evaluation for `www.ed-fi.org`
  targets discovery and lead capture, with follow-up owned by the Solutions Team rather than Customer
  Success. This PRD serves *active* community members. The two efforts share a domain and will need
  to be reconciled at the interface, but neither governs the other.
- **Website content and search strategy.** Whether `community.ed-fi.org` or `docs.ed-fi.org` is the
  community's front door, and how content and search traffic are divided between them, belongs to the
  Community team. **However:** any proposal that moves the participation surface away from Salesforce
  has direct implications for the Community team's stated goal of making the Community Hub the
  one-stop destination for members. That conversation is a prerequisite to Options A and B, not an
  afterthought.
- **Replacing Jira as the engineering backlog.** Fixed by decision D-7.
- **Salesforce as the CRM.** Organization and contact records, membership registration, and Chatter
  stay where they are. Only the case-intake and participation roles are in question.
- **Engineering work item granularity.** How teams structure subtasks, spikes, and technical debt in
  Jira is unchanged.
- **Migrating `docs.ed-fi.org`** or changing the documentation platform.

### Known limitations and assumptions

- **Assumption:** the read-to-participate ratio is at least 10:1, per staff estimate. It is not
  measured. If it is materially lower, the weight on anonymous viewing versus participation friction
  shifts, and Option B looks worse.
- **Assumption:** most active members hold a Salesforce Community account and many hold a Slack
  account; GitHub accounts are common only among the vendor persona. This is a staff estimate and is
  worth verifying against membership data before OQ-1 is decided.
- **Limitation:** no baseline metrics exist for any of the Section 1.4 success measures. Until
  FR-MEAS-5 is satisfied, improvement claims are not verifiable.
- **Limitation:** GitHub Issues adoption is recent and community response is still emerging. Judging
  Option B on current participation volume would be premature.
- **Risk:** every option preserves at least two member-facing systems for some period. The
  transitional state may be more confusing than the current one and needs an explicit communication
  plan.
- **Risk:** enabling community self-service filing (FR-IN-2, FR-PUB-*) increases triage, duplicate,
  and moderation load. Capacity for this has not been assessed.

## 8. Open questions and decision log

### Open questions

- **OQ-1 — What credential should public participation require?** The pivotal question. Requiring
  the Salesforce Community account matches the credential most active members already hold, but
  effectively rules out GitHub Issues as the long-term participation surface and revives the
  federation problem that defeated JPD. Accepting a GitHub account keeps the cheapest path and
  matches the vendor persona, but asks the platform-host persona to create an account they otherwise
  have no use for. Deliberately left open; it should be decided before any platform commitment.
- **OQ-2 — Can Salesforce present a genuinely anonymous, unauthenticated public roadmap view?** If
  not, Option A requires a published projection to an `ed-fi.org` domain regardless, which changes
  its cost.
- **OQ-3 — What do JSM and JPD actually cost at the relevant seat counts,** and would any Salesforce
  cost fall if case intake moved? The June 2026 discussion rejected JSM partly on a cost assumption
  that was never quantified.
- **OQ-4 — What is the concrete mechanism for getting roadmap content into the assistant's index?**
  A generated page, a feed, a scheduled export? Owner and cadence are undefined.
- **OQ-5 — Who owns the redaction gate,** and what response expectation applies to it? A publication
  gate with no service level becomes a bottleneck that quietly reintroduces the current failure mode.
- **OQ-6 — Should community-created items enter a moderation queue** before appearing publicly, or
  appear immediately and be curated after the fact? This interacts with FR-SAFE-1 and with triage
  capacity.
- **OQ-7 — Will MSDF IT approve and key the Slack integration,** and does the separation between
  Slack identities and Salesforce identities need solving or merely documenting?
- **OQ-8 — What is the actual distribution of accounts across the membership?** Directly informs OQ-1
  and is answerable from existing membership data.
- **OQ-9 — What response expectation applies to each intake channel** (FR-IN-3)? The requirement
  states that one must exist; the values are a staffing decision.
- **OQ-10 — What is the impact on RFCs?** The Data Standard team has begun posting RFCs for as
  Discussions in GitHub, allowing community members to read and respond to these draft documents
  in a public forum. Would moving away from GitHub issues have any impact on adoption of this
  process for comments? Would moving away offer different approaches? 

### Decision log

- **D-1 — October 2024: Jira Product Discovery rejected** for the product backlog. Reasons: no
  anonymous roadmap viewing; perceived onboarding friction for community members, including the
  expectation of a help desk ticket per person; unreliable contractor provisioning; and an orientation
  toward features rather than defects. Subject to re-test as Option C; not treated as permanently
  settled.
- **D-2 — Early 2026: GitHub Issues adopted as the product backlog,** formalizing the direction chosen
  in 2024. Public boards satisfy anonymous roadmap viewing.
- **D-3 — March 2026: dual-backlog model adopted** — GitHub Issues for the product backlog, Jira for
  the release backlog — with label-triggered promotion (`docs/dual-backlog-options.md`,
  `docs/superpowers/specs/2026-03-26-dual-backlog-design.md`).
- **D-4 — June 2026: Jira Service Management rejected** for case intake. Reasons: the broken
  Salesforce-to-Jira automation was repaired and improved by MSDF IT, removing the immediate pain;
  and Salesforce licensing cost would not fall while JSM licensing might be added. Cost was not
  quantified. Subject to re-test as Option C.
- **D-5 — Summer 2026: four integration automations deployed** (Section 2.2).
- **D-6 — 2026: lead generation scoped out** of this effort; Intercom Fin work proceeds separately
  under the Solutions Team.
- **D-7 — 2026-08-31: Jira Cloud remains the engineering backlog.** Fixed constraint; solutions that
  move sprint execution elsewhere are not viable.
- **D-8 — 2026-08-31: MSDF IT ownership of Salesforce and Jira administration is a fixed constraint.**
  Requirements depending on identity, SSO, or admin configuration must account for the dependency and
  its turnaround.
- **D-9 — 2026-08-31: Salesforce is not fixed as the case-intake surface,** and net-new licensing
  spend is not automatically disqualifying. Both were previously assumed constraints; both are open.
- **D-10 — 2026-08-31: the requirements are written tool-neutrally against the desired end state,**
  with a sequenced coherence-then-consolidation path rather than a single recommendation.

## 9. Glossary

- **Community Hub** — `community.ed-fi.org`, the Salesforce-hosted member site providing case
  submission, deflection, and member content.
- **Customer Success team** — the Technical Program Manager plus contract engineers and analysts who
  work the support caseload.
- **Deflection** — presenting a member with existing content that may answer their question before
  they file a case.
- **`EDFI` space** — the Jira project used by Customer Success as their daily Kanban board; the
  landing point for cases cloned from Salesforce.
- **Ed-Fi API** — the concrete implementation of the Ed-Fi Data Standard; the core of the Technology
  Suite.
- **Engineering backlog** — fine-grained execution work in Jira: tasks, subtasks, technical debt,
  spikes. Not published.
- **ESA** — Education Service Agency, a regional education organization.
- **Fiona** — the Ed-Fi AI assistant, built on Perplexity, whose sources are restricted by domain.
- **JPD** — Jira Product Discovery, Atlassian's product-management tool.
- **JSM** — Jira Service Management, Atlassian's service desk tool.
- **LEA** — Local Education Agency, typically a school district.
- **MSDF** — the Michael & Susan Dell Foundation, of which the Ed-Fi Alliance is a program.
- **Product backlog** — the curated, community-facing set of ideas and defects, currently GitHub
  Issues. Distinct from the engineering backlog.
- **Published item** — an item that has passed the staff gate and is publicly visible.
- **Rovo agent** — the custom Atlassian agent that copies a Jira work item out to GitHub.
- **SEA** — State Education Agency.
- **SIG / TAG** — Special Interest Group and Technical Advisory Group; Ed-Fi governance forums that
  generate product input.
- **System of record** — the single authoritative location for a given kind of information; all other
  copies are derived.
