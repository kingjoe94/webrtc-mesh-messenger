# WebRTC Mesh Messenger MVP Design

## Goal

Build a browser-only 1-to-1 text messaging MVP using WebRTC DataChannel and QR-based manual signaling.

The first MVP should prove only this:

- Two users can open the same single HTML file.
- Each browser gets a local device identity and key pair.
- The users exchange QR codes face to face.
- If both users are online, a text message can be delivered over WebRTC DataChannel.
- Message bodies are application-layer encrypted before DataChannel send.
- No server is needed for Phase 1.

Do not start with a large peer-to-peer network, native app, push/background system, or public service.

Important runtime note: "single HTML" does not mean "send the file to a phone and all browser APIs will work." Mobile browsers commonly require HTTPS for camera access and other powerful APIs. For phone QR testing, serve the single HTML from an HTTPS page.

## Non-Goals For The First MVP

- No mobile background operation guarantee.
- No Service Worker relay.
- No Background Sync or Periodic Background Sync relay.
- No native app or app store work.
- No image, video, or large file relay.
- No offline mailbox on the server.
- No complex multi-hop routing.
- No general-purpose P2P mesh.
- No remote automatic reconnection in Phase 1.

## Modes

Only two modes are planned.

### Normal Mode

Default user mode.

- Communicates only while the app is open.
- Helps relay only while the app is open.
- Relay behavior is automatic, not a repeated permission prompt.
- Limits must stay small:
  - retention: 5 to 30 minutes
  - max items: 10 to 50
  - max storage: 1 to 10 MB
- Mobile network, low battery, or poor connection should reduce or stop relay work.
- Text messages only for early versions.

### Recommended Device Mode

For an unused PC, old phone, tablet, or home device left open.

- Still browser-based and best-effort only.
- More relay retention than normal mode.
- User should keep it charging, on Wi-Fi, screen on, and browser open.
- Must show simple state:
  - delivery help active/inactive
  - connection status
  - held message count
  - approximate storage use

## Recommended MVP Cut

### Phase 1: Face-To-Face QR Delivery

Implement only direct online 1-to-1 messaging after face-to-face QR exchange.

Components:

- Single HTML browser client
- QR-based manual signaling
- WebRTC DataChannel between two browsers
- Local device identity
- Application-layer E2EE
- Basic text chat UI

Flow:

1. Browser creates or loads a `deviceId` and ECDH key pair.
2. Host creates WebRTC offer and shows a QR code containing SDP plus public key.
3. Guest scans or pastes the host code.
4. Guest creates WebRTC answer and shows a QR code containing SDP plus public key.
5. Host scans or pastes the guest code.
6. Both sides derive a shared AES-GCM key.
7. DataChannel opens.
8. Text messages are encrypted and sent over DataChannel.

At this phase, do not add third-party relay, provider-based signaling, or multi-device routing.

### Phase 2: Remote Reconnection Provider

Add a swappable `SignalingProvider` for remote reconnection.

- The provider carries only handshake data and presence.
- It must not carry message bodies.
- First provider can be Firebase or a small signaling endpoint.
- Future provider can be Nostr or another relay.

Provider interface sketch:

```js
connect(deviceId)
publishPresence(publicKeyRef)
sendSignal(toFriendId, signal)
onSignal(callback)
disconnect()
```

### Phase 3: Short-Lived Third-Party Relay

After direct delivery and remote reconnection work, add best-effort relay.

Initial relay should be simple:

- A connected peer can accept encrypted envelopes for other peers.
- Relay stores only ciphertext envelope.
- Relay enforces TTL, count limit, and byte limit.
- Relay deletes expired or delivered envelopes.
- Start with in-memory storage.
- Add IndexedDB only after behavior is correct.

This phase should still assume the app works only while open.

### Phase 4: IndexedDB Queue

Use IndexedDB for local unsent messages and encrypted relay envelopes after direct E2EE chat works.

Store:

- encrypted envelope
- created time
- expiration time
- approximate byte size
- delivery attempts

Do not store:

- plaintext body
- unnecessary peer profile data
- unnecessary route history

Envelope draft:

```json
{
  "id": "uuid",
  "type": "message",
  "from": "device-id",
  "to": "device-id",
  "createdAt": 0,
  "expiresAt": 0,
  "contentType": "text/plain",
  "ciphertext": "base64"
}
```

The relay device must not need to know plaintext.

## WebRTC And Signaling

WebRTC DataChannel should be the main message transport.

Phase 1 signaling is QR/manual exchange. Remote reconnection still needs a signaling provider later because browser peers cannot receive inbound NAT connections by themselves.

The provider should be deliberately small:

- Track currently reachable device IDs.
- Relay offer, answer, and ICE candidates.
- Optionally expose presence.
- Never store or relay message bodies.

## Why Not WebSocket Message Delivery

## Security And Privacy Notes

- Relay devices handle encrypted envelopes only.
- Server does not store plaintext message bodies.
- Phase 1 has no server.
- Future provider still sees signaling metadata such as device IDs, online status, and connection timing.
- WebRTC can expose IP or network metadata during connection setup.
- STUN/TURN behavior must be reviewed before public deployment.
- TURN may be required for reliable connectivity, but it adds an operated relay path.
- Metadata minimization should be treated as a later hardening task, not a Phase 1 blocker.

## User Explanation

Use simple, non-scary wording.

Suggested text:

> While the app is open, this device can help pass along encrypted data. This device cannot read the contents.

Avoid repeated permission prompts. Provide transparency through:

- first-run explanation
- settings
- a short "how delivery works" page
- visible status in recommended device mode

## Small Implementation Tasks

1. Create single HTML app shell.
2. Generate and persist `deviceId`.
3. Generate and persist ECDH key pair.
4. Implement host offer QR.
5. Implement guest offer scan/paste.
6. Implement guest answer QR.
7. Implement host answer scan/paste.
8. Open DataChannel.
9. Derive shared AES-GCM key.
10. Encrypt and decrypt text messages.
11. Add text length limits.
12. Add connection status UI.
13. Add local unsent queue with IndexedDB.
14. Add `SignalingProvider` abstraction.
15. Add first remote provider.
16. Add in-memory relay queue with TTL.
17. Add normal mode limits.
18. Add recommended device mode limits and status.
19. Add IndexedDB persistence for encrypted relay envelopes.
20. Add risk notes and manual test checklist.

## First Stop Condition

Stop the first implementation slice when this works:

- Open two browser windows.
- Each window has a different device ID and public key.
- Host and guest exchange offer/answer by QR or paste.
- WebRTC DataChannel reaches `open`.
- A text message sent from one window appears in the other.
- The DataChannel payload is encrypted JSON, not plaintext.
- No local server is needed.
