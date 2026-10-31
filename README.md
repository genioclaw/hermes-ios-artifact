# Hermes for iPhone — product artifact

A self-contained public product/design artifact for a private native SwiftUI client that connects to an owner-controlled Hermes Agent gateway.

## Open

Open `index.html` directly in a modern browser. It also works as a GitHub Pages root with no build step, package manager, external fonts, or remote assets.

## Contents

- `index.html` — responsive PRD reading layout and clickable iPhone prototype
- `PRD.md` — full product requirements document
- `README.md` — project orientation

## Prototype path

1. Unlock with Face ID
2. Open or create a session
3. Inspect streaming and tool activity
4. Open the approval request and approve or deny it
5. Open the attachment tray and voice recorder
6. Review gateway and security settings

Use the screen tabs beside the phone to jump directly to any state. Keyboard focus and reduced-motion preferences are supported.

## Security posture

- Direct TLS connection to Ren's gateway, preferably over Tailscale or another private route
- High-privilege credential stored only in iOS Keychain and gated with LocalAuthentication
- No vendor relay or analytics by default
- Redacted notifications and diagnostics
- Explicit approval for consequential actions
- Easy token replacement, revocation, lock, and local-data erase

The artifact contains no credentials and does not connect to a live gateway.

## Important contract note

No Hermes endpoint or event schema is assumed. The PRD requires a discovery spike and places all verified gateway behavior behind a `HermesClient` adapter protocol. The visual prototype uses local fixture data only.

## Tooling recorded for the later native build

- Xcode 26.4.1
- Swift 6.3.1
- iOS 26.4 simulator runtime
- Claude Code 2.1.314 authenticated through Claude Max

## Visual direction

Command/inspect surface with graphite and near-black materials, restrained warm amber, compact state language, and high information clarity. It deliberately avoids gradients, glassmorphism, fake metrics, and generic SaaS feature tiles.
