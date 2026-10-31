# BrowserBabyAlarm

A baby monitor that runs entirely in the browser. Put one phone in the baby's room, watch and listen from another phone or a computer. No app install, no account, no server of your own.

- Live **video and audio**, sent directly between your devices with WebRTC (via [PeerJS](https://peerjs.com))
- **Cry alarm** that triggers on sustained sound, gets louder until dismissed, and vibrates / notifies
- Up to **2 parents at once**, with a "Watching: N" badge, and **saved devices** so you only pair once
- **Remote control** of the baby phone: nightlight, black screen, flip camera, camera on/off and mic on/off (battery saving), sleep sounds, push-to-talk
- **Sleep sounds** generated in the browser: womb noise, rain, ocean waves, fireplace, lullaby
- **Connection badge** showing whether the video travels over your local network, the internet, or a relay
- Disconnect alarm, low-battery alert, Danish and English UI, installable as a PWA

> This is a hobby project, not a certified safety device. Browsers and phone operating systems can suspend pages, lose Wi-Fi or run out of battery. Do not rely on it as your only way of monitoring a baby.

## Quick start (GitHub Pages)

Camera and microphone access require **HTTPS** (or `localhost`). The easiest way to get HTTPS is GitHub Pages:

1. Fork or upload this repository to your GitHub account.
2. **Settings → Pages →** deploy from the `main` branch, root folder.
3. Open `https://<your-user>.github.io/<repo>/` on both devices.

**Baby phone:** tap *Baby device* → *Grant access*. A QR code and a code appear.
**Parent device:** tap *Parent device*, scan the QR code or type the code, and optionally give the device a name.

From then on, the parent device reconnects by itself when you open the app.

## Using it

| Feature | How |
| --- | --- |
| Pair another parent | On the baby phone press **Pair new device** (the pairing window is open for 5 minutes and closes after one device has paired) |
| Remove a device | Baby phone → *Allowed devices* → **Remove** |
| Revoke everyone | Baby phone → **New code** (also changes the code) |
| Cry threshold | Parent screen slider; the red line on the live level bar is the threshold. Alarm = sustained sound above it (about 0.6 s within 2 s) |
| Save battery | Parent: *Video on/off* turns the baby phone's camera off. *Microphone on/off* only mutes (and disables the cry alarm) |
| Keep the baby phone awake | Keep it plugged in with the page open. Use **Black screen** rather than the lock button |

## How it works

```
 Baby phone ──(signalling only)──► PeerJS cloud server ◄──(signalling only)── Parent
      │                                                                          │
      └──────────────── encrypted WebRTC media + data (direct) ─────────────────┘
```

- The video and audio are end-to-end encrypted by WebRTC and go **directly** between the devices. No server stores or sees them.
- A PeerJS cloud server (`0.peerjs.com`) and Google STUN servers are used to set up the connection. They can see IP addresses and connection metadata, but not the stream.
- If the devices cannot reach each other directly, a **TURN relay** is needed (see below). Media is still encrypted.
- The parent screen shows which route is used: *Local network*, *Internet (direct)* or *Relay (TURN)*.

### Security model

- The baby phone's code is a random 10-character secret that forms its peer ID.
- A device can only connect if it holds a **private token** issued by the baby phone, or while the **pairing window** is open. Tokens are stored in the browser's `localStorage` on both sides.
- At most 2 parents are connected at the same time; at most 4 devices are remembered.
- Calls and commands are only accepted from devices that authenticated first; remote commands are checked against a whitelist.
- A strict Content-Security-Policy limits scripts to this site and connections to the PeerJS server. Libraries are self-hosted in `lib/`.

See [SECURITY.md](SECURITY.md) for details and how to report problems.

## Network notes

- **Same Wi-Fi is not always enough.** Guest networks and routers with *client / AP isolation* block phone-to-phone traffic. VPN and device-management apps (for example mobile-security or work-profile apps) can block WebRTC too.
- If the parent's badge stays on `ICE: checking` or `failed`, add a **TURN server** to the `ICE` array near the top of the script in `index.html`:

  ```js
  const ICE=[{urls:'stun:stun.l.google.com:19302'},
    {urls:'turn:turn.example.com:3478',username:'USER',credential:'PASS'}];
  ```

  Anything in client-side code is visible to visitors, so use a limited / time-limited TURN account (or a private repository).
- **Fully local setup:** self-host a PeerJS server, remove the Google STUN entries, serve the page over HTTPS on your own network, and update `connect-src` in the CSP meta tag.

## Troubleshooting

| Symptom | Likely cause |
| --- | --- |
| Camera / microphone permission never appears | Page is not served over HTTPS |
| Black video, `ICE: checking` forever | Client isolation, VPN / security app, or a network that needs TURN |
| Video works on a computer but not between phones | Same as above; try both on the same non-guest Wi-Fi |
| No sound on the parent device | Tap the *Tap to enable audio* button (browser autoplay rules) |
| Cry alarm never fires / fires too often | Adjust the threshold slider using the live level bar; sleep sounds near the baby phone's microphone raise the level |
| "Pairing is closed" | Press **Pair new device** on the baby phone |
| Nothing connects after uploading | Check that all files, including `lib/`, were uploaded, and the CSP `connect-src` matches your signalling server |

## Development

There is no build step. Serve the folder and open it:

```bash
python3 -m http.server 8080      # then open http://localhost:8080
```

`localhost` counts as a secure context, so the camera works on the same machine. For two real devices you need HTTPS (GitHub Pages, or a local HTTPS server).

```
index.html            the whole app (HTML, CSS, JavaScript)
lib/                  PeerJS 1.5.4 and qrcodejs 1.0.0 (self-hosted) + their licenses
sw.js                 network-first service worker (offline start of the page)
manifest.webmanifest  PWA manifest
icon-192.png, icon-512.png
```

## Browser support

Current Chrome, Edge, Firefox and Safari (iOS 16.4+ recommended) with WebRTC, `getUserMedia` and Web Audio. Features that depend on browser APIs degrade gracefully: the Battery API (baby phone battery falls back to a manual estimate) and the Wake Lock API (keep the screen on yourself if it is missing).

## License

[MIT](LICENSE). Bundled third-party libraries keep their own licenses, see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
