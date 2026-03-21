# Changelog

All notable changes to Autensa (formerly Mission Control) will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.0.1] - 2026-03-21

### Fixed
- **Dispatch Hang on Stage Transitions** — All server-side dispatch fetch calls now have a 30-second `AbortSignal.timeout`. Previously, if the OpenClaw gateway was slow or unresponsive during testing/review/verification transitions, the PATCH request would hang indefinitely — potentially crashing the server. The timeout applies to all 10 dispatch call sites across the codebase. ([#84](https://github.com/crshdn/mission-control/issues/84))
- **Stale OpenClaw Connection After Timeout** — Added `forceReconnect()` to the OpenClaw WebSocket client. When a dispatch fails (timeout or connection error), the client now tears down the stale WebSocket and clears all pending requests, so the next dispatch attempt gets a fresh connection instead of reusing a dead one.
- **Stale Markdown in Agent Modal** — The Agent Modal now fetches fresh `soul_md`, `user_md`, and `agents_md` from the API each time it opens, instead of displaying cached data from the Zustand store that was loaded once at page load. ([#75](https://github.com/crshdn/mission-control/issues/75))

### Improved
- **Pre-Migration Database Backups** — Contributed by [@cgluttrell](https://github.com/cgluttrell). Before running any pending migrations, a timestamped backup is created using SQLite's `VACUUM INTO` (safe for WAL-mode databases). Keeps the last 5 backups. If backup fails, migrations abort entirely. ([PR #79](https://github.com/crshdn/mission-control/pull/79))
- **Migration 013 Data Guard** — Contributed by [@cgluttrell](https://github.com/cgluttrell). The destructive "fresh start" migration now checks for existing tasks and locally-configured agents before running. Databases with real data are preserved instead of silently wiped on upgrade. ([PR #79](https://github.com/crshdn/mission-control/pull/79))
- **Static Device Identity Path** — Contributed by [@org4lap](https://github.com/org4lap). Removed dynamic `filePath` parameter from `loadOrCreateDeviceIdentity()`, binding file operations to the module-level constant. Resolves TP1004 static analysis warning. ([PR #82](https://github.com/crshdn/mission-control/pull/82))

---

## [2.0.0] - 2026-03-20

### 🚀 Product Autopilot — The World's First Autonomous Product Engine

Autensa v2 transforms from a task orchestration dashboard into the world's first autonomous product improvement engine. Point it at any product and it runs a continuous research → ideation → build loop.

### Added

#### Product Autopilot Pipeline
- **Autonomous Research Engine** — AI agents analyze your codebase, scan your live site, and research your market automatically. Discovers competitors, user intent, conversion patterns, SEO gaps, and technical opportunities. Configurable schedules (daily, weekly, custom cron, or on-demand). Research results stored with full source attribution.
- **AI-Powered Ideation** — Research feeds into ideation agents that generate concrete, scored feature ideas. Each idea includes impact score (1–10), feasibility score (1–10), size estimate (S/M/L/XL), technical approach, and a direct link to the research that inspired it.
- **Swipe Interface (IdeaSwipe)** — Tinder-style card interface for reviewing ideas. Four actions: Pass (rejected, preference model learns), Maybe (saved to pool, resurfaces in 1 week), Yes (task created, build agent starts), Now! (urgent dispatch, priority queue). Full swipe history tracking.
- **Product Program** — Inspired by Karpathy's [AutoResearch](https://github.com/karpathy/autoresearch) `program.md` pattern. Each product has a living document that instructs research and ideation agents on what to look for, priorities, and constraints. Evolves as swipe data accumulates.
- **Preference Learning** — Per-product preference model trained from swipe history. Category weights, complexity preferences, and tag patterns adjust automatically. Ideas get sharper with every iteration.
- **Maybe Pool** — Ideas swiped "Maybe" enter a holding pool. Auto-resurface after configurable period with fresh market context. Batch re-evaluation mode. Promote to Yes at any time.
- **Product Scheduling** — Configure research and ideation cycles per product. Cron-style schedules with enable/disable toggles. Auto-dispatch rules for approved ideas.
- **Product Management UI** — Full product CRUD with URL scanning, research report viewer, ideation cycle history, swipe statistics, and per-product cost tracking.

#### Convoy Mode — Parallel Multi-Agent Execution
- **Convoy orchestration** — Large features decomposed into subtasks with dependency-aware scheduling. 3–5 agents work simultaneously on one feature.
- **Dependency graph visualization** — Visual DAG showing subtask dependencies, completion status, and agent assignments.
- **Inter-agent mailbox** — Convoy agents can send messages to each other during execution. Messages queued and delivered at checkpoints.
- **Convoy progress tracking** — Real-time progress aggregation across all subtasks. Parent task status derived from subtask completion.

#### Agent Health Monitoring
- **Stall detection** — Agents are monitored at configurable intervals (default 6 min). Stall threshold (5 min no activity), stuck threshold (15 min), and auto-nudge after 3 consecutive stall events.
- **Auto-nudge** — Stalled agents are automatically killed and restarted from their last checkpoint. If nudge fails, task is flagged for manual intervention.
- **Health indicators** — Real-time health badges on agent cards (healthy/stalled/stuck/offline).
- **Health API** — Per-agent and aggregate health endpoints for monitoring.

#### Operator Chat
- **Queued notes** — Add context to running tasks. Notes are delivered to the agent at its next checkpoint.
- **Direct messages** — Real-time delivery to the agent's active session. Agent incorporates changes immediately.
- **Chat history** — Full per-task chat log preserved. Every message, note, and agent response.
- **Chat listener** — Server-side relay that bridges operator messages to agent sessions via OpenClaw.

#### Cost Tracking & Budget Caps
- **Per-task cost events** — Every API call tracked with model, tokens, and cost.
- **Per-product cost aggregation** — See total spend across all tasks for a product.
- **Daily and monthly budget caps** — Set limits that auto-pause dispatch when exceeded.
- **Cost breakdown API** — Detailed reports by agent, model, task, product, and time period.
- **Cost dashboard UI** — Visual breakdown with charts and spending trends.

#### Checkpoint & Crash Recovery
- **Checkpoint save** — Agent progress saved at configurable intervals. Includes task state, files modified, and agent context.
- **Checkpoint restore** — Resume from any saved checkpoint. Manual restore via API or automatic on crash.
- **Checkpoint history** — View all checkpoints per task with timestamps and metadata.

#### Knowledge Base
- **Learner agent** — Captures lessons from every build cycle: what worked, what failed, patterns observed.
- **Knowledge injection** — Learner entries are injected into future dispatch messages so agents don't repeat mistakes.
- **Per-workspace knowledge** — Knowledge scoped to workspace for relevance.

#### Workspace Isolation
- **Git worktrees** — Repo-backed projects get isolated branches via git worktree. No conflicts between concurrent agents.
- **Task sandboxes** — Local/no-repo projects get dedicated directories under `.workspaces/task-{id}/`.
- **Port allocation** — Dev server ports from 4200–4299 range with unique constraint. No port conflicts between concurrent builds.
- **Serialized merge queue** — Completed tasks merge one at a time with conflict detection. Product-scoped locking for concurrent completions.
- **Workspace status UI** — ISOLATED badge on kanban cards, WorkspaceTab in task modal showing workspace path, branch, port, and merge status.

#### Automation Tiers
- **Supervised** — PRs created automatically, you review and merge manually. Default for production apps.
- **Semi-Auto** — PRs auto-merge when CI passes and review agent approves. For staging and trusted repos.
- **Full Auto** — Everything automated end-to-end. Idea to deployed feature. For side projects and MVPs.
- **Per-product configuration** — Change automation level anytime per product.

#### New UI Components
- `SwipeDeck` — Stacked card interface for idea review with swipe animations
- `IdeaCard` — Detailed idea card with scores, tags, and research links
- `ResearchReport` — Research cycle viewer with progress tracking
- `ActivityPanel` — Real-time autopilot activity stream with auto-scroll
- `BuildQueue` — Visual build queue with agent assignments
- `MaybePool` — Maybe idea management interface
- `ProductProgramEditor` — In-app editor for product programs
- `IdeasList` — Sortable/filterable ideas table
- `ConvoyTab` — Convoy subtask visualization
- `DependencyGraph` — Interactive DAG for convoy dependencies
- `HealthIndicator` — Agent health status badges
- `TaskChatTab` — Operator chat interface
- `WorkspaceTab` — Workspace isolation status and controls
- `costs/` — Cost breakdown dashboard components

#### New API Endpoints
- `GET/POST /api/products` — Product CRUD
- `GET/PATCH/DELETE /api/products/[id]` — Individual product management
- `POST /api/products/[id]/research/run` — Trigger research cycle
- `GET /api/products/[id]/research/cycles` — Research cycle history
- `POST /api/products/[id]/ideation/run` — Trigger ideation cycle
- `GET /api/products/[id]/ideation/cycles` — Ideation cycle history
- `GET /api/products/[id]/swipe/deck` — Get swipeable idea deck
- `POST /api/products/[id]/swipe` — Record swipe decision
- `GET /api/products/[id]/swipe/history` — Swipe history
- `GET /api/products/[id]/swipe/stats` — Swipe statistics
- `GET/POST /api/products/[id]/maybe` — Maybe pool management
- `POST /api/products/[id]/maybe/[ideaId]/resurface` — Force resurface
- `POST /api/products/[id]/maybe/evaluate` — Batch re-evaluation
- `GET/POST /api/products/[id]/schedules` — Product schedules
- `GET/PATCH/DELETE /api/products/[id]/schedules/[schedId]` — Individual schedule
- `GET /api/products/[id]/ideas` — All ideas for product
- `GET /api/products/[id]/ideas/pending` — Unreviewed ideas
- `GET/PATCH/DELETE /api/products/[id]/ideas/[ideaId]` — Individual idea
- `GET /api/products/[id]/costs` — Product cost summary
- `GET /api/products/[id]/activity` — Product activity feed
- `GET /api/products/[id]/workspaces` — Product workspace listing
- `POST /api/products/scan-url` — Scan URL for product metadata
- `GET/POST /api/tasks/[id]/convoy` — Convoy management
- `POST /api/tasks/[id]/convoy/dispatch` — Dispatch convoy subtasks
- `GET /api/tasks/[id]/convoy/subtasks` — List convoy subtasks
- `GET /api/tasks/[id]/convoy/progress` — Convoy progress
- `POST /api/convoy/[convoyId]/mail` — Inter-agent convoy mail
- `GET/POST /api/tasks/[id]/chat` — Operator chat
- `GET/POST /api/tasks/[id]/checkpoint` — Checkpoint management
- `POST /api/tasks/[id]/checkpoint/restore` — Restore from checkpoint
- `GET /api/tasks/[id]/checkpoints` — Checkpoint history
- `GET/POST /api/tasks/[id]/workspace` — Task workspace status
- `GET /api/agents/health` — Aggregate agent health
- `GET /api/agents/[id]/health` — Per-agent health
- `POST /api/agents/[id]/health/nudge` — Manual nudge
- `GET/POST /api/agents/[id]/mail` — Agent mailbox
- `GET/POST /api/costs` — Cost event tracking
- `POST /api/costs/event` — Record cost event
- `GET /api/costs/breakdown` — Cost breakdown report
- `GET/POST /api/costs/caps` — Budget cap management
- `GET /api/costs/caps/status` — Current cap status

#### Database — 8 New Migrations (014–021)
- Migration 014: Convoy tables (`convoys`, `convoy_subtasks`), agent health monitoring (`agent_health`), work checkpoints (`work_checkpoints`), agent mailbox (`agent_mailbox`)
- Migration 015: Task table expansion (workspace columns, dispatch lock, retry count, convoy reference, product/idea references)
- Migration 016: Product Autopilot tables (`products`, `research_cycles`, `ideas`, `swipe_history`, `preference_models`, `maybe_pool`, `product_feedback`)
- Migration 017: Cost tracking tables (`cost_events`, `cost_caps`)
- Migration 018: Product scheduling (`product_schedules`, `operations_log`), autopilot activity log (`autopilot_activity_log`)
- Migration 019–021: Workspace isolation (`workspace_ports`, `workspace_merges`), schema refinements

### Changed
- **Project identity** — "Mission Control" → "Autensa" throughout. Tagline updated to "The Autonomous Product Engine"
- **Architecture** — Added Autopilot Engine layer between dashboard and agent runtime
- **Task dispatch** — Now supports workspace isolation strategy detection before dispatch. Agents receive isolated paths, ports, branches, and workspace boundaries
- **Merge on completion** — Task completion triggers workspace merge with product-scoped serialization lock

### Technical
- 21 total database migrations (up from 13)
- 18 new database tables
- 80+ new API endpoints
- 2,831 lines of new core library code (autopilot + convoy + health + checkpoints + workspace + mailbox + chat + learner)
- 8 new UI components for autopilot features
- Shared LLM completion helper (`lib/autopilot/llm.ts`) for stateless HTTP calls to AI providers

---

## [1.5.3] - 2026-03-13

### Fixed
- **Agent Status Stale After Stage Handoff** — When a task moved between pipeline stages (builder → tester → reviewer → done), the previous agent's status remained `working` in the database permanently. Now the workflow engine resets the outgoing agent to `standby` on every stage handoff (unless the agent has other active tasks). The task PATCH endpoint also resets the assigned agent when a task moves to `done`.

---

## [1.5.2] - 2026-03-13

### Fixed
- **Dispatch Deadlock Bug** — Fixed race condition where a failed dispatch left the task stuck in `in_progress` permanently. The planning poll idempotency check now detects stale dispatches (no agent activity within 2 minutes) and retries instead of silently skipping. Previously, if the OpenClaw WebSocket dropped during dispatch, the task would never recover.
- **Dispatch Error Recovery** — The dispatch endpoint now resets the task to `assigned` with a recorded error when delivery to the agent fails, instead of returning a generic 500 and leaving the task in a broken state. This allows the UI and poll loop to detect and retry the dispatch.

---

## [1.5.1] - 2026-03-12

### Added
- **Canonical Gateway Agent Sync** — OpenClaw-installed agents are now treated as the canonical catalog and synced into Mission Control automatically (startup/scheduled + dispatch-triggered).
- **Dynamic Task Routing (Hybrid)** — Task dispatch now supports dynamic per-task routing using planner candidates plus role/fallback safeguards.
- **Governance Tests** — Added unit tests for evidence gating, done-state validation blocking, fixer auto-provisioning, and stage failure counting.

### Changed
- **Team Role Assignment UX** — Task role names are now normalized and non-freeform in Team assignment flow to prevent duplicate role keys (e.g., `Learner` vs `learner`).
- **Board Override Path** — Added explicit board-only override support (disabled by default) with audit logging hooks.

### Fixed
- **Done-State Consistency** — Prevented tasks from ending in `done` when validation/failure state indicates unresolved issues.
- **Stage Evidence Gates** — Enforced deliverable + activity requirements for stage progression; fail-back now requires a failure reason.
- **Failure Escalation** — Added escalation after repeated same-stage failures with guaranteed fixer provisioning when missing.
- **Working/Standby Agent Badges** — Agent status tags are now reconciled from active task state and update live with task updates (no manual refresh required after initial reload).
- **Duplicate Learner Role Rows** — Fixed duplicate learner role assignment caused by case mismatch.

## [1.5.0] - 2026-03-10

### Added
- **Task Image Attachments** — Upload reference images (UI mockups, screenshots, etc.) to tasks. Images are included in agent dispatch context so AI agents can see what they're building. New Images tab in Task Modal with grid view, upload, and delete. (fixes #60)

### Fixed
- **PORT env var** — Dev and start scripts now respect `PORT` env var instead of hardcoding 4000. Config fallback URL also uses `process.env.PORT`. (fixes #68)
- **Webhook auth bypass** — Webhook routes (`/api/webhooks/*`) now bypass `MC_API_TOKEN` middleware, relying on their own HMAC signature validation. Fixes broken callbacks in split-service deployments. (fixes #64)
- **Agent "Working" status** — Agents now correctly reset to standby when they have no remaining active tasks. Previously the Working tag persisted after task completion/deletion. (fixes #61)

---

## [1.4.1] - 2026-03-10

### Added
- **Kanban UX Improvements** — Improved horizontal scrollbar visibility and hit area. Added optional compact empty columns mode (off by default, toggleable via Settings → Kanban UX). (PR #66)
- **Docker CI Workflow** — GitHub Actions workflow to automatically build the Dockerfile on push. (PR #69)
- **Pipeline Documentation** — Added `docs/HOW-THE-PIPELINE-WORKS.md` explaining the full multi-agent pipeline lifecycle, stages, loop-back mechanics, and Learner knowledge injection.

### Fixed
- **Workspace Deletion** — Fixed `SQLITE_CONSTRAINT_FOREIGNKEY` error when deleting workspaces that have auto-created workflow templates or knowledge entries. Cascade deletion now properly cleans up dependent records. (PR #71, fixes #70)

---

## [1.4.0] - 2026-03-03

### Added
- **Multi-Agent Workflow Pipeline** — Full task lifecycle now supports staged orchestration: `planning → inbox → assigned → in_progress → testing → review → verification → done`.
- **Core Agent Bootstrap** — New workspaces can auto-bootstrap a 4-agent core team: Builder (🛠️), Tester (🧪), Reviewer (🔍), and Learner (📚).
- **Workflow Engine Coordination** — Added queue-aware review draining (`drainQueue()`), automatic role-based stage handoffs, and fail-loopback routing.
- **Learner Knowledge Loop** — Learner notifications on stage transitions plus knowledge injection into future dispatch messages.
- **New API Routes**
  - `POST /api/tasks/[id]/fail`
  - `GET /api/tasks/[id]/roles`
  - `POST /api/workspaces/[id]/knowledge`
  - `GET /api/workspaces/[id]/workflows`

### Changed
- **Strict template defaults** — Strict workflow is now default, with review as queue stage and verification owned by the `reviewer` role.
- **Workspace initialization** — New workspaces can clone workflow templates and bootstrap core agents automatically.
- **Project branding/docs** — Updated project branding to Autensa (formerly Mission Control) and added explicit privacy-first statement in docs.

### Fixed
- **Role mismatch** — Fixed strict template verification role (`verifier` → `reviewer`).
- **Review queue bypass** — Fixed auto-advance behavior that could skip proper review queue flow.
- **Dispatch status transition** — Fixed dispatch route using hardcoded `done`; now uses computed next workflow status.
- **Assigned-status resolution** — Fixed mapping so `assigned` resolves to builder stage dispatch correctly.
- **Task template assignment** — Fixed task creation path so default workflow template is attached automatically.
- **Learner role assignment** — Fixed missing `task_roles` learner assignment so the learner receives transition events.

### Migration
- **Migration 013: Fresh Start** — Resets runtime task/agent/event data, sets Strict as default workflow template, and bootstraps core agents for the default workspace.

---

## [1.3.0] - 2026-03-02

### Added
- **Agent Activity Dashboard** — Dedicated page for monitoring agent work with mobile card layout. (#48 — thanks @pkgaiassistant-droid!)
- **Remote Model Discovery** — Discover AI models from OpenClaw Gateway via `MODEL_DISCOVERY=true` env var. (#43 — thanks @davetha!)
- **Proxy Troubleshooting** — Added docs for users behind HTTP proxies experiencing 502 errors on agent callbacks.

### Fixed
- **Force-Dynamic API Routes** — All API routes now use `force-dynamic` to prevent stale cached responses. (#43)
- **Null Agent Assignment** — `assigned_agent_id` can now be null in task creation schema. (#38 — thanks @JamesCao2048!)
- **Dispatch Spec Forwarding** — Planning spec and agent instructions now forwarded in dispatch messages. (#51)
- **Dispatch Failure Recovery** — Tasks stuck in `pending_dispatch` auto-reset to planning status. (#52)

---

## [1.2.0] - 2026-02-19

### Added

- **Gateway Agent Discovery** — Import existing agents from your OpenClaw Gateway into Mission Control. New "Import from Gateway" button in the agent sidebar opens a discovery modal that lists all Gateway agents, shows which are already imported, and lets you bulk-import with one click. Imported agents display a `GW` badge for provenance tracking. ([#22](https://github.com/crshdn/mission-control/issues/22) — thanks [@markphelps](https://github.com/markphelps)!)
- **Docker Support** — Production-ready multi-stage Dockerfile, docker-compose.yml with persistent volumes, and `.dockerignore`. Runs as non-root, uses `dumb-init` for signal handling, includes health checks. ([#21](https://github.com/crshdn/mission-control/pull/21) — thanks [@muneale](https://github.com/muneale)!)
- **Agent Protocol Conventions** — Added `PROGRESS_UPDATE` and `BLOCKED` message formats to the Agent Protocol docs to prevent agent stalling. ([#24](https://github.com/crshdn/mission-control/pull/24) — thanks [@nice-and-precise](https://github.com/nice-and-precise)!)

### Fixed

- **Planning Flow Improvements** — Refactored polling to prevent stale state issues, fixed "Other" free-text option (case mismatch bug), made `due_date` nullable, increased planning timeout to 90s for larger models, auto-start polling on page load. ([#26](https://github.com/crshdn/mission-control/pull/26) — thanks [@JamesTsetsekas](https://github.com/JamesTsetsekas)!)
- **WebSocket RPC Deduplication Bug** — The event deduplication cache was silently dropping repeated RPC responses with the same payload hash, causing request timeouts. RPC responses now bypass dedup entirely.
- **Next.js Response Caching** — Dynamic API routes that query live state (e.g., agent discovery) now use `force-dynamic` to prevent stale cached responses.

---

## [1.1.0] - 2026-02-16

### 🔒 Security

- **API Authentication Middleware** — Bearer token authentication for all API routes. Set `MC_API_TOKEN` in `.env.local` to enable. Same-origin browser requests are automatically allowed.
- **Webhook HMAC-SHA256 Validation** — Agent completion webhooks now require a valid `X-Webhook-Signature` header. Set `WEBHOOK_SECRET` in `.env.local` to enable.
- **Path Traversal Protection** — File download endpoint now uses `realpathSync` to resolve symlinks and validate all paths are within the allowed directory.
- **Error Message Sanitization** — API error responses no longer leak internal details (stack traces, file paths) in production.
- **Security Headers** — Added `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, and `Permissions-Policy` headers via Next.js config.
- **Input Validation (Zod)** — Request payloads for tasks, agents, and workspaces are validated with Zod schemas before processing.
- **Repository Audit** — Purged sensitive files from git history, updated `.gitignore` to block database files and backups.

### Added

- **Ed25519 Device Identity** — Gateway pairing now uses Ed25519 key-based device identity for secure handshakes.
- **ARIA Hook** — Real-time agent tracking bridge between ARIA and Mission Control (`scripts/aria-mc-hook.sh`).
- **Planning Poll Endpoint** — New `POST /api/tasks/[id]/planning/poll` for long-poll planning updates.
- **Retry Dispatch** — New `POST /api/tasks/[id]/planning/retry-dispatch` to retry failed task dispatches.
- **Auto-Dispatch Module** — `src/lib/auto-dispatch.ts` for automatic task assignment after planning.
- **Planning Utilities** — `src/lib/planning-utils.ts` with shared planning logic.
- **MC Bridge Scripts** — Python and shell bridge scripts for external integrations.

### Changed

- **Node.js v25 Support** — Updated `better-sqlite3` to v12.6.2 for Node v25 compatibility.
- **Default Port** — Mission Control now defaults to port 4000 (previously 3000).
- **Improved Planning Tab** — Enhanced UI with better question rendering, progress tracking, and error handling.
- **Agent Sidebar Improvements** — Better status display, model selection, and agent management.
- **Activity Log Overhaul** — Cleaner timeline UI with better type icons and formatting.
- **Live Feed Improvements** — Better real-time event display with filtering options.

### Fixed

- **Same-origin browser requests** — Auth middleware no longer blocks the UI's own API calls.

---

## [1.0.1] - 2026-02-04

### Changed

- **Clickable Deliverables** - URL deliverables now have clickable titles and paths that open in new tabs
- Improved visual feedback on deliverable links (hover states, external link icons)

---

## [1.0.0] - 2026-02-04

### 🎉 First Official Release

This is the first stable, tested, and working release of Mission Control.

### Added

- **Task Management**
  - Create, edit, and delete tasks
  - Drag-and-drop Kanban board with 7 status columns
  - Task priority levels (low, normal, high, urgent)
  - Due date support

- **AI Planning Mode**
  - Interactive Q&A planning flow with AI
  - Multiple choice questions with "Other" option for custom answers
  - Automatic spec generation from planning answers
  - Planning session persistence (resume interrupted planning)

- **Agent System**
  - Automatic agent creation based on task requirements
  - Agent avatars with emoji support
  - Agent status tracking (standby, working, idle)
  - Custom SOUL.md personality for each agent

- **Task Dispatch**
  - Automatic dispatch after planning completes
  - Task instructions sent to agent with full context
  - Project directory creation for deliverables
  - Activity logging and deliverable tracking

- **OpenClaw Integration**
  - WebSocket connection to OpenClaw Gateway
  - Session management for planning and agent sessions
  - Chat history synchronization
  - Multi-machine support (local and remote gateways)

- **Dashboard UI**
  - Clean, dark-themed interface
  - Real-time task updates
  - Event feed showing system activity
  - Agent status panel
  - Responsive design

- **API Endpoints**
  - Full REST API for tasks, agents, and events
  - File upload endpoint for deliverables
  - OpenClaw proxy endpoints for session management
  - Activity and deliverable tracking endpoints

### Technical Details

- Built with Next.js 14 (App Router)
- SQLite database with automatic migrations
- Tailwind CSS for styling
- TypeScript throughout
- WebSocket client for OpenClaw communication

---

## [0.1.0] - 2026-02-03

### Added

- Initial project setup
- Basic task CRUD
- Kanban board prototype
- OpenClaw connection proof of concept

---

## Roadmap

- [x] Multiple workspaces
- [x] Webhook integrations
- [x] API authentication & security hardening
- [x] Product Autopilot (Research → Ideation → Swipe → Build)
- [x] Convoy Mode (parallel multi-agent execution)
- [x] Agent health monitoring & auto-nudge
- [x] Operator chat (mid-build communication)
- [x] Cost tracking & budget caps
- [x] Checkpoint & crash recovery
- [x] Workspace isolation (git worktrees + sandboxes)
- [x] Preference learning from swipe history
- [ ] Team collaboration
- [ ] Multi-tenant SaaS mode
- [ ] Agent performance metrics dashboard
- [ ] Mobile app
- [ ] Dark/light theme toggle
- [ ] Plugin system for custom research sources

---

[2.0.1]: https://github.com/crshdn/mission-control/compare/v2.0.0...v2.0.1
[2.0.0]: https://github.com/crshdn/mission-control/compare/v1.5.3...v2.0.0
[1.4.0]: https://github.com/crshdn/mission-control/compare/v1.3.1...v1.4.0
[1.3.1]: https://github.com/crshdn/mission-control/releases/tag/v1.3.1
[1.3.0]: https://github.com/crshdn/mission-control/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/crshdn/mission-control/releases/tag/v1.2.0
[1.1.0]: https://github.com/crshdn/mission-control/releases/tag/v1.1.0
[1.0.1]: https://github.com/crshdn/mission-control/releases/tag/v1.0.1
[1.0.0]: https://github.com/crshdn/mission-control/releases/tag/v1.0.0
[0.1.0]: https://github.com/crshdn/mission-control/releases/tag/v0.1.0
