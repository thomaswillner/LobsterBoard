# CackalackyCon MUD Project

A custom MUD (Multi-User Dungeon) for CackalackyCon using [GoMud](https://github.com/GoMudEngine/GoMud).

## Project Status

- [x] GoMud cloned to `/Users/richardcurry/clawd/GoMud`
- [x] Go 1.26.0 installed
- [x] Successfully built (`./gomud` binary)
- [x] Tested server startup — works!
- [ ] Design con-themed world
- [ ] Build custom zone
- [ ] Create NPCs, quests, items
- [ ] Test with players
- [ ] Deploy for con

## Quick Start

```bash
cd /Users/richardcurry/clawd/GoMud
./gomud

# Connect via:
# - Telnet: telnet localhost 33333
# - Web: http://localhost/webclient
# - Default login: admin / password
```

## Architecture Overview

GoMud is ~79k lines of Go with a JavaScript scripting engine (Goja).

### Key Directories

```
GoMud/
├── _datafiles/
│   └── world/
│       └── empty/           # Minimal starter world
│           ├── rooms/       # Room definitions (YAML + optional JS)
│           ├── mobs/        # NPCs and monsters
│           ├── quests/      # Quest chains
│           ├── items/       # Weapons, gear, loot
│           ├── conversations/ # NPC dialogue trees
│           ├── spells/      # Magic system
│           └── buffs/       # Status effects
├── internal/
│   ├── scripting/           # Goja JS engine
│   ├── rooms/               # Room management
│   ├── mobs/                # NPC AI
│   ├── quests/              # Quest system
│   └── web/                 # Built-in web client
└── gomud                    # Compiled binary
```

### Scripting System

Full JavaScript via Goja engine. Scripts can:
- Handle custom commands (`onCommand_search`, `onCommand_hack`)
- React to events (`onPlayerEnter`, `onDie`, `onLoad`)
- Spawn mobs/items dynamically
- Manipulate rooms (locks, exits, messages)
- Track persistent data per room/player

Example room script:
```javascript
function onCommand_hack(rest, user, room) {
    if (rest === "terminal") {
        user.SendText("You exploit a buffer overflow...");
        room.AddTemporaryExit("secret", "hidden passage", 1337, "1h");
        return true;
    }
    return false;
}
```

### Room Definition (YAML)

```yaml
roomid: 100
zone: CackalackyCon
title: Badge Pickup
description: The registration desk buzzes with activity. Volunteers hand out badges while attendees compare lanyards.
biome: indoor
exits:
  north:
    roomid: 101
  east:
    roomid: 102
spawninfo:
- mobid: 10
  message: A volunteer appears behind the desk.
  respawnrate: 5 real minutes
```

---

## Core Concept: Con → Fantasy Portal

**The Hook:** Players start at CackalackyCon, complete a short quest chain, then something goes wrong and they get transported INTO the existing GoMud fantasy world.

### Why This Works
- Leverage the full existing GoMud world (Frostfang, Dark Forest, Catacombs — 800+ rooms)
- Con zone is a custom "intro/tutorial" (~20-30 rooms)
- Memorable transition moment ties the hacker theme to fantasy adventure
- Fantasy NPCs can reference "the travelers from the glowing rectangles"
- Players get both experiences: con atmosphere + full MUD gameplay

### Portal Trigger Ideas

1. **The Badge Hack** — Your electronic badge glitches mid-hack. Screen flickers. You're IN the system now.
2. **CTF Gone Wrong** — That last exploit opened something it shouldn't have. Reality fragments.
3. **The Old Server** — Someone powered on a dusty machine. It was running something from 1994...
4. **VR Demo Malfunction** — Vendor booth VR headset won't come off. This isn't a demo.
5. **Energy Drink Incident** — That third Monster hit different. Way different.

### World Flow

```
┌─────────────────────────────────────┐
│      CACKALACKYCON ZONE             │
│      (~25-30 custom rooms)          │
│                                     │
│  Registration → Con Floor → CTF     │
│       ↓                             │
│  [Complete quest / trigger event]   │
│       ↓                             │
│  ═══ DRAMATIC TRANSITION ═══        │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      FROSTFANG CITY                 │
│      (Existing GoMud world)         │
│                                     │
│  800+ rooms, full quest system,     │
│  combat, NPCs, dungeons, etc.       │
└─────────────────────────────────────┘
```

### Room Map (CackalackyCon Zone)

```
                         ┌─────────────┐
                         │ Lockpick    │ (Unregistered)
                         │ Village     │
                         └──────┬──────┘
                                │
┌──────────┐  ┌──────────┐  ┌───┴───────┐  ┌──────────┐
│ Career   │──│ Workshop │──│  North    │──│ Badge    │ (Nutcrunch)
│ Corner   │  │ Hallway  │  │  Hallway  │  │ Hacking  │
└──────────┘  └──────────┘  └─────┬─────┘  └────┬─────┘
(Emwav)                           │              │
                                  │         ┌────┴─────┐
                         ┌────────┴───────┐ │ Hardware │ (s0lray)
                         │                │ │ Hacking  │
┌──────────┐             │   CON FLOOR    │ └──────────┘
│ AI Lab   │─────────────│     (Hub)      │
└──────────┘             │                │
(Threlfall)              └───┬───┬────┬───┘
                             │   │    │
              ┌──────────────┘   │    └──────────────┐
              │                  │                   │
        ┌─────┴─────┐     ┌──────┴──────┐     ┌──────┴─────┐
        │ Main Talk │     │ Registration│     │  LobbyCon  │
        │   Track   │     │             │     │            │
        └───────────┘     └──────┬──────┘     └──────┬─────┘
        (Curbob)          (Base16)                   │
                                 │              ┌────┴─────┐
                          ┌──────┴──────┐       │ Courtyard│
                          │   Lobby     │       │  & Pool  │
                          └──────┬──────┘       └──────────┘
                                 │
                          ┌──────┴──────┐
                          │   Main      │
                          │  Entrance   │  ← PLAYER START
                          └─────────────┘

        === SOUTH WING ===

┌──────────┐  ┌──────────┐  ┌───────────┐  ┌──────────┐
│  CTF     │──│ Wireless │──│  South    │──│   Bar    │ (Kiwi)
│  Arena   │  │ Shootout │  │  Hallway  │  │          │
└──────────┘  └──────────┘  └─────┬─────┘  └──────────┘
(Uncue)                           │
                          ┌───────┴───────┐
                          │  Restaurant   │
                          └───────────────┘

        === HIDDEN AREAS ===

┌──────────────┐      ┌─────────────────┐
│   Service    │ ───? │  Server Room    │  ← PORTAL TRIGGER
│   Hallway    │      │  (Hidden)       │
└──────────────┘      └─────────────────┘
  (off Con Floor)       (The Old Machine)


        === VILLAGE CLUSTER (off Con Floor) ===

┌──────────┐  ┌──────────┐  ┌──────────┐
│  Phreak  │──│ Village  │──│  Cipher  │
│   Me     │  │  Hall    │  │   Room   │
└──────────┘  └──────────┘  └──────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   ┌────┴───┐           ┌─────┴────┐
   │  SAV   │           │   IoT    │
   │ Lounge │           │Playground│
   └────────┘           └──────────┘

        === SOCIAL WING (off LobbyCon) ===

┌──────────┐  ┌──────────┐  ┌──────────┐
│ Karaoke  │──│  Social  │──│  Hacker  │
│ Lounge   │  │  Hall    │  │   Swan   │
└──────────┘  └──────────┘  └──────────┘
                   │
              ┌────┴─────┐
              │  Trivia  │
              │  Stage   │
              └──────────┘
```

### Room Count: ~27 rooms

**Core Flow (12):**
1. Main Entrance (START)
2. Lobby
3. Registration (Base16)
4. Con Floor (central hub)
5. Main Talk Track (Curbob)
6. LobbyCon
7. Courtyard & Pool
8. North Hallway
9. South Hallway
10. Workshop Hallway
11. Bar (Kiwi)
12. Restaurant

**Villages (11):**
13. Lockpick Village (Unregistered)
14. Career Corner (Emwav)
15. Badge Hacking (Nutcrunch)
16. Hardware Hacking (s0lray)
17. AI Lab (Threlfall)
18. CTF Arena (Uncue)
19. Wireless Shootout
20. Village Hall (sub-hub)
21. Phreak Me
22. Cipher Room
23. SAV Lounge / IoT Playground

**Social (4):**
24. Social Hall (sub-hub)
25. Karaoke Lounge
26. Trivia Stage
27. Hacker Swan Ballroom

**Hidden (2):**
28. Service Hallway
29. Server Room (PORTAL)

---

## Venue Reference

Single floor hotel layout. Map saved at: `~/.openclaw/media/inbound/9d83e46f-e6fc-45e9-b644-f58d9b47c823.png`

**Actual Venue Layout (for reference, not 1:1 in MUD):**

```
                    ┌─────────────────────────────────┐
                    │  Lockpicking    Oak/Dogwood     │
                    │  Workshop       Career Village  │
                    │         Elevator    Gym Area    │
    ┌───────┐       ├─────────────────────────────────┤
    │       │       │                                 │
    │  P    │       │    Main Talk      LobbyCon     │
    │  A    │ Svc   │    Track                       │
    │  R    │ Hall  │              Courtyard   Pool  │
    │  K    │       │    Registration                │
    │  I    │       │                                 │
    │  N    │       │    Chill Out / Camellia        │
    │  G    │       │    Hardware Hacking    Bar     │
    │       │       ├─────────────────────────────────┤
    └───────┘       │  Magnolia        Restaurant    │
                    │  CTF/Wireless    Lobby   HAM   │
                    │                  Porch         │
                    └──────── MAIN ENTRANCE ─────────┘
```

**Hidden Server Room Location Ideas:**
- Off the **Service Hallway** (maintenance access)
- Missing room number in **Gym area** (149... 147?... 145)
- Behind the **Pool equipment room**
- Basement access near **Elevator**

---

## CackalackyCon Zone Design

### Rooms to Build

**Main Areas:**
1. **Registration Hall** — Starting area, badge pickup
2. **Main Conference Floor** — Hub connecting to villages
3. **Server Room** — Hidden, requires exploits to access (PORTAL TRIGGER)

**Villages & Events (18 total!):**

| Room | Event | MUD Mechanic |
|------|-------|--------------|
| Lockpick Village | LPV | Use built-in lockpicking system! |
| Career Corner | Career | NPC job recruiters, resume drops |
| AI Lab | AI | NPCs that speak in ML gibberish |
| SAV Lounge | Social Attack Vector | Social engineering challenges, manipulate NPCs |
| IoT Playground | IoT village | Hackable "smart" objects |
| Phone Closet | Phreak Me | Rotary phones, blue boxes, 2600 Hz |
| Badge Bench | Badge Hacking | Solder items, modify badge stats |
| Cipher Room | Crypto | Encoded messages, decrypt for hints |
| CTF Arena | CTF | Combat-style flag captures |
| RF Range | Wireless Shootout | Signal-based puzzles |
| Karaoke Lounge | Karaoke | Jukebox, song commands, social |
| Guitar Workshop | Guitar Hacking | NPC "Pi Guitar Guy" - Raspberry Pi guitar mods, gives buff? |
| Trivia Stage | Trivia | NPC quizmaster, answer for rewards |
| Hacker Swan Ballroom | Hacker Swan | Fancy dress-up party, NPCs in formal wear, late-night vibes |
| Code Pit | Coding Challenge | Puzzle commands, logic gates |
| Quest Board | Quests | Central quest pickup location |
| Chaos Workshop | Chaos Workshops | Random crafting, explosions, fun |

*(HAM Exam removed)*

**Notable NPCs:**
- **Pi Guitar Guy** — Shows off his Raspberry Pi guitar, loops sounds, maybe gives a musical buff or unique item
- **Hacker Swan Attendees** — Fancy-dressed hackers at the formal party, good for humor/Easter eggs

### Hacker-Themed Mechanics

- **Buffer Overflow Doors** — Rooms that require specific command sequences to unlock
- **Hex-Speaking NPCs** — Decode their speech for hints
- **Lockpicking** — Already built into GoMud, works great for physical challenges
- **Root Access Quest** — Multi-step chain ending in admin powers
- **Hidden `/dev/null` Room** — Teleports you randomly
- **Badge Fragments** — Collectible quest items scattered around
- **SQL Injection Easter Eggs** — `'; DROP TABLE users;--` as a command does something fun

### NPCs

- **Con Volunteers** — Give hints, directions
- **Speakers** — Can be "asked" about their talks
- **Badge Hackers** — Trade items, give quests
- **Goons** — Security, might block certain areas
- **l33t h4x0r** — Speaks in leet, requires translation

### Quests

1. **Badge Hunter** — Find 5 hidden badge components
2. **Social Engineer** — Get 3 NPCs to reveal secrets
3. **Lockpick Legend** — Pick 10 locks
4. **Root Shell** — Exploit your way to the server room
5. **Party Animal** — Attend all after-hours events

### Items

- Electronic badge (equipable, gives stats)
- Lockpicks (consumable)
- USB rubber ducky
- WiFi pineapple
- Soldering iron
- Energy drinks (healing)
- Conference swag (collectibles)

---

## CackalackyCon Background

- **History:** Formerly CarolinaCon, 20 years between the two names
- **Location:** Durham, NC — Hotel near RDU Airport (4810 Page Creek Lane)
- **Date:** 2026 (specific dates TBD)
- **Theme:** 

### The Lime Lore 🍋‍🟩

**IMPORTANT INSIDE JOKE:** Limes are a thing at CackalackyCon.
- 2025 logo was an open-cut lime (LimeWire style)
- Work lime references into the MUD (NPCs mention limes, lime-themed items, etc.)
- Possible items: "Suspicious Lime", "Lime Wedge of Power", "The Sacred Citrus"
- Easter egg: typing `squeeze lime` does something fun

### 2026 Badge: Virus Transmission

Electronic badges this year will pass "viruses" between each other (badge viruses, not real malware).

**MUD tie-in ideas:**
- Player badges can get "infected" with buffs/debuffs
- Proximity to other players spreads effects
- Quest to find "Patient Zero" badge
- Antivirus item that clears badge debuffs
- Rare "beneficial virus" that gives stat boosts

### Known Staff (NPC candidates)

| Name | Role | NPC Idea |
|------|------|----------|
| **Emwav** | Career Village lead, résumé reviews | Career counselor NPC, gives job-related quests, backstory about becoming "world's okay-est hacker" |
| **Unregistered** | Lockpick Village | Lockpicking master NPC, teaches picking skills, gives lockpick-related quests, name is perfect for hacker con |
| **Curbob** | Main Talk Room | Always in the talks, never misses a session. Knows everything about the schedule. Ask him what's next. |
| **Uncue** | CTF Arena | CTF master, runs the competition, gives flag hints (for a price?), judges challenges |
| **sq33k** | EVERYWHERE | The Goddess. Keeps the con running. Wanders all rooms, fixes problems, knows everything. Special NPC with unique quests. |
| **Kiwi** | Bar | The bartender/bar regular. Serves drinks, trades rumors, always knows the gossip. Lime Rickeys on tap. |
| **Nutcrunch** | Badge Hacking Village | Badge wizard. Builds the electronic badges, can fix/mod yours, knows about the virus transmission system. |
| **Threlfall** | AI Lab | The AI whisperer. Talks about models, prompts, agents. May or may not be an AI himself. |
| **Base16** | Registration Desk | First face you see. Hands out badges, checks you in, knows where everything is. The gateway NPC. |
| **s0lray** | Hardware Hacking Village | Badge & hardware guru. Works with Nutcrunch. Soldering station master, PCB wizard. |
| **melvin2001** | Random/Wandering | Con regular, shows up places, chats |
| **Pandatrax** | Random/Wandering | Con regular, shows up places, chats |
| **Hoodoer** | Random/Wandering | Con regular, shows up places, chats |
| **Vic** | RARE - Dark corners | Mysterious figure in dark hoodie with "Vic" on back. Rarely seen. Knows things. Easter egg NPC. |

### Inside Jokes & Easter Eggs

- **Limes** — everywhere, subtle and not-so-subtle
- **"World's okay-est hacker"** — Emwav's self-description
- **Shadowrun / Hackers (1995)** — Emwav's origin story references
- *(add more as discovered)* 

---

## Technical Notes

### Ports

- Telnet: 33333
- HTTP/WebSocket: 80

### Config Override

Create a custom config to avoid modifying the original:
```bash
CONFIG_PATH=./cackalacky-config.yaml ./gomud
```

### Creating a New Zone

1. Create directory: `_datafiles/world/empty/rooms/cackalackycon/`
2. Add `zone-config.yaml`:
   ```yaml
   name: CackalackyCon
   roomid: 1000  # Starting room ID
   autoscale:
     minimum: 1
     maximum: 10
   defaultbiome: indoor
   ```
3. Add room files: `1000.yaml`, `1001.yaml`, etc.
4. Optional: Add scripts `1000.js` for custom behavior

### Useful Docs

- [GoMud GitHub](https://github.com/GoMudEngine/GoMud)
- [Scripting API](/Users/richardcurry/clawd/GoMud/internal/scripting/context.md)
- [Discord](https://discord.gg/cjukKvQWyy)
- [Guides](https://github.com/GoMudEngine/GoMud/blob/master/_datafiles/guides/README.md)

---

## Log

### 2026-03-04
- Cloned GoMud repo
- Installed Go 1.26.0 via Homebrew
- Built successfully, tested server startup
- Explored codebase: rooms, mobs, quests, scripting system
- Created this project doc
