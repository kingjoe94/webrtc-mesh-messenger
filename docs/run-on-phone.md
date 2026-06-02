# Run On Phone

## Important

Do not send `index.html` to a phone as a local file and expect the full app to work.

Many mobile browsers restrict these APIs when opened as a downloaded file:

- camera access for QR scanning
- Web Crypto
- WebRTC behavior
- loading CDN scripts

For the QR scanner flow, open the app as an HTTPS web page.

## Practical Options

### Option 1: Use Code Paste Instead Of Camera

This is the simplest no-hosting test.

1. Open `index.html` on two devices.
2. If QR scanning does not work, copy the generated code manually.
3. Paste the code into the other device's text area.
4. Repeat for the answer code.

This avoids camera access, but still depends on the browser allowing WebRTC and Web Crypto for the local file.

### Option 2: Host The Single HTML Over HTTPS

This is the recommended phone test.

Use any static HTTPS host:

- GitHub Pages
- Cloudflare Pages
- Netlify
- Vercel static hosting
- a private HTTPS server

The app is still a single HTML file. Hosting it does not mean message bodies go to the host. The host only serves the app file.

### Option 3: Local LAN Server

A plain local HTTP server is useful for desktop testing, but mobile camera APIs usually require HTTPS.

For phone camera testing, local LAN HTTP is usually not enough.

## Current MVP Assumption

Phase 1 is serverless for message delivery and signaling. It is not necessarily file-transfer-only.

The clean model is:

- HTML is served from an HTTPS page.
- QR codes carry the WebRTC offer/answer.
- Message bodies are sent over WebRTC DataChannel.
- Message bodies are encrypted before sending.
- No server stores or relays message bodies.
