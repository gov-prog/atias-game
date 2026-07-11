# אטיאס

Companion web app for "אטיאס" (Atias) — a physical-board party word game like Alias/Taboo, played by teams on
their own phones while a real board/dice sit on the table. Ported from the Claude Design prototype in
`../project/אטיאס.dc.html` (see `../chats/chat1.md` for the design history) into a React app.

## Run locally

```bash
npm install
npm run dev
```

## What it does

- Home → create a game (get a 4-character room code) or join one with a code.
- Lobby: host names their team, sets the turn length, reorders team turn-order, watches teams join live
  (with a sound + ambient music) — needs 2+ teams to start.
- Gameplay: the active team picks the board number their piece sits on, gets a flip card with 8 words (the word
  at their number highlighted), a countdown timer, ✓ correct / ⏭ skip. A shared "red circle" round variant is
  also available. Winner is whoever's board position passes the board length.

## Sync

Realtime sync is via Firebase Firestore (project `atias-game-42edb`, provisioned during the design session),
with localStorage/BroadcastChannel as a same-browser fallback. See `src/lib/firebase.js`.

**Note:** Firestore's security rules are currently in test mode (open read/write) and expire 2026-08-10 —
they'll need tightening or extending in the Firebase console before then.
