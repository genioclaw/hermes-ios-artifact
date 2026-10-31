# Hermes for iPhone: Product Requirements

Status: product and interaction concept, public and sanitized
Artifact scope: design document plus a self-contained HTML prototype (`index.html`). No production client code, no credentials, no live data.
Target: native SwiftUI iPhone app talking to one owner-run Hermes Agent gateway

> **Fixture notice.** "Ren" is a sanitized demo persona: a Brazilian software entrepreneur whose company, Genio Tech, builds the apps Genio IA, Anotar, and GlipOne. Ren studies German (A2) and Mandarin (A1) and is interested in tennis, aviation, travel, fitness, and design. Every reminder, message, calendar entry, product, price, merchant, delivery estimate, memory node, confidence value, and timestamp in this repository is invented for illustration. None of it is account data, business data, or a claim about real progress.

---

## 1. Product brief

Hermes for iPhone is a private, native client for the owner's own Hermes Agent gateway. Chat stays the universal entry point. Work that recurs every day gets a dedicated surface that is faster to scan and safer to act on than a transcript:

| Surface | Job | Default posture |
|---|---|---|
| **Today** | Morning brief: reminders, inbox and calendar summaries, travel, fitness, and business cards that deep link to other surfaces | Read-only, except checking off reminders |
| **Chat** | Start or resume anything, attach context, watch tool activity, decide on approvals | Command |
| **Learn** | German A2 target words with example and audio, Mandarin A1 characters, flag challenge | Drill |
| **Shop** | Research shortlist with images, price, specs, merchant, delivery; favorite, eliminate, compare | Inspect; never buys |
| **Knowledge Map** | Visual graph of what Hermes knows, with provenance, confidence, privacy, and correct/forget controls | Inspect and govern |
| **Settings** | Connection, lock policy, privacy filters, revocation, local erase | Secure |

The client connects directly to the gateway over TLS on a private network (Tailscale or equivalent). There is no vendor relay, no analytics SDK, no advertising identifier, and no cloud account layer.

## 2. Principles

1. **Private by construction.** The gateway key lives only in Keychain. Content stays out of logs, analytics, and notification previews.
2. **Inspect before acting.** Consequential work pauses for an explicit, informed decision. Silence, timeouts, and backgrounding never approve.
3. **Every structured view degrades to text.** If the client cannot render a block, it shows the text fallback. Nothing is silently dropped.
4. **Memory is accountable.** Every remembered fact shows where it came from, how sure Hermes is, who can see it, and how to correct or delete it.
5. **Honest state.** The app never shows "sent", "approved", "deleted", or "verified" until the gateway confirms it. Unverified commerce data says so.
6. **Native, not wrapped.** SwiftUI conventions, Dynamic Type, VoiceOver, 44-pt targets, reduced motion, platform security.
7. **No invented contracts.** Only documented, verified gateway behavior reaches production code. Everything else is labelled a proposal.

## 3. Audience and jobs

Primary user: the owner of a single Hermes Agent deployment, using an iPhone away from the desk.

- Check what needs attention today in under a minute, without opening mail or calendar apps.
- Keep a daily language and geography routine going in short sessions.
- Narrow a purchase decision without opening merchant pages.
- See and fix what Hermes believes before it acts on a wrong belief.
- Continue a conversation, send a photo, and approve or deny a paused action.
- Lock the client and revoke its credential quickly.

### Non-goals

- Hosting or relaying Hermes traffic
- Purchasing, paying, sending mail, accepting invitations, or editing calendars from the phone in MVP
- Replacing the gateway's permission model
- Exposing raw chain-of-thought
- App Store distribution in this phase
- Any endpoint, event, or field that has not been verified against the deployed gateway

## 4. Surface requirements

### 4.1 Today

- **Reminders.** List of due items with source label (business, fitness, learning). Checking one off is a local optimistic change that becomes durable only after the gateway confirms the write. If no verified reminder write path exists, check-off stays local and is labelled so.
- **Inbox briefing.** Summaries of threads worth attention: sender label, one-line gist, time. Read-only. No send, reply, archive, delete, or mark-read controls in MVP. Each briefing carries a citation (source system and retrieval time).
- **Calendar briefing.** Today's events and open focus blocks. Read-only. Attendees, locations, and meeting links are hidden by default and revealed only after unlock.
- **Workspace cards.** Travel, Fitness, Business (Genio Tech), Shopping, and Learning cards. Each has one primary deep link.
- **Deep links.** Proposed in-app routes (not registered URL schemes until security review):
  - `hermes://learn/german`, `hermes://learn/mandarin`, `hermes://learn/flags`
  - `hermes://shop/<workspace>`
  - `hermes://map/node/<node-id>`
  - `hermes://chat/new?prompt=<template-id>` prefills a draft and never auto-sends
  - Rule: no deep link can approve, delete, purchase, send, or insert a credential. Deep links only navigate and prefill.

### 4.2 Chat

- Session list with search and attention-needed sorting.
- Streaming transcript with Markdown and code.
- Tool activity collapsed to one row per tool call: label, status, sanitized target, expandable detail.
- Structured blocks inline (section 5), each with a "text fallback" disclosure.
- Composer: text, photo, file, voice note; preview and remove before send; per-message delivery state (`Sending`, `Sent`, `Waiting for connection`, `Failed: retry`).
- Approval card in transcript plus a modal sheet. The sheet shows action, schedule, scope, notification content, reversibility, and money impact, with Deny and Approve separated. High-impact classes require Face ID again.

### 4.3 Learn

**German A2.** One target word at a time: word with article and part of speech, English gloss, example sentence, translation, audio.

- Rating scale: **1 = new**, **2 = forgot** (seen before but not recalled), **3 = known**.
- Any rating records the result and **immediately shows the next unseen word**. No extra "next" tap, no repeat within the session.
- Keyboard shortcuts 1/2/3 on hardware keyboards. Buttons are at least 44 pt and labelled for VoiceOver.
- Session tally and a completion state. Ratings are queued offline and reconciled on reconnect, keyed by word ID and rating timestamp so a replay cannot double-count.
- Audio: MVP uses `AVSpeechSynthesizer` with a German voice, labelled as device speech. Recorded or gateway-produced audio needs a verified media path.

**Mandarin A1.** Character first, then reveal pinyin with tone marks, meaning, and an example. Audio via `AVSpeechSynthesizer` (zh-CN). Characters render with system CJK fonts and respect Dynamic Type.

**Flag challenge.** Show a flag with the answer hidden. The user types a guess (English or Portuguese, accent- and case-insensitive) or reveals without guessing. The reveal shows the **English name, Portuguese name, location (region and capital), and ISO 3166-1 alpha-2/alpha-3**. Flag images ship as bundled vector assets with a documented source and license. The prototype uses simplified geometric drawings.

### 4.4 Shop

- Product cards show **image, price, key specs, merchant, delivery estimate**, and a verification badge on each commercial field (`verified at <time>` or `unverified`).
- Images: merchant images load only through the gateway or a vetted image proxy decision made during discovery, never by hotlinking from arbitrary hosts in the client. A missing image shows a labelled placeholder, never a stand-in that looks like the real product.
- Actions: **Favorite** (toggle; favorites-only filter), **Eliminate** (moves the item to an "Eliminated" section with restore; removes it from comparison), **Compare** (2–3 items side by side in a sheet).
- The client never purchases, adds to cart, or stores payment data. A retailer link opens Safari after a confirmation that names the destination host.
- Seller claims (battery life, weight) are labelled "claimed" unless a cited test exists.

### 4.5 Knowledge Map

- Nodes for the person, work (Genio Tech and its apps), learning (German, Mandarin, flags), travel, fitness, and interests (tennis, aviation, travel, fitness, design), connected by labelled edges.
- **Touch:** one-finger pan, pinch zoom around the gesture midpoint, tap to inspect. Zoom buttons and a reset control for users who cannot pinch. Node labels keep a 44-pt hit area at every zoom level. The canvas is not scaled.
- **Inspector** (bottom sheet): title, summary, category, **provenance** (source type, reference, time), **confidence** (value plus plain-language band: stated/confirmed, likely, inferred), **privacy class**, last updated, correction state, connected nodes as tappable chips.
- **Controls:** filter by category; hide personal nodes; **Correct** (edit summary → pending correction → confirmed); **Forget** (confirm → tombstone → gateway deletion → readback). Forgetting the root profile node is not allowed from the map.
- **Accessibility:** a list view of the same nodes and edges for VoiceOver users. The map is never the only way to reach a memory.

### 4.6 Settings

- Connection: gateway URL (HTTPS only), profile, route status, connection test against documented health and capabilities endpoints.
- Security: Face ID on launch and after idle, re-auth for approvals, notification preview policy, lock now.
- Knowledge Map privacy: show/hide personal nodes; sensitive class is never projected.
- Local data: reset caches, revoke mobile key (local removal plus instructions to rotate on the gateway), erase all local data.
- A "simulate offline" switch exists only in debug builds and the prototype.

## 5. Structured content blocks

### 5.1 Problem

The documented Hermes API server returns OpenAI-compatible chat output, typed Responses events, and run events. It does not document a structured UI block type. Lessons, flag quizzes, products, comparisons, briefings, and graph patches therefore need a convention that (a) the native app can render richly, and (b) every other Hermes surface (CLI, messaging bots) can still read as plain text.

### 5.2 Proposed convention (to validate in discovery)

The assistant message stays human-readable text. Structured data rides alongside it in a fenced code block with a reserved info string:

````text
Here is today's word: **die Reise** (the trip).
Example: Die Reise nach Lissabon war entspannt.

```hermes-block
{"type":"lesson","version":1,"id":"lsn_…","fallbackText":"German A2 · die Reise (the trip)…",
 "provenance":[{"kind":"skill","ref":"daily-learning","at":"…"}],
 "sensitivity":"personal","payload":{…}}
```
````

- A Hermes skill or system instruction produces blocks. That integration must be confirmed with the gateway owner. It is not a server feature today.
- The client strips `hermes-block` fences from the visible text and renders the block in place. Other surfaces show the surrounding prose plus a JSON code block, which is still readable.
- If the gateway later documents a native structured-output channel, the adapter switches to it without changing the domain model.

### 5.3 Envelope

```text
ContentBlock {
  type: lesson | flagQuiz | product | comparison | briefing | approval | graphPatch | citation
  version: Int
  id: String                 // stable, for dedupe across reconnects
  fallbackText: String       // required, plain text, complete on its own
  provenance: [Provenance]   // at least one
  sensitivity: public | personal | sensitive
  payload: JSON              // schema per (type, version)
}
```

Rules:

- **Unknown type or version → render `fallbackText`.** Never discard, never crash.
- **Malformed payload → render `fallbackText`** and log a redacted local diagnostic (type, version, error class; no content).
- `fallbackText` is required. A block without it renders as the raw prose around it.
- Blocks are idempotent by `id`. A replayed stream must not duplicate a card.
- `sensitivity: sensitive` blocks are never cached to disk and never appear in notifications.
- Actions inside blocks (rate, favorite, approve) produce client intents, not direct writes. Section 8 defines how intents reach the gateway.

## 6. Knowledge graph governance

### 6.1 Records

```text
GraphNode {
  id, category, title, summary,
  provenance: [Provenance], confidence: 0...1, confidenceBand,
  sensitivity: public | personal | sensitive,
  createdAt, updatedAt,
  correction: none | pending | confirmed | rejected,
  lifecycle: active | tombstoned | deleted
}
GraphEdge { id, from, to, relation, provenance, confidence, updatedAt, lifecycle }
Provenance { kind: stated | inferred | imported | ownerEdited | skill,
             ref: String, at: Date, excerpt?: String /* redacted, optional */ }
```

### 6.2 Source of truth

Hermes Agent keeps its own memory. The documented API mentions an `X-Hermes-Session-Key` header that scopes long-term memory. It does not document a memory or graph read/write API. The map is therefore a **projection**: a derived, read-mostly view built from whatever verified memory export path discovery finds (a skill that emits `graphPatch` blocks, a documented endpoint in a later gateway version, or an owner-run export). The client never becomes the system of record for memory.

### 6.3 Policy

- **Projection filter before layout.** `sensitive` records are excluded on the gateway side before they reach the phone. `personal` records are shown by default and can be hidden. `public` is information already public (for example, published app names).
- **Confidence bands.** ≥ 0.85 "stated or confirmed", 0.70–0.85 "likely", < 0.70 "inferred, unconfirmed". Inferred nodes are visually distinct and never drive actions without confirmation.
- **Correction.** An owner edit creates a pending correction carrying the old value, new value, and time. It shows as pending until the gateway confirms. A confirmed correction replaces the summary, sets provenance to `ownerEdited`, and keeps the prior source as history.
- **Deletion ("Forget").** Two-step confirm → local tombstone (node hidden, edges hidden, not cached) → deletion request through a verified path → readback that the memory is gone → purge the tombstone. If no verified deletion path exists, the UI must say "hidden on this device only" and not "forgotten".
- **Export.** Owner-initiated only, excludes `sensitive`, shows a preview, and uses the share sheet. No background export.
- **Audit.** Local, redacted log of corrections and deletions (node ID, action, time, result) that the owner can view and clear.
- **Retention.** The cached projection expires and is rebuilt on unlock. Tombstones persist until deletion is confirmed.

## 7. Gateway integration: documented surface and discovery

### 7.1 What is documented

Source: Hermes Agent user guide, "API Server" page (<https://hermes-agent.nousresearch.com/docs/user-guide/features/api-server>), read 2026-10-31. Endpoints, fields, and behavior must be re-verified against the **deployed** gateway version before use.

| Need | Documented surface | Notes for the client |
|---|---|---|
| Liveness | `GET /health`, `GET /v1/health`, `GET /health/detailed` | Connection test |
| Feature gating | `GET /v1/capabilities` | Includes `auth`, `features` (`chat_completions`, `responses_api`, `run_submission`, `run_status`, `run_events_sse`, `run_stop`, …) and `endpoints`. The client enables features only when advertised |
| Chat | `POST /v1/chat/completions` (stateless; `stream: true` → SSE with `chat.completion.chunk` and `hermes.tool.progress`); `POST /v1/responses`, `GET`/`DELETE /v1/responses/{id}` (server-side state via `previous_response_id`) | Choose one in discovery |
| Sessions | `GET`/`POST /api/sessions`; `GET`/`PATCH`/`DELETE /api/sessions/{id}`; `GET …/{id}/messages`; `POST …/{id}/fork`; `POST …/{id}/chat`; `POST …/{id}/chat/stream` | Session list, resume, history after reconnect |
| Long work + approvals | `POST /v1/runs`; `GET /v1/runs/{run_id}` (exposes `pending_controls`); `GET …/events` (SSE: `tool.started`, `tool.completed`, `message.interim`, `approval.request`, `run.completed`, …); `POST …/approval` (`request_id`, `execution_generation`, `choice`); `POST …/clarify`; `POST …/stop` | Approval sheet and tool rows |
| Scheduled work | `/api/jobs` (list, create, get, update, delete, pause, resume, run) | Backs "create a fare-watch job"-style approvals |
| Introspection | `GET /v1/models`, `GET /v1/skills`, `GET /v1/toolsets` | Settings diagnostics |
| Auth | `Authorization: Bearer <API_SERVER_KEY>` required on every deployment; per-profile routes `/p/<profile>/…` require that profile's own key | Dedicated mobile profile and key |
| Headers | `X-Hermes-Session-Id` (continue session), `X-Hermes-Session-Key` (memory scope), `Idempotency-Key` (allowed in CORS headers) | Idempotency semantics need verification |
| Images | `image_url` / `input_image` parts with `http(s)` or `data:image/…` URLs | Photos supported |

Documented constraints that shape the product:

- The API server **gives full access to Hermes's toolset, including terminal commands.** The mobile key is effectively a remote-execution credential.
- It binds to `127.0.0.1:8642` by default and is disabled until enabled. Exposing it to the phone is a deliberate operator step.
- **No file upload.** Non-image `data:` URLs and file parts return `400 unsupported_content_type`.
- API sessions **cannot bind to sessions owned by other surfaces** (CLI, TUI, desktop, messaging) and return `409 permission_denied`. The phone cannot take over a Telegram or CLI conversation.
- Unconsumed run-event buffers **expire after five minutes**. Status polling still works.
- SSE sends `: keepalive` comments after 10 s of silence. Parsers must ignore lines starting with `:`.
- Stale `execution_generation` or foreign request IDs on approval return `409`.

### 7.2 Not documented (no endpoint will be invented)

- Reminders, inbox, and calendar briefings (likely produced by Hermes skills or tools; the output channel must be discovered)
- Learning records and spaced-repetition state
- Shopping offers, verification timestamps, and images
- Memory/knowledge-graph read, correct, delete, and export
- Structured content blocks (section 5 proposes a convention)
- Push notifications
- Voice notes and non-image files (given "no file upload")

For each, discovery picks one of: a documented endpoint in the deployed version, a skill that emits `hermes-block` output over the documented chat or run channel, or deferral. The decision is recorded in an adapter mapping document.

### 7.3 Discovery spike

1. Record the deployed Hermes version, `GET /v1/capabilities` output, and enabled toolsets (sanitized).
2. Confirm the auth model: a dedicated mobile profile with its own key; whether toolsets can be restricted per profile; key rotation procedure.
3. Capture sanitized transcripts of: session create → stream → tool events → approval → completion; stop; clarify; reconnect within and after the 5-minute event buffer.
4. Decide Chat Completions vs Responses vs Runs for chat. Recommendation to test: Runs for anything that can request approval, sessions API for history.
5. Verify `Idempotency-Key` semantics on send and approval.
6. Validate image attachments (size limits, data URL vs URL). Confirm file and voice are unsupported.
7. Prototype the `hermes-block` convention with one skill (daily learning) and test how it renders on another Hermes surface.
8. Find a memory projection path. If none exists, the Knowledge Map ships read-only from a skill export or is deferred.
9. Build a local fixture gateway from sanitized captures. Contract tests run against it in CI and against a controlled real gateway before release.

Deliverable: `AdapterMapping.md` plus executable contract tests. Nothing unverified enters production code.

## 8. Offline, reconnect, and consistency

- **Cache (SwiftData, non-secret only):** session summaries, last brief, learning deck, shop workspace, graph projection (non-sensitive), drafts. Each cached item records `fetchedAt`. Stale data shows its age.
- **Outbox actor:** queued intents (`sendMessage`, `rateWord`, `toggleReminder`, `favorite`, `eliminate`, `correctNode`, `forgetNode`), each with a client UUID and created time. Persisted across launches.
- **Never queued:** approvals and clarifications. They are tied to a live `execution_generation` and must be decided online. Offline, the approval sheet shows "Reconnect to decide". Nothing auto-approves.
- **Reconnect sequence:** path change or foreground → health check → capabilities (version gate) → for each active run, `GET /v1/runs/{id}` (authoritative status and `pending_controls`) → resubscribe to events if the buffer is still valid, otherwise reconcile from status and session messages → flush outbox in order with idempotency keys if verified; otherwise check server state before each retry.
- **Dedupe:** messages by server ID; blocks by block `id`; `message.interim` respects `already_streamed`.
- **UI states:** global banner (offline / reconnecting / reconciled N items), per-item state on messages and pending corrections. "Sent" and "Deleted" appear only after server confirmation.
- **Background:** iOS suspends sockets. MVP promises no background streaming. Foreground reconciliation is the baseline. A later private push relay would carry only opaque event IDs.

## 9. Architecture

### 9.1 Stack

- **SwiftUI** for all UI; `NavigationStack`, `TabView` (Today, Chat, Learn, Shop, Map; Settings from a toolbar button), sheets for approvals, comparison, and inspector.
- **Observation** (`@Observable`) feature models per surface; views hold no transport state.
- **Swift concurrency:** `async/await`, `AsyncThrowingStream` for SSE, and **actors** for shared mutable state:
  - `GatewayClient` actor: URLSession, auth header injection, SSE parsing, retry/backoff
  - `RunCoordinator` actor: run lifecycle, pending controls, approval submission
  - `Outbox` actor: durable intent queue and flush
  - `KeychainVault` actor: key storage and retrieval behind LocalAuthentication
  - `LocalCache` actor: SwiftData model context for non-secret cache
  - `GraphProjection` actor: filtering by sensitivity before layout, tombstones
- **Keychain:** gateway key as a generic password, `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`, access control `.biometryCurrentSet` or `.userPresence` per lock policy; excluded from backups and never synced.
- **LocalAuthentication / Face ID:** unlock on launch and after idle; `LAContext` re-evaluation for high-impact approvals; passcode fallback per policy.
- **TLS:** HTTPS only (ATS default, no exceptions). Standard trust evaluation; optional pinning only with a rotation and recovery plan.
- **Private gateway route:** Tailscale (tailnet address or `tailscale serve` with HTTPS certificates) or another private network. The gateway is never exposed on the public internet.
- **Network.framework** `NWPathMonitor` for path hints only; request success is authoritative.
- **AVFoundation** `AVSpeechSynthesizer` for lesson audio; **PhotosUI** `PhotosPicker` for selected-only photo access.
- **UserNotifications** with content hidden by default.

### 9.2 Layering

```text
SwiftUI views
   ↓
@Observable feature models (Today, Chat, Learn, Shop, Map, Settings)
   ↓
Domain: ContentBlock, GraphNode, Intent, Session, Run, Approval
   ↓
HermesClient protocol  ←  only verified behavior crosses here
   ↓
Gateway adapter (versioned) + event normalizer + block parser
   ↓
GatewayClient actor (URLSession, SSE)

KeychainVault · LocalCache · Outbox · GraphProjection · PathMonitor
```

```swift
protocol HermesClient: Sendable {
    func health() async throws -> GatewayHealth
    func capabilities() async throws -> GatewayCapabilities
    func sessions() async throws -> [SessionSummary]
    func messages(in session: SessionID) async throws -> [Message]
    func startRun(_ request: RunRequest) async throws -> RunID
    func events(for run: RunID) -> AsyncThrowingStream<RunEvent, Error>
    func status(of run: RunID) async throws -> RunStatus   // includes pending controls
    func decide(_ decision: ApprovalDecision) async throws -> ApprovalReceipt
    func stop(_ run: RunID) async throws
}
```

Method names are client domain names. The adapter maps them to the documented endpoints in 7.1 after discovery confirms them.

### 9.3 Storage boundaries

| Keychain | SwiftData / app container | Memory only |
|---|---|---|
| Gateway key (per profile) | Hostname, profile label, lock policy | Decrypted key |
| Optional pinning metadata | Session summaries, drafts, outbox intents | Raw SSE frames before normalization |
| | Non-sensitive cached blocks and graph projection | Sensitive blocks |
| | Redacted diagnostics, audit log | Raw approval payloads before sanitizing |

## 10. Security and privacy

- **Credential scope.** Use a dedicated Hermes profile for mobile with its own key (`/p/<profile>/…`). Restrict that profile's toolsets if the gateway supports it (open question). Never reuse an administrator key.
- **Stolen unlocked phone:** short idle lock, Face ID on return, re-auth for approvals, protected data class `complete` for cache files.
- **Backup extraction:** device-only Keychain item; cache excluded from backup where practical.
- **Notifications:** no content and no actionable approve button.
- **Prompt-injected actions:** approval sheets render from normalized run fields (`pending_controls`), not model prose alone.
- **Commerce:** no purchase capability; retailer links confirm the host; prices and delivery always show their verification state.
- **Memory:** sensitive class never leaves the gateway in projections; forget is verified, not cosmetic.
- **Logs:** allowlisted fields, no headers, no bodies, short retention; export only by deliberate user action.
- **Snapshots:** obscure the app switcher; mark sensitive views private where supported.
- **Deep links:** navigation and prefill only.
- **Public artifact hygiene:** this repository contains no hostnames, keys, accounts, addresses, contacts, or business metrics. `gateway.example` is a placeholder.

## 11. Accessibility and design

- Graphite surfaces, warm amber for attention and primary action, green for confirmed, coral for destructive or unverified. Status is never conveyed by color alone.
- 44-pt minimum targets everywhere, including map nodes at every zoom.
- Dynamic Type through accessibility sizes; VoiceOver labels and actions (rate word, favorite, compare, inspect node); a list alternative for the map; reduced motion disables sheet and map animations.
- No fake metrics, gradients, glass effects, or decorative activity.

## 12. MVP scope and milestones

Milestones are ordered by dependency. Each has an exit criterion.

**M0 · Contract discovery.** Section 7.3. Exit: adapter mapping document and contract tests pass against the fixture gateway and a controlled real gateway. Go/no-go per surface recorded.

**M1 · Secure shell.** Project, tab structure, design tokens, lock screen, Keychain vault, Face ID policy, settings, connection test against health and capabilities. Exit: storage inspection finds no key outside Keychain; lock policy works on device.

**M2 · Chat and approvals.** Sessions, streaming, tool rows, run approvals and clarify, photo attachments, outbox, reconnect. Exit: network loss mid-run recovers without duplicate messages or false completion; approvals cannot be decided offline or by timeout.

**M3 · Structured blocks + Learn.** Block parser with fallback, lesson and flag blocks, German rating flow, Mandarin, flags, device-speech audio. Exit: unknown and malformed blocks render fallback; ratings reconcile without double counting.

**M4 · Today + Shop.** Brief cards, read-only inbox and calendar summaries, deep links, product and comparison blocks, favorite, eliminate, compare. Exit: every commercial field shows verification state; no write path exists for mail or calendar.

**M5 · Knowledge Map.** Projection, filters, inspector, correct and forget with verified paths (or labelled device-only hide if no path exists), list alternative. Exit: sensitive records never reach the device; forget shows "forgotten" only after readback.

**M6 · Hardening and private pilot.** Accessibility audit, redaction tests, threat review, background/foreground tests, TestFlight to the owner only. Exit: all acceptance criteria pass on physical devices.

Deferred: push relay, widgets, Live Activities, App Intents, voice notes and file upload (blocked by documented API), mail/calendar actions, purchase flows, iPad/macOS, multiple gateways.

## 13. Risks and open questions

| Risk / question | Impact | Mitigation |
|---|---|---|
| The API key grants full toolset access including terminal | Phone compromise = remote execution | Dedicated mobile profile, restricted toolsets if supported, Face ID, fast revocation, private network only |
| No documented memory/graph API | Map may be read-only or deferred | Projection via skill export; label device-only hides honestly |
| No documented structured output | Rich surfaces depend on a convention | `hermes-block` convention with text fallback; validate on other surfaces |
| API sessions cannot bind to other surfaces' sessions (409) | Can't continue a Telegram/CLI thread on the phone | Set expectation in UI; start phone-owned sessions |
| No file upload | Files and voice notes unsupported | Photos only in MVP; on-device text extraction or transcription only after privacy review |
| 5-minute event buffer expiry | Lost live events after long suspension | Reconcile from run status and session messages |
| `Idempotency-Key` semantics unverified | Duplicate sends after retry | Verify; otherwise check server state before retry |
| API changes between Hermes versions | Adapter breakage | Capability gating, version check, contract tests per release |
| Shopping data staleness and image rights | Misleading prices or images | Verification timestamps, "unverified" badges, no hotlinking, no purchase |
| Inference errors in memory | Wrong personalization | Confidence bands, inferred styling, correction flow |
| Exposing the gateway beyond loopback | Attack surface | Tailnet-only binding or `tailscale serve`; never public |

Open questions: Can toolsets be restricted per profile? Is there a supported way to read or delete individual memories? Which skills produce briefings today and where does their output go? Is device speech acceptable for German and Mandarin pronunciation quality? Which minimum iOS version is required for the Observation and SwiftData features used?

## 14. Acceptance criteria

**Security and connection**
1. A fresh install configures an HTTPS gateway; the key exists only in a device-only Keychain item.
2. Face ID or passcode is required per lock policy; "Lock now" works immediately.
3. Revoke removes the local key and shows gateway-side rotation steps; erase removes all local data.
4. No analytics SDK or vendor relay in the binary; no network traffic except to the configured gateway (and Safari after a confirmed retailer link).

**Chat and approvals**
5. Streaming replies settle to the server's final message; tool activity is one compact row per call.
6. Approvals show action, scope, schedule, reversibility, and money impact; they require an explicit tap; high-impact ones require Face ID; offline shows "Reconnect to decide"; timeout never approves; completion is read back from run status.
7. Photos can be previewed, removed, and sent; unsupported attachment types are explained, not silently dropped.

**Structured blocks**
8. Every block type renders; unknown types/versions and malformed payloads render `fallbackText`; replay never duplicates a block.

**Learn**
9. German shows word, gloss, example, translation, and audio; ratings 1/2/3 each record and immediately advance to the next unseen word; offline ratings reconcile once.
10. Mandarin shows characters, then pinyin and meaning on reveal; flags hide the answer until guess or reveal, then show English name, Portuguese name, location, and ISO codes.

**Today and Shop**
11. Inbox and calendar briefings are read-only with citations; no send/archive/edit control exists.
12. Deep links navigate and prefill only.
13. Product cards show image (or labelled placeholder), price, specs, merchant, and delivery with verification state; favorite, eliminate/restore, and 2–3-item compare work; no purchase path exists.

**Knowledge Map**
14. Pan, pinch zoom, zoom buttons, reset, and tap-to-inspect work with 44-pt targets at all zoom levels; a list alternative exists.
15. The inspector shows provenance, confidence band, privacy class, updated time, correction state, and related nodes.
16. Correct shows pending until confirmed; Forget tombstones locally and shows "forgotten" only after gateway readback (or "hidden on this device" if no path exists); sensitive records never reach the device.

**Offline and quality**
17. Airplane mode, route change, gateway restart, and app suspension cause no duplicate messages, ratings, or false success states.
18. Notifications, app-switcher snapshots, logs, cache, and diagnostics contain no key or sensitive content.
19. VoiceOver, Dynamic Type at accessibility sizes, reduced motion, and contrast pass manual review on device.
20. Contract tests cover every implemented endpoint and event and fail clearly on an incompatible gateway version.

## 15. Release plan

1. Develop against the fixture gateway built from sanitized captures.
2. Physical-device testing over the private network with a dedicated, revocable mobile key.
3. Security review: Keychain policy, transport, redaction, approval integrity, memory projection, deep links.
4. Private TestFlight for the owner only; crash reporting off unless a reviewed redacted path exists.
5. Freeze the first adapter contract against a pinned Hermes version.

Rollback: rotate the mobile key on the gateway, remove the TestFlight build, erase local data. Gateway operation never depends on the phone.

## 16. Prototype mapping

`index.html` demonstrates the interaction model with fixtures only. It has no model, gateway, Face ID, microphone, payment, or merchant behind it. Demo actions are labelled. Browser speech synthesis stands in for device speech. The "simulate offline" switch demonstrates queueing and reconciliation. Product images are illustrative inline SVG drawings of fictional products. Flags are simplified geometric drawings.

## 17. Local tooling (checked 2026-10-31)

- Xcode 26.4.1 (build 17E202)
- Apple Swift 6.3.1
- iOS 26.4 simulator runtime

These show the Mac can host a later native build. They say nothing about gateway compatibility or production readiness.
