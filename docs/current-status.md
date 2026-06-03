# Current Status

## Last Verified MVP Flow

Verified on 2026-06-02:

1. Host opens the GitHub Pages app.
2. Host taps `招待する`.
3. Guest scans the host invite QR with the phone's standard camera.
4. The invite URL opens the app on the guest device.
5. Guest automatically loads the `offer` from the URL.
6. Guest shows a reply QR.
7. Host taps `2. 返信を読む` and scans the guest reply QR in the app.
8. WebRTC DataChannel opens.
9. Text message delivery works.

Working app version:

```txt
2026-06-02.10-camera-offer-url
```

Current local app version:

```txt
2026-06-03.07-cleanup-errors
```

Current URL:

```txt
https://kingjoe94.github.io/webrtc-mesh-messenger/
```

## Current Architecture

- App hosting: GitHub Pages
- Invite QR: standard camera opens an app URL containing the compressed WebRTC offer
- Reply QR: host reads the compressed WebRTC answer with the in-app scanner
- QR libraries: vendored in `vendor/`
- Message transport: WebRTC DataChannel
- Message body server storage: none
- Signaling server: none in the current flow

## Confirmed Issue: Two QR Reads Are Cumbersome

The current no-signaling-server flow requires two manual exchanges:

1. Host offer QR
2. Guest answer QR

This is expected for serverless WebRTC because both sides must exchange connection descriptions. It works, but the UX is noticeably cumbersome.

Potential improvement:

- Add a minimal signaling provider that carries only `offer`, `answer`, and ICE candidates.
- Keep message bodies off the provider.
- Then the QR can contain only a short room URL or invite token.
- The guest opens the URL, and the answer can be returned through the signaling provider without a second QR scan.

Tradeoff:

- Much better UX.
- No need for the second QR.
- Requires a small external signaling component.

Decision update:

- Keep two-way QR exchange for first-time pairing.
- Do not use signaling to complete first-time friend addition from only one QR.
- The two-way QR exchange is important because this service is intended for people who actually meet in person.
- A one-QR invite that can be forwarded would weaken the "met in person" property.

## Confirmed Issue: Reload Requires Reconnection

If either side reloads or closes the page after connecting, the WebRTC peer connection is lost and the users must reconnect.

This is expected because:

- WebRTC connection state is in memory.
- SDP/ICE connection details are ephemeral.
- Browser pages cannot receive inbound connections after reload without a fresh handshake.

Potential improvement:

- Store friend identity and public key locally.
- Add a `SignalingProvider` only for reconnecting with already paired friends.
- On page load, rejoin presence and automatically attempt a fresh WebRTC handshake with known paired friends.

Tradeoff:

- Better recovery after reload.
- Requires a provider and friend/session state.
- Still does not store message bodies on the server.
- Must not allow first-time pairing through signaling alone.

## Recommended Next Step

Keep the current two-way QR flow as the first-time pairing baseline, then implement local friend persistence.

After local friend persistence works, prototype a minimal `SignalingProvider` as Phase 2 only for already paired friends.

The provider should only handle:

- reconnect requests between already paired friend IDs
- presence
- offer/answer/ICE exchange
- short-lived cleanup

The provider should not handle:

- plaintext message bodies
- encrypted message body storage for this phase
- user accounts
- public friend search
- first-time friend addition

## Next Implementation Order

Done:

1. Save paired friend records after the two-way QR exchange succeeds.
2. Show a simple friend list from local storage.
3. Allow manual reconnect to a saved friend through a targeted QR offer.
4. Add a `SignalingProvider` abstraction with the current QR exchange flow as the default provider.
5. Add Firebase provider configuration shell and setup notes.
6. Route saved-friend reconnect offer and answer through Firebase.
7. Confirm Firebase reconnect works between already paired devices.
8. Delete processed and expired Firebase signal documents from the receiving client.
9. Keep Firebase delete permission failures from repeating as signal processing failures.

Next:

1. Update Firestore Rules to allow signal deletion.
2. Add separate ICE candidate exchange if complete-SDP reconnect is unreliable.
3. Improve Firebase reconnect status messages.
4. Attempt reconnect only when both sides are open and already paired.

Friend record draft:

```json
{
  "friendId": "public-key-hash",
  "displayName": "optional local label",
  "publicJwk": {},
  "pairedAt": 0,
  "lastConnectedAt": 0
}
```
