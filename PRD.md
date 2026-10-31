# Hermes for iPhone

Status: product and interaction concept  
Owner: Ren  
Artifact scope: public design document; no production client code or live credentials  
Target: native SwiftUI iPhone app

## Product brief

Hermes for iPhone is a private, native command surface for Ren's own Hermes Agent gateway. It is for checking active work, continuing a conversation, sending context, and approving consequential actions when away from the Mac.

The client connects directly to a gateway controlled by Ren over TLS, preferably through Tailscale or another private network. It has no vendor relay, analytics SDK, advertising identifier, or cloud account layer by default.

This document does not define a Hermes wire protocol. Endpoint paths, authentication headers, stream framing, event types, attachment transfer, session pagination, approval semantics, and reconnect behavior need validation against the live Hermes gateway before implementation. The app will isolate those details behind an adapter protocol.

## Product principles

1. Private by construction. Secrets stay in Keychain; message content stays out of logs and notification previews.
2. Inspect before acting. Tool activity is legible, and consequential work pauses for explicit approval.
3. Quiet while working. The interface shows compact state rather than a feed of noisy progress messages.
4. Native, not wrapped. SwiftUI conventions, Dynamic Type, VoiceOver, keyboard behavior, sheets, haptics, and platform security are first-class.
5. Recoverable. Network loss must not lose a drafted message or disguise whether an action completed.

## Audience and jobs

Primary user: Ren, connecting to a Hermes instance he controls.

Core jobs:

- Check what Hermes is doing without opening a laptop.
- Resume an existing session with full conversational context.
- Start a focused session and stream the reply as it arrives.
- Attach a photo, document, or voice note as working context.
- Inspect tool use without reading internal execution noise.
- Approve or deny an action with enough context to make a safe decision.
- Lock access quickly and revoke a compromised gateway token.

## Scope

### MVP

- Face ID or device-passcode gate on launch and after a configurable idle period
- Gateway URL and single profile setup
- Token storage in iOS Keychain, protected by LocalAuthentication
- TLS-only transport; private network recommended
- Connection test with useful, non-sensitive errors
- Session list with search, status, timestamps, create, and resume
- Streaming chat with Markdown and code rendering
- Compact tool activity rows with expandable detail
- Explicit approval sheet for consequential actions
- Photo and file attachment picker
- Record, preview, delete, and send a voice note
- Reconnect behavior and clear delivery state
- Local notifications with redacted content by default
- Dark and light appearances
- Dynamic Type, VoiceOver labels, reduced motion, and 44-point minimum targets
- Local cache for non-secret session metadata and drafts

### Later

- Push notifications through a small private relay, if background requirements demand one
- Full cron, skills, memory, bot, and admin management
- Widgets and Live Activities
- Siri and App Intents
- Durable offline send queue
- Multiple profiles and gateways
- Rich diff review and side-by-side file inspection
- iPad and macOS layouts

### Not in scope

- Hosting or brokering Hermes traffic
- Shipping credentials inside the app bundle
- Replacing the Hermes gateway permission model
- Exposing raw chain-of-thought or hidden model reasoning
- Defining an API based on assumptions
- Publishing to the App Store in this phase

## User stories

- As Ren, I can unlock with Face ID so that casual access to the phone does not expose my agent.
- As Ren, I can see recent and active sessions so that I know where work stands.
- As Ren, I can open a session and watch the answer stream without waiting for a complete response.
- As Ren, I can see that Hermes is reading a file or using a tool without receiving a new chat bubble for every step.
- As Ren, I can inspect an approval request, understand its target and risk, and approve or deny it deliberately.
- As Ren, I can send a photo, file, or voice note when typing is inconvenient.
- As Ren, I can tell whether a message is sending, sent, failed, or waiting for connectivity.
- As Ren, I can revoke the token and erase local data from settings.

## Experience flow

### First run

1. The app explains direct private connectivity and local secret storage.
2. Ren enters an HTTPS gateway URL and selects the default profile.
3. The token is entered into a secure field and written directly to Keychain.
4. The app performs a capability and connection check through the gateway adapter.
5. Face ID is enrolled as the local unlock requirement.
6. The app opens the session list.

### Returning use

1. A lock screen shows only the product name, connection posture, and unlock action.
2. Face ID succeeds.
3. Sessions appear with title, last activity, and a compact state such as Streaming, Awaiting approval, or Complete.
4. Ren resumes a session or creates one.

### Active chat

1. Messages render as a calm transcript with Markdown and code blocks.
2. Incoming text streams in place.
3. Tool events collapse into an activity row. Opening it reveals sanitized operation details and status.
4. A consequential request interrupts with an approval sheet. The sheet states the action, target, data affected, and whether it can be undone.
5. The composer supports text, attachments, and voice. Send state remains visible through reconnects.

## Information architecture

Primary navigation uses three destinations:

- Sessions: active and historical conversations
- New chat: starts a session from any screen through a prominent compose action
- Settings: connection, security, notifications, appearance, and local data

Chat is a pushed detail view. Approval is a modal sheet that blocks the related operation but not reading the transcript.

## Interaction details

### Sessions

- Active or attention-needed sessions sort above completed sessions.
- Search matches titles and locally cached non-sensitive metadata. Full message search is deferred until the gateway contract is understood.
- Swipe actions may archive locally or request remote deletion only after the API spike confirms semantics.
- Empty and offline states explain what the user can do next.

### Streaming and tools

The transcript distinguishes four states: connecting, streaming, waiting on a tool, and awaiting approval. A compact status line updates in place. Tool rows show a human label, start time, status, and sanitized target. Raw payloads are hidden by default and must never include credentials.

### Approval

Every request includes:

- Plain-language action
- Actor and target
- Scope of data or system change
- Reversibility
- Sanitized parameters
- Approve and deny controls separated spatially

High-impact approvals require a second Face ID check. The client never interprets silence, app backgrounding, or network timeout as approval.

### Attachments and voice

- Files use the system document picker and security-scoped URLs.
- Photos use PhotosPicker with selected-only access.
- Voice recording shows elapsed time, audio level, stop, replay, delete, and send.
- Upload progress and failure are attached to the draft or message, not shown as transient toast-only state.
- Size limits and accepted MIME types come from the capability response, not hard-coded assumptions.

## Functional requirements

### Authentication and connection

- FR-1: Accept only `https://` gateway URLs outside local debug builds.
- FR-2: Store bearer or gateway credentials only in Keychain with a device-only accessibility class.
- FR-3: Gate credential retrieval with LocalAuthentication according to the selected policy.
- FR-4: Support token replacement, revocation guidance, logout, and local erasure.
- FR-5: Report reachability, TLS, authentication, protocol mismatch, and server errors distinctly without echoing secrets.

### Sessions and chat

- FR-6: List, create, open, and resume sessions through the gateway adapter.
- FR-7: Stream assistant output and reconcile the final server state after reconnect.
- FR-8: Render a safe subset of Markdown, fenced code, links, lists, and tables.
- FR-9: Preserve drafts locally without storing the gateway token or raw sensitive tool payloads.
- FR-10: Expose message delivery state and retry only when doing so cannot duplicate a consequential request.

### Tool activity and approvals

- FR-11: Render normalized tool lifecycle events: queued, running, succeeded, failed, and approval required.
- FR-12: Collapse routine activity by default while preserving access to sanitized detail.
- FR-13: Require an explicit tap to approve or deny.
- FR-14: Re-authenticate locally for configured high-impact action classes.
- FR-15: Verify approval outcome from the gateway before marking it complete.

### Media and system behavior

- FR-16: Attach supported photos and files with preview and removal before send.
- FR-17: Record and send voice notes with permission handling and local cleanup.
- FR-18: Reconnect with bounded exponential backoff and user-visible state.
- FR-19: Use local notifications only for events available to the app; hide content by default.
- FR-20: Respect scene phase, protected data availability, and background execution limits.

## Non-functional requirements

- Security: no secrets in UserDefaults, SwiftData, logs, crash metadata, analytics, pasteboard, screenshots generated by the app, or notification bodies.
- Privacy: no analytics or third-party telemetry in MVP. Any later diagnostics are opt-in, redacted, and documented.
- Performance: session list usable within one second from local cache; first streamed token displayed as soon as transport delivers it; scrolling remains responsive during streaming.
- Reliability: reconnects must not reorder messages or duplicate approval decisions. Final state comes from the gateway.
- Accessibility: support VoiceOver, Dynamic Type through accessibility sizes, sufficient contrast, reduced motion, and non-color status cues.
- Compatibility: target the current public iOS SDK at implementation time; document the minimum OS after validating required APIs.
- Maintainability: transport and wire schema remain isolated from views and domain models.
- Observability: local diagnostic logs use categories, short retention, and strict redaction. Export requires deliberate user action and review.

## Architecture

### Client stack

- SwiftUI for interface and navigation
- Observation for view state
- async/await and actors for connection, session, upload, and recording coordination
- URLSession WebSocket or URLSession bytes/SSE, selected after the discovery spike
- Keychain Services plus LocalAuthentication for credentials and unlock policy
- Network.framework for path awareness, not as a substitute for end-to-end request success
- SwiftData only for non-secret cache: session summaries, draft text, UI preferences, and normalized delivery markers
- UserNotifications for redacted local alerts

### Layering

```text
SwiftUI views
    ↓
Feature models (Observation)
    ↓
HermesClient protocol
    ↓
Gateway adapter + event normalizer
    ↓
URLSession transport

KeychainVault actor      LocalCache actor      Reachability monitor
```

Proposed adapter boundary:

```swift
protocol HermesClient: Sendable {
    func capabilities() async throws -> GatewayCapabilities
    func sessions() async throws -> [SessionSummary]
    func createSession(profile: String) async throws -> SessionID
    func events(for session: SessionID) -> AsyncThrowingStream<HermesEvent, Error>
    func send(_ draft: OutboundDraft, to session: SessionID) async throws -> MessageReceipt
    func decide(_ decision: ApprovalDecision) async throws -> ApprovalReceipt
}
```

Types above describe client needs, not a confirmed server contract. The discovery spike maps verified Hermes endpoints and events into these domain types.

### Data boundaries

Keychain:

- Gateway credential
- Optional server identity or pinning metadata, if the deployment requires it

SwiftData / app container:

- Gateway hostname without credentials
- Profile label
- Session summaries and timestamps
- Draft text and pending local attachments
- Appearance and notification preferences
- Redacted diagnostic events

Memory only where practical:

- Decrypted token
- Active stream frames before normalization
- Raw approval payload before sanitization

### Transport and trust

The recommended deployment is direct HTTPS over Tailscale or another private route. The client validates normal TLS trust. Certificate pinning is optional and must include a rotation and recovery design; brittle pinning can lock the owner out during a certificate renewal.

The token is high privilege. The gateway should support scoped, revocable client credentials rather than reuse an administrator secret. If only a broad token exists, MVP documentation must state the exposure clearly and make revocation simple.

### Background behavior

iOS will suspend a direct socket. MVP does not promise continuous background streaming. On foreground, the client reconnects and asks the gateway for authoritative session state before resuming live events. Local notifications can only cover events observed while the app is active or within granted background time.

If reliable remote alerts are required later, use a minimal private push relay. It should receive opaque event identifiers, never message text or gateway credentials. The phone fetches details directly after unlock.

## API discovery spike

Before production implementation:

1. Inventory the live Hermes gateway's supported authentication and capability discovery.
2. Capture a session lifecycle: list, create, send, stream, tool activity, approval, completion, cancel, and reconnect.
3. Validate WebSocket versus SSE behavior, framing, heartbeats, ordering, replay cursors, and terminal events.
4. Test attachment upload and voice-note representation.
5. Determine pagination, error envelopes, idempotency, and token revocation.
6. Document version negotiation and unsupported capability behavior.
7. Build a local fixture server from sanitized captures for deterministic client tests.

Deliverable: a versioned adapter mapping document and executable contract tests. No endpoint name or event field enters production code until verified.

## Security and abuse cases

- Stolen unlocked phone: short auto-lock window, Face ID on launch/return, optional re-auth for approvals, protected local data.
- Token extracted from backup: use device-only Keychain accessibility and exclude credentials from backups.
- Malicious notification: no message content or actionable approval button in a notification.
- Compromised network: TLS required; private routing recommended; never allow a user-tappable bypass for certificate failures.
- Prompt-induced tool action: approvals show concrete operation details from trusted normalized events, not only model-authored prose.
- Log leakage: central redaction policy, allowlisted fields, no raw headers or event dumps.
- Duplicate send after reconnect: client-generated idempotency key if supported; otherwise require reconciliation before retry.
- Screenshot exposure: mark sensitive views for privacy where practical and obscure the app switcher snapshot.
- Deep-link abuse: no deep link can approve an action or insert a credential without in-app confirmation.

## Milestones

### 0. Contract spike

- Validate gateway auth, session, streaming, approval, media, and reconnect behavior
- Produce adapter fixtures and security notes

Exit: contract tests pass against a controlled gateway and sanitized fixtures.

### 1. Secure shell

- SwiftUI project, navigation, design tokens, lock screen
- Keychain vault, LocalAuthentication, settings, connection test
- SwiftData cache schema for non-secrets

Exit: credentials survive restart in Keychain, app locks correctly, and local storage inspection finds no token.

### 2. Sessions and streaming chat

- Session list, create/resume, transcript, Markdown/code
- Normalized stream and reconnect reconciliation
- Compact tool state

Exit: a network interruption can recover without duplicate messages or false completion.

### 3. Approvals and media

- Approval sheet and local re-auth policy
- Photo/file attachment and voice recorder
- Upload progress, errors, cleanup

Exit: approval decisions are explicit and read back from the gateway; media works within verified limits.

### 4. Hardening and pilot

- Accessibility audit, redaction tests, background/foreground tests
- Threat review, performance pass, TestFlight packaging
- Revocation and incident runbook

Exit: all MVP acceptance criteria pass on physical devices and a private pilot gateway.

## Risks and open questions

- What gateway API and event schema are stable enough for a mobile adapter?
- Can the gateway issue a scoped mobile credential with explicit revocation?
- Are approval requests durable and queryable after reconnect?
- Does session replay expose a cursor or sequence number?
- What attachment sizes, MIME types, and upload mechanism are supported?
- Is voice transcribed by Hermes, passed as audio, or both?
- Which tool fields are safe and useful to show?
- How should remote session deletion and archival differ?
- What minimum iOS version balances reach with Observation and security APIs?
- Are reliable background alerts required for MVP, or is foreground reconciliation enough?
- Is TLS pinning needed on top of a private Tailscale route?

## MVP acceptance criteria

1. A fresh install can configure an HTTPS gateway without writing a token outside Keychain.
2. Face ID or device passcode is required according to the configured lock policy.
3. A user can list, create, resume, and search sessions.
4. A reply streams progressively and settles into an authoritative final message.
5. Routine tool activity stays compact; sanitized detail is available on demand.
6. Consequential actions pause for explicit approval or denial. No timeout or background transition can approve them.
7. Approval completion is confirmed by reading gateway state.
8. Photos, supported files, and voice notes can be previewed, sent, retried safely, and removed locally.
9. Airplane mode, route changes, gateway restart, and app suspension do not create duplicate messages or incorrect success states.
10. Notifications, app-switcher snapshots, logs, cache, and exported diagnostics reveal no token or sensitive message content by default.
11. VoiceOver, Dynamic Type, reduced motion, light/dark mode, and 44-point targets pass manual review.
12. Settings can replace or revoke the credential, lock immediately, and erase local data.
13. No analytics or vendor relay is present in the binary.
14. Contract tests cover all implemented gateway events and fail clearly on incompatible versions.

## Release plan

1. Internal development against a local fixture gateway.
2. Physical-device testing over a private network with a dedicated, revocable test token.
3. Security review focused on Keychain policy, redaction, transport, approval integrity, and local storage.
4. Small private TestFlight pilot for Ren, with crash reporting disabled unless an explicitly reviewed redacted path is added.
5. Fix pilot issues and freeze the adapter contract for the first release.
6. Private distribution first. Consider wider distribution only after the gateway credential model supports safe onboarding and revocation.

Rollback is simple: expire the mobile token, remove the TestFlight build, and erase local app data. Gateway access must not depend on the phone client.

## Design direction

The product should feel like a premium native utility: near-black graphite surfaces, warm restrained amber for attention and primary action, high-contrast typography, compact status language, and clear inspection affordances. Avoid purple gradients, glass effects, oversized marketing cards, fake metrics, or decorative activity.

The phone is primarily a command/inspect surface. Information hierarchy and safe decisions matter more than visual novelty.

## Verified local tooling

The design phase machine reported:

- Xcode 26.4.1 (build 17E202)
- Apple Swift 6.3.1
- iOS 26.4 simulator runtime
- Claude Code 2.1.314, authenticated through Claude Max

These checks establish that the Mac can support a later native implementation. They do not prove gateway compatibility or production readiness.
