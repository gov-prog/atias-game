# Handoff: אטיאס — Word-Guessing Party Game (Alias-style, companion app to a physical board)

## Overview
"אטיאס" is a mobile-first companion app for an Alias-style word-guessing party game. Players use a **real physical board and game piece** (not shown/modeled in the app beyond a fixed 72-space track) to track position; this app supplies:
- Room creation/join via a 4-character code, synced live across devices through Firebase.
- Turn management with configurable turn length and automatic turn order.
- Digital "cards" — 8 Hebrew words at a time, one highlighted as the current target — cycled through during a 1-minute (configurable) turn.
- A "red circle" shared/simultaneous round variant (see Interactions).
- Win detection once a team's cumulative advance reaches the board length (72 spaces, currently hardcoded).
- Sound effects (Web Audio synthesized, no audio files) and a lobby background ambient loop.

## About the Design Files
The bundled file (`אטיאס.dc.html`) is a **working HTML/JS prototype** — a real, functioning single-file app (not just a static mockup). It runs today as-is (open directly in a browser) and is fully wired to a live Firebase Firestore project for cross-device sync. The task for the receiving developer is to **port this into the target codebase's stack** (React Native / native iOS+Android / whatever the team standardizes on for the real product) using the target stack's own component and state-management patterns — this prototype is not meant to be shipped as literal production code, but every piece of game logic, copy, and interaction below should be preserved exactly.

## Fidelity
**High-fidelity.** Colors, typography, spacing, copy (Hebrew, RTL), and all interactions are final as designed. Recreate pixel-for-pixel where feasible; where platform conventions differ (e.g., native iOS vs. web), preserve the visual language (dark Nocturne base + warm Moroccan-night ornamental screens + gold parchment "cards") and the exact interaction/timing behavior.

## Screens / Views

### 1. Home
Full-bleed phone-shaped card (max-width 420px, ~760px tall, 32px corner radius) with a warm "Moroccan night" scene: layered radial gradients (warm amber glow top corners) over a dark indigo-to-black vertical gradient, a few small white star dots, and a flat dark "skyline" silhouette strip at the bottom (simple rectangles of varying height/width, 55% opacity).
- **Title cartouche**: cream (#f6ecd6) plaque, 2px gold (#b08d3f) border, 28px radius, 4 small gold diamond corner accents (8×8px squares rotated 45°). Title "אטיאס" in Frank Ruhl Libre, 900 weight, 52px, color #4a3016. Subtitle below: "משחק המילים לקבוצות — לוח פיזי, קלפים כאן", 13px, #8a6a2f.
- **Two glass buttons** (pill, 999px radius, 1px solid rgba(255,255,255,0.35) border, backdrop-filter blur 10px):
  - "יצירת משחק" — teal tint rgba(31,111,92,0.28), ➕ icon in a translucent white circle.
  - "הצטרפות למשחק" — maroon tint rgba(122,35,49,0.28), 🔑 icon.
- Footer note in a dark translucent pill: explains the room-code sync model.

### 2. Join
Simple centered form: back button, title "הצטרפות לקבוצה", large 4-character code input (32px, letter-spacing 8px, uppercase, center), "חיפוש קבוצה" button, inline error text if code not found.

### 3. Setup (name your team)
Shown to any device joining an existing lobby (not the host — the host names their team inline in the Lobby screen, see below). Title "איך קוראים לקבוצה שלכם?", subtitle explaining they'll be told their turn number, single text input, "הצטרפות ←" button (disabled until non-empty).

### 4. Lobby (merged create+waiting room)
This is the **single screen** shown immediately after "יצירת משחק" is pressed (no separate intermediate "create" screen) — the code, waiting list, and settings are all here:
- Large room code display (40px, 8px letter-spacing, accent color).
- Status line: host sees "שתפו את הקוד עם שאר המכשירים, סדרו את התור והתחילו כשמוכנים"; non-host sees "המספר ליד כל קבוצה הוא סדר התור — רק המארח יכול לשנות אותו".
- **Team list**: each row = order badge (number chip labeled "בתור" underneath, accent-colored text), color swatch, team name, "אתם" tag if it's you. If you are the host, ▲▼ reorder buttons appear on every row (drag-free, just move up/down one slot).
- **Host-only name input**: if the host hasn't joined a team yet, an inline "איך קוראים לקבוצה שלכם?" input + "הוספה" button appears (same underlying action as the Setup screen, just embedded here).
- **Host-only turn-length picker**: chip row, options 0:30 / 0:45 / 1:00 / 1:30 / 2:00, selected chip filled with accent color.
- **Host-only "🚀 התחלת המשחק"** button, disabled until ≥2 teams have joined.
- Board length is fixed at 72 (not user-editable) — see Design Tokens.
- **Sounds**: a rising 4-note arpeggio plays once when the room is created; a short two-note "ding" plays on every device whenever the team count increases (someone joined). A soft ambient pad loop (5-note pentatonic cycle, one note every 2.2s, very low gain) plays continuously while any device is sitting in the lobby, and stops the moment the game starts or the device leaves the lobby view.

### 5. Playing screen (shared shell)
Header bar: solid block in the active team's color, team name (white, 600 weight), "תור {position}/{boardLength}" on the right.

Sub-states (only one shown at a time, driven by `room.phase`):

**a. Picking (active team only)** — "איזה מספר על הלוח?" card with a 4×2 grid of 8 colored number buttons (1–8, each a distinct hue) representing the board-space number the team's piece is currently on, plus a full-width red "🔴 עיגול אדום — תור משותף" button for the shared/red-circle variant (see Interactions).

**b. Turn in progress (active team only)** — the countdown (34px, turns red + pulses under 10s) sits above a **flippable card**:
- Perspective 1400px 3D flip container (0.8s cubic-bezier transition), front face = word list, back face = branded "אטיאס!" card-back (dark teal/navy background, nested gold+teal ornamental borders, 4 small gold "✦" corner marks, a simple geometric fez-hat shape in orange atop a cream cartouche reading "אטיאס!").
- **Front face**: gold-gradient outer border (5px, `linear-gradient(135deg, #cfa049, #8a6a2f, #cfa049)`) around a cream (#f6ecd6) card with a 1.5px #b08d3f border, 4 small gold diamond corner accents, a title cartouche ("אטיאס!" in Frank Ruhl Libre), then a vertical list of 8 words each prefixed with a ★ star glyph — the current target word (matching the number chosen) is bold/larger (21px vs 16px), red-brown text (#8a2d1c) on a warm highlighted row background (#ece0c2), red star; the other 7 rows are smaller (16px), muted brown (#6b4a1e), transparent background. A footer line shows "מספר N • ✓ correct · ⏭ skip" tally.
- Two action buttons below the card: "⏭ דילוג" (outlined/secondary) and "✓ נכון!" (filled green, oklch(0.55 0.15 145)).
- **Sounds**: a bright 3-note ascending chime on ✓; a short flat 2-note blip on ⏭; a 3-beat low "dun-dun-dun" suspense cue at the exact halfway point of the timer; a 2-pulse low buzzer when time hits 0.
- On every ✓/⏭ tap: the card does a full 360° flip (imperceptible at the front, but passes through the branded back face at the midpoint) while new words are swapped in — the flip takes ~0.8s total, intentionally slow enough to read as a real card flip.

**c. Red round (active team only)** — dark red-tinted card ("🔴 עיגול אדום — מילה X מתוך 5"), one shared word shown large (32px, Frank Ruhl Libre), then a vertical list of buttons — one per team (including the reader's own) — "נקודה ל-{team name}", tinted in that team's color. Tapping one instantly awards that team +1 board position, checks for a win, and (unless the round's 5-word cap is reached) deals the next shared word. A "סיימתי" button ends the round early and passes the turn.

**d. Result (active team only)** — "הזמן נגמר!" then a large number = spaces to advance (max(correct − skip, 0)), tally line, "לתור הבא ←" button which advances `currentTeamIdx` and returns everyone to Picking for the next team.

**e. Spectating (every non-active device, including the host if not playing)** — a passive card: "התור של {team}", a status line ("בוחרים מספר...", "מסבירים כרגע", "🔴 עיגול אדום — תור משותף, הקשיבו!", or "מסכמים תור"), and — only while a turn is actively running — a live-updating countdown (synced from the active device roughly every 3 seconds, not per-second-precise).

Bottom of the Playing screen: a row of small dots, one per team, the active team's dot larger/fully opaque, others dim — a lightweight "whose turn" indicator.

### 6. Game Over
Shown to **every device** regardless of what view/phase they were on, the instant any team's position reaches the board length: 🏆 emoji, "מנצחים!", the winning team's name in their color (34px, Frank Ruhl Libre), "חזרה לדף הבית" button.

## Interactions & Behavior

### Room / device model
- A "room" is a single Firestore document at `rooms/{4-char code}` containing the entire game state (see State Management). Every device subscribes to that one document via `onSnapshot` and re-renders on every change — there is no separate per-team document.
- Every browser/device generates and persists a random device id in `localStorage` (`atias_device_id`) on first load. The room document's `hostDeviceId` field identifies the device that created the room; `team.claimedBy` identifies which device "owns" (plays as) each team.
- BroadcastChannel + localStorage mirror the same room document as an offline/same-browser fallback in case Firestore is unreachable — not required in the ported app if you have your own realtime backend, but keep *some* realtime layer, since the whole multi-device flow depends on it.

### Turn flow
1. Host creates room → picks turn length (and, implicitly, board length = 72) → shares the 4-char code.
2. Each other device enters the code, is shown the Setup screen, types a team name, and is appended to `room.teams` (order = join order, color assigned round-robin from an 8-color palette). The host can also add their own team from within the Lobby screen.
3. Host reorders teams (up/down) and can still change the turn length, any time before pressing Start.
4. Host presses Start → `room.phase` becomes `"picking"`, `room.currentTeamIdx = 0`.
5. The **current team's own device** (matched by `team.id === myTeamId`) sees the Picking screen; everyone else sees Spectating.
6. Picking a number 1–8 starts the per-turn countdown immediately (no separate "ready" step) and deals the first 8-word card.
7. Each ✓ increments a local `correctCount` and deals a new card; each ⏭ increments `skipCount` and deals a new card. Words are drawn without repetition until the pool (~230 built-in Hebrew words) is exhausted, then the exclusion list resets.
8. When the countdown hits 0: `advance = max(correct − skip, 0)` is added to the active team's cumulative `position`. If `position >= 72`, the room enters Game Over with that team as winner; otherwise `phase` becomes `"result"`.
9. Tapping "לתור הבא" advances `currentTeamIdx` (wrapping), resets to `"picking"`.
10. **Red round** is an alternative to picking a number: the active device can choose it instead. It deals one shared word at a time (no timer), and the active device's own player (not a countdown) manually taps which team answered it correctly first — that team's position +1 immediately, and a win check runs after every point. Ends after 5 words or an early "סיימתי" tap, then proceeds exactly like a normal result → next turn (no separate result screen for red rounds).

### Timing specifics
- Card flip: 3D `rotateY`, 0.8s `cubic-bezier(0.4,0.1,0.2,1)`, two separate absolutely-positioned faces with `backface-visibility: hidden` (not a 2D scale trick) — word content is swapped 420ms into the animation (roughly the midpoint, while the back face is showing).
- Countdown turns red and pulses (`scale 1 → 1.05`, 0.6s loop) in the final 10 seconds.
- Suspense stinger fires exactly once, at `Math.floor(duration/2)` seconds remaining.

## State Management
Single Firestore document per room, shape:
```
{
  code: "AH44",
  hostDeviceId: "<random id>",
  turnSeconds: 60,           // 30 | 45 | 60 | 90 | 120
  boardLength: 72,           // fixed
  teams: [
    { id, order, name, color, claimedBy, position }, ...
  ],
  currentTeamIdx: 0,
  phase: "lobby" | "picking" | "playing" | "redround" | "result" | "gameover",
  activeNumber: 1-8 | null,
  turnResult: { advance, correct, skip } | null,
  redWord: "...", redWordsUsed: 0, redMax: 5,   // only while phase === "redround"
  winnerTeamId: "..." | null,
  timeLeft: <int>,           // best-effort, throttled sync of the active device's countdown for spectators
  updatedAt: <epoch ms>
}
```
Local-only (per device, not synced) state: which team you are (`myTeamId`, persisted in `localStorage` per room code), whether you're a pure spectator, the currently drawn 8-word card, the countdown tick, the flip animation angle, and the used-words exclusion list for the active device's own card draws.

## Design Tokens

**Base system**: Nocturne (bound design system) — dark ground `--color-bg: #161826`, text `--color-text: #e9e9ed`, surfaces `--color-surface: #232532`, accent `--color-accent: #9184d9` (blurple), dividers `--color-divider`, radii `--radius-sm/md/lg` (4/8/14px), shadows `--shadow-sm/md/lg`, font Inter (`--font-heading`/`--font-body`). Used for all default UI chrome (buttons, inputs, cards, lobby, spectator screens).

**Deliberate exceptions** (documented, not accidental): team-identity colors and the number-picker grid use a hand-picked 8-hue OKLCH palette (not the mono Nocturne accent) because the game genuinely needs simultaneous, distinguishable team colors — this is the same category of exception as a chart/data-viz legend. The Home screen, the physical "card," and the card-back are a separate, intentionally warm "Moroccan night / gold parchment" motif layered on top of Nocturne, using:
- Gold: `#b08d3f` (borders/accents), `#cfa049` → `#8a6a2f` (card outer gradient), `#c9a227` (card-back accents).
- Parchment: `#f6ecd6` (card front + cartouches), text `#6b4a1e` / `#4a3016`.
- Target-word highlight: bg `#ece0c2`, text `#8a2d1c`.
- Card-back ground: `linear-gradient(160deg, #16303a, #0c1f27)`, teal border `#2f7a63`.
- Decorative display font: **Frank Ruhl Libre** (500/700/900) for all "אטיאס" wordmarks and the target word — loaded via Google Fonts alongside Inter.
- Team palette (8 OKLCH hues, ~L0.6 C0.15-0.18): red, green, blue, amber, magenta, cyan, orange, teal — same lightness/chroma family, hue-only variation.
- Home screen background: two soft warm radial glows (`rgba(255,176,90,.3/.26)`) over a `#1b2647 → #06090f` vertical gradient, plus a flat dark skyline silhouette strip.

## Assets
No image/photo assets — everything is CSS gradients, borders, and a handful of simple geometric shapes (rotated squares for diamonds, a rounded rectangle for the fez). No hand-drawn SVGs. Two emoji are used decoratively (🏆, 👀, 🔴, 🔑, ➕, ✦, ★, ✓, ⏭, 🚀) — replace with an icon set if the target platform's brand guidelines disallow emoji.

All sound is synthesized at runtime via the Web Audio API (`OscillatorNode` + `GainNode` envelopes) — there are no audio files to bundle. See the `playTone`/`playSuccess`/`playSkipSound`/`playTension`/`playTimeUp`/`playRoomCreated`/`playJoined` functions in the source for exact frequencies/durations/waveforms to reproduce natively.

## Firebase (reference backend used by the prototype)
The prototype syncs room state via a single Firestore collection `rooms`, one document per room code, using `setDoc` (full overwrite on every state change) and `onSnapshot` (realtime listener) — no subcollections, no auth, test-mode security rules (open read/write, time-boxed). This is almost certainly **not** how the production app should be built (no auth, whole-document overwrites are a race-condition risk with >2 simultaneous writers) — treat it as a proof of the realtime-sync requirement only. A real implementation should use per-field updates or transactions (especially for `awardRedPoint`/`finishTurn`, which read-modify-write the teams array) and proper auth/security rules.

## Files
- `אטיאס.dc.html` — the complete working prototype (template + logic + Firebase wiring in one file). Open directly in any browser to run it.
