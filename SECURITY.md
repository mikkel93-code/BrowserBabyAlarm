# Security policy

BrowserBabyAlarm streams video and audio from a baby's room, so security reports are taken seriously.

## Reporting a vulnerability

Please do **not** open a public issue for security problems. Use GitHub's private vulnerability reporting
(*Security → Report a vulnerability*) on this repository, or contact the maintainer directly.

## Design summary

- Media and data travel directly between devices over WebRTC (DTLS-SRTP encrypted). No media server is involved unless you configure a TURN relay, which only forwards encrypted packets.
- The PeerJS cloud server and STUN servers see IP addresses and connection metadata, not the stream.
- A device connects only with a private token issued by the baby phone, or during a short pairing window opened on the baby phone. At most 2 parents watch at once.
- Content-Security-Policy restricts scripts to this origin and connections to the PeerJS server; libraries are self-hosted.

## Known limitations

- Pairing codes, tokens and device names are stored in the browser's `localStorage`. Anyone with access to an unlocked paired device can watch the stream.
- Anything in `index.html` (including a TURN username/password) is visible to everyone who can load the page. Use limited or time-limited TURN credentials.
- The public PeerJS signalling server is a third party. Self-host it if you need full control.
