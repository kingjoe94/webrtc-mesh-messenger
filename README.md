# WebRTC Mesh Messenger

Browser-only WebRTC DataChannel messaging MVP.

## Current MVP

- Single `index.html`
- Face-to-face QR/manual signaling
- 1-to-1 text chat
- ECDH key exchange
- AES-GCM application-layer encryption before DataChannel send
- No message body storage on a server

## Phone Test

Open the app through HTTPS, for example GitHub Pages.

Sending `index.html` to a phone as a local file is not enough for reliable QR scanning because mobile browsers often restrict camera access, Web Crypto, and related APIs for local files.

## Notes

This is an early proof of concept. It is not production-ready security software.

## Firebase Signaling Setup

See `docs/firebase-setup.md` for the optional Firestore signaling provider setup.
