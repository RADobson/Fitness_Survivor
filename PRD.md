# PRD.md — Fitness Survivor (App Version)

## 1) Summary

Build an **installable app** version (mobile-first web app / PWA) of the boardgame **Fitness Survivor**, including **online multiplayer**.

Core loop:
- 2–4 players
- On your turn you may play **one sabotage card before rolling**
- Roll **2d6** to move around a **40-square loop**
- The **square number** determines the reps/seconds for the exercise on that square
- Passing **GO** awards a sabotage card
- If you fail your exercise, you are **OUT**
- Last player standing **wins**

This PRD targets an MVP that matches the existing “Island Loop Alpha” behavior and feel, but as a clean, maintainable app.

Reference: an alpha HTML prototype exists (provided by the user). The app should preserve the same rules, UI intent, and island-board vibe.

---

## 2) Initial Goals (MVP)

### Gameplay + UX
- Mobile-first UI that works great on phones, tablets, and desktop
- Smooth turn flow: setup → card play → roll → move animation → challenge → done/fail
- Clear visuals:
  - island board with 40 numbered squares
  - player tokens on the path
  - dice display and roll animation
  - player list with OUT/IN status and effects
  - hand of sabotage cards + target selection
- Installable as PWA (Add to Home Screen) **if feasible**

### Game Modes (MVP must support both)
- **Local “couch” play:** pass the device around
- **Online multiplayer:** players join from different devices via a room code/link

---

## 3) Next Goals (Post-MVP)

- Accounts (email/social login), persistent profiles, avatars
- Cloud sync (resume games across devices)
- Matchmaking / public lobbies
- Leaderboards / stats
- Higher-fidelity art pipeline (custom board art, animation polish)
- Larger card deck / expansions
- Rich audio/music system (MVP only needs simple SFX)

---

## 4) Platforms

- Primary: Mobile browsers (iOS Safari, Android Chrome)
- Secondary: Desktop browsers
- Optional: PWA install support (offline-capable shell for local mode)

---

## 5) User Stories

### Setup (Local)
- As a player, I can select player count (2–4) and enter names
- As a player, I can start a new local game and see everyone at GO

### Setup (Online)
- As a player, I can **create an online room** and receive a join link/code
- As a player, I can **join a room** by entering a code or opening a link
- As a room host, I can see who joined and start the game when ready
- As a player, I can pick a display name and see which seat/token color I am

### Gameplay (Local + Online)
- As a player, I can see whose turn it is
- As a player, I can (optionally) play **one sabotage card before rolling**
- As a player, I can roll 2d6 and watch my token move
- As a player, I see the challenge (exercise + amount) for my landing square
- As a player, I can mark **Done** or **Fail**
- As a group, we can see who is OUT and who remains
- As a group, the app declares a winner when one player remains

### Cards & Effects
- As a player, I receive a sabotage card when passing GO
- As a player, I can target an opponent when a card requires it
- As a player, I can see active effects on each player
- As a player, I can see effects applied to my next challenge and then consumed

### Online Reliability
- As a player, if my connection drops briefly, I can **reconnect** and continue
- As a group, we can see when someone disconnects/reconnects
- As a group, the game does not desync between clients

### UX / Quality
- As a user, I can toggle sound on/off (local device preference)
- As a user, I can view a game log of what happened
- As a user, I see a safety warning/disclaimer

---

## 6) Game Rules (MVP)

### Board
- Loop of **40 squares** numbered 1–40
- Square 1 is GO
- Landing on a square triggers its exercise (except GO)

### Dice & Movement
- Movement is only by **2d6**
- Token moves forward by sum
- If the move crosses GO (wrap-around), the player **draws one sabotage card**

### Challenges
- The required amount is the **square number**
  - e.g., Square 23 => 23 reps (or 23 seconds for plank)
- “Plank” uses seconds; most others use reps
- Completing the challenge ends your turn (tap Done)
- Failing eliminates you (tap Fail)

### Sabotage Cards (MVP deck)
MVP can use the same cards as alpha:
- **Double**: target does DOUBLE next square
- **Freeze**: target must freeze 10s before next exercise
- **Skip**: target skips next turn
- **+8**: target gets +8 reps/seconds next square
- **Relief**: halve your next square (round up)

Rules:
- You may play **one card per turn**, and only **before rolling**
- Effects are stored on the player and consumed when applied
- Skip is consumed at the start of the target’s next turn

---

## 7) Functional Requirements

## 7.1 Screens / Panels

### Home
- Choose mode: **Local** or **Online**
- Quick rules blurb + safety disclaimer access

### Setup (Local)
- Player count selector (2–4)
- Names input (comma separated or per-field)
- Start game button
- Sound toggle
- Display safety disclaimer

### Online Lobby
- Create room (host)
- Join room (code/link)
- Lobby shows:
  - room code/link
  - player list (names, connected status)
  - ready indicators (optional)
  - host “Start Game” button (enabled only when 2–4 players)

### Game Screen (Local + Online)
#### Turn Panel
- Current player indicator
- Card hand for current player
- Card select + target select + “Play selected card”
- Roll button (disabled when not allowed)
- Dice UI (two dice) + roll total
- Done / Fail buttons (only enabled after a roll)
- Challenge display (exercise, icon, amount, unit)
- Optional timer UI for Freeze (countdown)

#### Board
- Island board (SVG or canvas) with:
  - track path loop
  - 40 tiles placed along path
  - tile number visible on each tile
  - exercise icon/label (or icon only) on each tile
  - exercise assignment randomized **at the start of each game** and fixed for that game

#### Players
- List all players showing:
  - IN/OUT
  - current square position
  - active effects
  - whose turn

#### Log
- Append-only log of key events:
  - game start
  - joins/leaves/reconnects (online)
  - card draws
  - card plays
  - dice rolls
  - passing GO
  - challenges completed/failed
  - eliminations
  - winner

---

## 7.2 Online Multiplayer Requirements (MVP)

### Networking Model
- **Server authoritative** for all game state changes.
  - Clients send **intent** (“play card X on Y”, “roll”, “done”, “fail”)
  - Server validates rules and broadcasts updates
- Server controls:
  - dice results
  - card draws
  - square exercise assignment randomization
  - turn order
  - effect application/consumption

### Realtime Transport
- WebSocket-based realtime updates (or equivalent realtime channel).
- Messages must include:
  - room id
  - player id (session token)
  - client sequence id (idempotency)
  - server timestamp/turn counter

### Room + Identity (MVP)
- No accounts required.
- Joining creates a **player session token** stored locally (so refresh/rejoin works).
- Room join via:
  - short code (e.g., 6–8 chars) **and/or**
  - shareable link containing room code

### Reconnect / Dropouts (MVP)
- If a player disconnects:
  - mark them disconnected in lobby/game UI
  - allow rejoin within a grace period (e.g., 10 minutes) using their token
- If current player disconnects during their turn:
  - MVP option A: pause turn until they reconnect
  - MVP option B: allow others to vote “eliminate” or “skip” (NOT MVP unless requested)
  - MVP choice: **pause turn** (simplest)

### Turn Enforcement
- Only the current player can act.
- Exactly one sabotage card can be played per turn, before rolling.
- Server rejects invalid actions and returns a user-friendly error.

### Security / Abuse (MVP)
- Never trust client for RNG or state.
- Validate all actions server-side.
- Basic rate limiting per connection to prevent spam.

---

## 7.3 State & Phase Model

Game state machine with phases:
- setup → lobby (online) → preRoll → animatingMove → challenge → turnEnd → gameOver

Notes:
- “animatingMove” can be client-side animation driven from server events
- The canonical state is the server snapshot; clients render from snapshot + events

---

## 8) Non-Functional Requirements

- Responsive layout; works at 320px wide
- No jank: keep animations smooth on mobile
- Accessibility:
  - readable text sizes
  - clear button states
  - color + label cues (not color-only)
- Online multiplayer:
  - tolerate normal mobile latency without desync
  - reconnect support
  - stable room cleanup (inactive rooms expire)

---

## 9) Acceptance Criteria (MVP)

### Local Mode
- Can start a game with 2–4 players and names
- Can roll 2d6 and move token around a 40-square loop
- Passing GO draws exactly 1 card
- Can play 1 sabotage card before rolling; correct targeting rules
- Effects modify next challenge correctly and are consumed correctly
- Done ends turn; Fail eliminates player
- Winner declared when 1 player remains
- Game log records key events

### Online Mode
- Host can create a room and share a join code/link
- 2–4 players can join from separate devices and see the same lobby
- Host can start the game and all clients enter the same game state
- Actions are server-authoritative:
  - only current player can roll/play/done/fail
  - dice/card draws/exercise assignments match on all clients
- Disconnect/reconnect works:
  - a player can refresh and rejoin the same seat within the grace window
  - the game state remains consistent across clients

### Build/Run
- Docker (if present) can rebuild/run successfully and the app loads
- Provide “How to run” instructions in README (local + online dev)

---

## 10) Implementation Notes (Guidance for Codex)

- Keep the core game logic in a small “engine” module (pure functions where possible)
- UI should be a thin layer on top of game state
- Prefer a single source of truth for state (store + reducer/state machine)
- Online: server owns canonical state; clients render from server snapshots/events
- Randomization must be stable:
  - server picks square exercise assignment at game start (seeded RNG is acceptable)
- Keep asset usage minimal for MVP (emoji/icons acceptable)
- Do not over-engineer: MVP first, then polish

---

## 11) Task List (Ralphy)

### Epic A — Project Setup
- [ ] Detect existing project structure; if none exists, scaffold a mobile-first web app project
- [ ] Add a basic README with run instructions and a short description
- [ ] Add a simple dev script and (if applicable) docker-compose services for:
  - [ ] web client
  - [ ] multiplayer server

### Epic B — Game Engine (Shared Logic)
- [ ] Implement game state types: players, squares, deck, effects, turn index, phase
- [ ] Implement square generation (40 squares; GO at 1; randomized exercise mapping per game)
- [ ] Implement dice roll + move logic (wrap detection + pass GO card draw)
- [ ] Implement card system (draw, play, targeting rules, effect application)
- [ ] Implement effect consumption rules (skip at turn start; others at challenge calc)
- [ ] Implement win condition and elimination logic
- [ ] Implement event log entries for all major actions

### Epic C — Multiplayer Server (Authoritative)
- [ ] Create room lifecycle:
  - [ ] create room → join room → leave → expire
- [ ] Create session identity:
  - [ ] issue player token on join
  - [ ] allow rejoin with token
- [ ] Implement authoritative game actions:
  - [ ] start game (host)
  - [ ] play card
  - [ ] roll
  - [ ] done/fail
- [ ] Server-side validation for all rules + friendly error responses
- [ ] Broadcast state updates/events to all clients
- [ ] Implement disconnect/reconnect + grace window
- [ ] Basic rate limiting

### Epic D — Board Rendering
- [ ] Port/replicate the island board look (SVG-based MVP is fine)
- [ ] Render track path and place 40 tiles along it (stable positions)
- [ ] Render tile numbers and icons/labels
- [ ] Render tokens per player with offset stacking on same square
- [ ] Add token highlight for current player

### Epic E — UI (Local + Online)
- [ ] Home screen (choose Local vs Online)
- [ ] Local Setup panel
- [ ] Online Lobby screen (create/join, player list, start)
- [ ] Game screen with:
  - [ ] Turn panel (buttons enabled/disabled correctly by phase + authority)
  - [ ] Cards UI (hand, select, target, play)
  - [ ] Players panel (effects, IN/OUT, turn)
  - [ ] Log panel (scrollable)
- [ ] Sound toggle (device-local preference)

### Epic F — Animations & Sound (MVP polish)
- [ ] Dice roll animation (simple, non-janky)
- [ ] Token move animation along path driven by server events
- [ ] Simple sound effects + toggle (safe defaults)

### Epic G — Validation & Tests
- [ ] Add automated tests for core engine rules:
  - [ ] pass GO draws card
  - [ ] skip consumes at turn start
  - [ ] double/+8/relief/freeze modify next challenge and consume
  - [ ] fail eliminates and winner detection
- [ ] Add multiplayer server tests (minimum):
  - [ ] only current player can act
  - [ ] server rejects invalid actions
  - [ ] reconnect restores seat/state
- [ ] Add a smoke test path (manual steps in README)
- [ ] Ensure builds/runs cleanly via the repo’s standard workflow

### Epic H — Optional PWA
- [ ] Add PWA manifest + installability
- [ ] Ensure local mode works offline after first load (online mode requires network)

### Epic I — Finishing
- [ ] Confirm all Acceptance Criteria met
- [ ] Update PRD checkboxes as completed
- [ ] Final README “How to Play” + “How to Host/Join Online” sections

---

## 12) Manual Test Script (for humans)

### Local
1) Start local game with 4 players
2) On Player 1:
   - play a card (if any) before rolling (or skip)
   - roll 2d6, confirm token moves and log updates
   - mark Done, confirm next player turn
3) Force a pass-GO scenario:
   - confirm “pass GO → draw card” happens exactly once per crossing
4) Apply each sabotage card at least once and confirm effect behavior
5) Eliminate players with Fail until winner appears

### Online
1) Device A: create room, note code/link
2) Devices B/C/D: join room via code/link, confirm lobby shows all players
3) Host starts game; all devices load the same board state
4) Current player rolls; all devices see same dice + movement + challenge
5) Current player plays a card before rolling next turn; verify server-enforced rules
6) Simulate disconnect:
   - refresh Device B browser
   - confirm it reconnects and restores same seat/state within grace window
7) Finish game and confirm winner appears on all devices

---

## 13) Safety Disclaimer (display in UI)

“Warm up. Stop if you feel pain, dizziness, or discomfort. This app is not
medical advice. You are responsible for exercising safely.”

