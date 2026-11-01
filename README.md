# Waki Toki — Group Video & Push to Talk

A responsive browser-based, video-only group calling room. The app does not request microphone access or transmit audio.

## What's included

- Responsive dark video-call UI for mobile and desktop.
- Responsive participant video tiles, camera on/off, leave and invite-link copy.
- Video-only media capture (`audio: false`); no microphone permission is requested and no audio is transmitted.
- Firebase Realtime Database presence and room membership.
- WebRTC media calls through PeerJS. Only participants listed in the same Firebase room are considered; the app does not call arbitrary PeerJS peers.
- Anonymous Firebase Authentication; no user password or phone number is collected.

## Firebase setup (required before joining a room)

### Current Firebase project

The app is currently configured to use the existing `voter-slip-machine` Firebase project. Its web configuration is already in `index.html`; the Firebase Web API key is a client-side identifier, not an admin credential. Do not put service-account keys or database secrets in this file.

The app stores its presence under the separate `wakiTokiRooms/` path so it does not collide with the Voter Slip Machine's `rooms/` data.

1. In [Firebase Console](https://console.firebase.google.com/), select `voter-slip-machine`.
2. Open **Authentication → Sign-in method** and enable **Anonymous**.
3. Confirm **Realtime Database** is enabled and that the database URL in `index.html` matches the URL shown in the console.
4. Open **Realtime Database → Rules**. **Do not replace your entire existing rules file** because this Firebase project is also used by Voter Slip Machine. Merge the following `wakiTokiRooms` child into the existing top-level `rules` object, preserving all existing rules for other apps, then publish:

```json
"wakiTokiRooms": {
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
```

For example, the child belongs alongside your other top-level app paths inside `"rules": { ... }`; do not paste it as a replacement for the complete rules document. If your existing rules grant broader access at the root or at a parent path, those parent permissions can override these child restrictions, so review the combined rules carefully before publishing.

These starter rules let authenticated anonymous users read room presence and create/update/delete only their own participant record. Room codes are not strong authentication, and these rules do not provide private-room invitations or end-to-end encryption.

## Run / deploy

- Serve the repository over **HTTPS** (or localhost during development). Camera and microphone access are blocked on ordinary insecure HTTP origins.
- GitHub Pages can host the static page after Firebase is configured.
- Open the deployed page once: it automatically creates a new room, assigns a temporary `Guest-XXXX` name, and starts joining.
- The host taps **Invite** and shares the generated link. Anyone opening that exact link automatically joins the same room; nobody needs to type a name or room ID.
- The browser still asks each participant to allow camera/microphone access. If permission or connection setup fails, use the on-screen retry button.

## Important limitations before production

- This is an early browser prototype, not a WhatsApp replacement. PeerJS's public cloud signaling server is used by default; it is convenient for testing but has no production SLA.
- Media uses a peer-to-peer mesh. Every participant sends a separate stream to each other participant, so bandwidth and CPU use rise quickly. Keep test rooms small (about 2–4 people) until a proper SFU such as LiveKit is integrated. Failed initial peer connections are retried a few times, but this is not a substitute for a TURN server or an SFU.
- Some mobile networks and restrictive NAT/firewalls require a TURN relay. Add a trusted TURN service for reliable production connectivity.
- Firebase room reads are available to any authenticated user under these simple rules. For private production rooms, add room access tokens/invitations, stronger validation, rate limits and abuse controls.
- Camera permission is required to join a video call. If camera permission is denied or the camera is unavailable, joining fails; microphone access is never requested.
- The page currently supports video-only calls. It does not include microphone/audio, chat, call recording, screen sharing, end-to-end encryption, or native push notifications.

## Technology

HTML · CSS · JavaScript modules · Firebase Authentication · Firebase Realtime Database · PeerJS / WebRTC (video-only)
