# Waki Toki — Group Video & Push to Talk

A responsive browser-based group video room with push-to-talk and an active-speaker stage.

## What's included

- Responsive dark video-call UI for mobile and desktop.
- Dynamic speaker spotlight: one active speaker gets the wide focus tile; two speakers get equal tiles; three/four speakers get a balanced 2×2 layout. Other participants remain visible.
- Hold-to-talk microphone control (muted by default), camera on/off, mute, leave and invite-link copy.
- Firebase Realtime Database presence and room membership.
- WebRTC media calls through PeerJS. Only participants listed in the same Firebase room are considered; the app does not call arbitrary PeerJS peers.
- Anonymous Firebase Authentication; no user password or phone number is collected.

## Firebase setup (required before joining a room)

This repository intentionally does not contain a Firebase project's credentials. Configure your own Firebase project:

1. Open [Firebase Console](https://console.firebase.google.com/) and create/select a project.
2. Add a **Web app** in Project settings and copy its Firebase web configuration.
3. In **Authentication → Sign-in method**, enable **Anonymous**.
4. In **Realtime Database**, create a database and copy its exact database URL.
5. In `index.html`, replace the six `YOUR_...` values in `firebaseConfig` with your web app values. Set `databaseURL` to the exact URL shown by Firebase (it may be in a regional location).
6. In Realtime Database → Rules, use the rules below, then publish them:

```json
{
  "rules": {
    "rooms": {
      "$roomId": {
        ".read": "auth != null",
        "participants": {
          "$peerId": {
            ".write": "auth != null && (newData.child('uid').val() === auth.uid || data.child('uid').val() === auth.uid)",
            ".validate": "!newData.exists() || (newData.hasChildren(['uid', 'id', 'name']) && newData.child('uid').val() === auth.uid)"
          }
        }
      }
    }
  }
}
```

These rules let signed-in anonymous users read room presence and create/update/delete only their own participant record. They do not provide private-room invitations or end-to-end encryption. Treat room codes as discoverable identifiers, not as strong authentication.

## Run / deploy

- Serve the repository over **HTTPS** (or localhost during development). Camera and microphone access are blocked on ordinary insecure HTTP origins.
- GitHub Pages can host the static page after Firebase is configured.
- Open the deployed page on each device, enter a display name and the same room code, then select **रूम बनाएं / जुड़ें**.
- Share the invite link; it contains the room code in the URL.

## Important limitations before production

- This is an early browser prototype, not a WhatsApp replacement. PeerJS's public cloud signaling server is used by default; it is convenient for testing but has no production SLA.
- Media uses a peer-to-peer mesh. Every participant sends a separate stream to each other participant, so bandwidth and CPU use rise quickly. Keep test rooms small (about 2–4 people) until a proper SFU such as LiveKit is integrated.
- Some mobile networks and restrictive NAT/firewalls require a TURN relay. Add a trusted TURN service for reliable production connectivity.
- Firebase room reads are available to any authenticated user under these simple rules. For private production rooms, add room access tokens/invitations, stronger validation, rate limits and abuse controls.
- The page currently supports video and push-to-talk. It does not yet include chat, call recording, screen sharing, end-to-end encryption, or native push notifications.

## Technology

HTML · CSS · JavaScript modules · Firebase Authentication · Firebase Realtime Database · PeerJS / WebRTC
