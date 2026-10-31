# Hermes for iPhone: public product artifact

A self-contained, public, sanitized design artifact for a private native SwiftUI client that talks to an owner-run [Hermes Agent](https://hermes-agent.nousresearch.com/) gateway.

> **Everything here is a demo fixture.** "Ren" is a sanitized persona: a Brazilian software entrepreneur whose company, Genio Tech, builds Genio IA, Anotar, and GlipOne. Ren studies German A2 and Mandarin A1 and is interested in tennis, aviation, travel, fitness, and design. Reminders, messages, calendar items, products, prices, merchants, deliveries, memories, and confidence values are invented. No credentials, hostnames, accounts, addresses, contacts, or business metrics are included.

## Open

Open `index.html` in a modern browser. No build step, package manager, external fonts, remote images, or network requests. It also works as a static GitHub Pages root.

## Files

- `index.html`: page with an interactive iPhone prototype and a product summary (inline CSS and JS)
- `PRD.md`: full product requirements: surfaces, structured blocks, knowledge-graph governance, the documented Hermes API surface and discovery plan, offline/reconnect, architecture, security, milestones, risks, acceptance criteria
- `README.md`: this file

## Prototype tour

The phone starts locked. Tap **Unlock with Face ID (demo)**; no biometric runs in a browser. Use the phone's tab bar or the list beside the phone.

| Surface | Try |
|---|---|
| **Today** | Check off reminders; expand the read-only inbox and calendar briefings; tap the Travel/Fitness cards to deep link to a map node, Business to prefill a chat draft, Shopping to compare |
| **Chat** | Expand the tool row; toggle a structured block's text fallback; attach demo photo/file chips and remove them; record a simulated voice note; send (fixture reply); open the approval and approve or deny |
| **Learn** | German A2: play the example (browser speech, if available), rate **1 new / 2 forgot / 3 known** with buttons or keys. Each rating jumps to the next unseen word. Mandarin A1: reveal pinyin and meaning. Flags: guess in English or Portuguese, then see EN/PT names, location, and ISO codes |
| **Shop** | Favorite, favorites-only filter, eliminate/restore, select 2–3 to compare. Images are illustrative inline SVG; price, merchant, and delivery are marked unverified |
| **Knowledge Map** | Drag to pan, pinch or scroll to zoom, +/−/reset buttons, tap a node for provenance, confidence, and privacy. Follow related chips, correct a summary, forget a node (with undo), filter categories |
| **Settings** | Simulate offline (chat sends queue, then reconcile on reconnect), hide personal map nodes, lock now, reset demo state, demo revoke/erase |

Escape closes sheets and the inspector. All demo changes reset on reload.

## Posture

- Direct TLS to the owner's gateway over a private network (Tailscale or similar); never public
- Gateway key in a device-only Keychain item behind Face ID; dedicated mobile profile and key
- No vendor relay, analytics, or content in notifications
- Approvals need an explicit decision; offline or timeout never approves
- Briefings are read-only; shopping never purchases; memory deletion is verified, not cosmetic

## API contract note

The PRD uses only endpoints documented on the Hermes Agent [API Server](https://hermes-agent.nousresearch.com/docs/user-guide/features/api-server) page (read 2026-10-31): health, capabilities, chat completions, responses, runs with SSE events and approvals, sessions, jobs. It lists what is **not** documented: memory/graph, briefings, learning records, shopping data, structured UI blocks, file upload. Those surfaces need discovery and an adapter mapping before implementation. The structured `hermes-block` format is a proposal, not a Hermes feature.

## Known limitations of this artifact

- No real Face ID, microphone, gateway, model, merchant, or memory store
- Audio uses the browser's speech synthesis when present; quality and voice availability vary
- Flags are simplified geometric drawings, not official artwork; product images are illustrations of fictional products
- The map is a fixed demo layout, not a force-directed layout of real memory
- Demo revoke/erase use browser `confirm()` dialogs

## Local tooling (checked 2026-10-31)

Xcode 26.4.1 (17E202), Apple Swift 6.3.1, iOS 26.4 simulator runtime.
