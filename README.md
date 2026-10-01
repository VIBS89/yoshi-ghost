# Yoshi Ghost

A retro browser-based endless side-scroller: pilot **Yoshi in a ghost bedsheet costume**, spit fireballs, blast lasers, and dodge your way between the haunted **Ghost Manor** and sunny **Yoshi Island**. Built as a single HTML file with vanilla JavaScript and the HTML5 Canvas API — all character, enemy, and world art generated with [Higgsfield](https://higgsfield.ai).

![Gameplay](img/gameplay-ghost-manor.png)

## How to Play

1. Open `index.html` in a browser (or serve via `python -m http.server 8080`)
2. Click, tap, or press **Space** to start and jump
3. Press **F** to spit a fireball and defeat enemies
4. Press **L** to fire the laser (20s recharge) and blast holes through pipes
5. Fly through **red gates** to activate Impossible Mode (reversed gravity for 10s)
6. Grab a floating **Ghost Mushroom** to phase through pipes and gates for 6s
7. Score 12 points to flip between **Ghost Manor** and **Yoshi Island**

## Controls

| Input | Action |
|-------|--------|
| Space / Click / Tap | Jump (or start/restart) |
| F | Spit fireball (defeats enemies, short cooldown) |
| L | Fire laser (blasts holes through pipes/gates) |

## Characters & Creatures

All sprites below are AI-generated (Higgsfield, flat-cartoon style) and rendered with a transparent-background fallback to the original procedural canvas drawing if an image ever fails to load.

<table>
<tr>
<td align="center" width="260">
<img src="assets/sprites/yoshi-ghost.png" width="220" alt="Yoshi Ghost"><br>
<b>Yoshi Ghost</b><br>
The player character — a Yoshi hiding under a bedsheet ghost costume, spitting fireballs across two haunted/tropical worlds.
</td>
<td align="center" width="260">
<img src="assets/sprites/enemy-ghost-manor.png" width="220" alt="Ghost Manor enemy"><br>
<b>Manor Bat-Ghost</b><br>
A spooky flying enemy native to Ghost Manor. Defeat it with a fireball for a point.
</td>
<td align="center" width="260">
<img src="assets/sprites/enemy-yoshi-island.png" width="220" alt="Yoshi Island enemy"><br>
<b>Island Sky Imp</b><br>
A masked imp riding a cloud puff, patrolling the skies of Yoshi Island.
</td>
</tr>
<tr>
<td align="center" width="260">
<img src="assets/sprites/ghost-mushroom.png" width="220" alt="Ghost Mushroom powerup"><br>
<b>Ghost Mushroom</b><br>
A rare powerup — collect it to phase through pipes and gates for 6 seconds (ground is still solid).
</td>
<td align="center" width="260">
<img src="assets/sprites/fireball.png" width="220" alt="Fireball projectile"><br>
<b>Fireball</b><br>
Yoshi's ranged attack, fired with <b>F</b>, defeats enemies on contact.
</td>
<td align="center" width="260">
<img src="assets/sprites/manor-ghost.png" width="220" alt="Ghost Manor background"><br>
<b>Ghost Manor</b><br>
World backdrop — a crooked haunted mansion with twisted trees and tombstones.
</td>
</tr>
</table>

<p align="center">
  <img src="assets/sprites/hill-island.png" width="420" alt="Yoshi Island background"><br>
  <sub>Yoshi Island — the second world, unlocked every 12 points</sub>
</p>

## Features

- Gravity-based physics with delta-time integration
- Two alternating worlds (Ghost Manor / Yoshi Island) that swap every 12 points, each with its own palette, pipe texture, clouds, and enemy
- Flying enemies defeated with fireballs (**F**)
- **Ghost Mushroom** powerup: temporary wall-phasing through pipes/gates, with a pulsing translucent glow on Yoshi while active
- Laser beam that blasts holes through pipes and gates (**L**)
- Red impossible gates that reverse gravity for 10 seconds — fixed to never re-stack/re-trigger from a second gate queued before the first is crossed
- Parallax cloud layers with varying opacity and speed, themed per world
- Score and high score persistence via localStorage
- Screen shake on collision, particle/defeat effects
- Velocity-based Yoshi tilt animation, idle bob on start screen
- Custom TTS-recreated Yoshi voice lines on level start and death
- Touch support for mobile
- 1.5x scaled canvas (720×960) for larger, clearer sprites and UI

## Project Structure

```
├── index.html              # Single-file game (HTML + CSS + JS)
├── game-config.json        # Physics parameters reference
├── assets/
│   ├── sprites/             # AI-generated game art (Higgsfield)
│   │   ├── yoshi-ghost.png     # Player character
│   │   ├── enemy-ghost-manor.png
│   │   ├── enemy-yoshi-island.png
│   │   ├── ghost-mushroom.png  # Wall-phasing powerup
│   │   ├── fireball.png
│   │   ├── cloud-ghost.png / cloud-island.png
│   │   ├── hill-island.png / manor-ghost.png   # World backdrops
│   │   └── pipe-ghost.png / pipe-island.png    # Pipe column textures
│   ├── yoshi_start.wav     # Level start voice line
│   ├── yoshi_death.wav     # Death voice line
│   ├── jump.wav            # Jump sound effect
│   ├── game_over.wav       # Legacy sound effect (unused, kept for reference)
│   └── ghosty.png          # Legacy sprite (unused, kept for reference)
├── img/
│   ├── gameplay-ghost-manor.png
│   └── gameplay-island-world.png
└── .kiro/
    ├── specs/flappy-kiro/
    │   ├── requirements.md
    │   ├── design.md
    │   └── tasks.md
    └── steering/
        ├── game-coding-standards.md
        ├── game-architecture.md
        ├── game-mechanics.md
        ├── canvas-and-collision.md
        └── visual-design.md
```

## CONFIG Object

All tunable constants live in a single `CONFIG` object exposed on `window.CONFIG` for live console tweaking. Values are scaled 1.5x from the original layout:

| Path | Default | Description |
|------|---------|--------------|
| `gravity` | 1200 | Downward acceleration (px/s²) |
| `jumpVelocity` | -450 | Upward velocity on jump (px/s) |
| `pipe.speed` | 180 | Horizontal pipe speed (px/s) |
| `pipe.gap` | 210 | Vertical gap between pipes (px) |
| `wall.spacing` | 525 | Distance between pipe spawns (px) |
| `enemy.hitboxScale` | 0.55 | Enemy collision box as a fraction of sprite size (centered) |
| `powerup.duration` | 6 | Ghost Mushroom wall-phasing duration (s) |
| `impossible.duration` | 10 | Reversed gravity duration (s) |
| `laser.rechargeTime` | 20 | Laser cooldown (s) |
| `laser.holeSize` | 90 | Size of laser holes (px) |

## Architecture

### Game State Machine

```mermaid
stateDiagram-v2
    [*] --> Start
    Start --> Playing : Space / Click / Tap
    Playing --> GameOver : Collision
    GameOver --> Playing : Space / Click / Tap (after 300ms)
```

### Game Loop (Playing State)

```mermaid
flowchart LR
    A[Process Input] --> B[Update Physics]
    B --> C[Spawn Pipes/Gates/Enemies/Powerups]
    C --> D[Move Entities]
    D --> E[Powerup Pickup → Phase Timer]
    E --> F[Check Collisions]
    F --> G[Update Score]
    G --> H[Update Timers]
    H --> I[World Switch Check]
    I --> J[Render Frame]
    J --> A
```

### System Modules

```mermaid
graph TD
    CONFIG[CONFIG Object] --> Physics
    CONFIG --> PipeManager
    CONFIG --> CloudManager
    CONFIG --> CollisionDetector
    CONFIG --> Renderer
    CONFIG --> LaserSystem
    CONFIG --> GateSystem
    CONFIG --> EnemySystem
    CONFIG --> PowerupSystem
    CONFIG --> FireballSystem
    CONFIG --> WorldSystem

    subgraph Core Logic
        Physics[Physics Engine<br/>updateKiro, jump]
        PipeManager[Pipe Manager<br/>spawnPipe, updatePipes, checkPipePass]
        CloudManager[Cloud Manager<br/>initClouds, updateClouds]
        CollisionDetector[Collision Detector<br/>checkCollision, isInHole, phasing]
        Scoring[Scoring<br/>formatScore, updateHighScore]
        GateSystem[Gate System<br/>spawnGate, updateGates, checkGatePass, hasPendingGate]
        LaserSystem[Laser System<br/>fireLaser, updateLaser]
        EnemySystem[Enemy System<br/>spawnEnemy, updateEnemies, checkEnemyCollision]
        PowerupSystem[Powerup System<br/>spawnPowerup, updatePowerups, checkPowerupCollision]
        FireballSystem[Fireball System<br/>fireFireball, updateFireballCollisions]
        WorldSystem[World System<br/>worldForScore, getWorldTheme]
    end

    subgraph Presentation
        Renderer[Renderer<br/>render]
        AudioManager[Audio Manager<br/>playSound]
        SpriteManager[Sprite Manager<br/>loadSprite, spriteReady]
    end

    subgraph Infrastructure
        InputHandler[Input Handler<br/>handleInput]
        GameLoop[Game Loop<br/>gameLoop, idleLoop]
        Persistence[Persistence<br/>loadHighScore, saveHighScore]
    end

    GameLoop --> Physics
    GameLoop --> PipeManager
    GameLoop --> CloudManager
    GameLoop --> CollisionDetector
    GameLoop --> Scoring
    GameLoop --> GateSystem
    GameLoop --> LaserSystem
    GameLoop --> EnemySystem
    GameLoop --> PowerupSystem
    GameLoop --> FireballSystem
    GameLoop --> WorldSystem
    GameLoop --> Renderer
    Renderer --> SpriteManager
    InputHandler --> GameLoop
    InputHandler --> LaserSystem
    InputHandler --> FireballSystem
```

### Collision Detection Flow

```mermaid
flowchart TD
    A[checkCollision] --> B{Ground?}
    B -->|Yes| C[Return true]
    B -->|No| P{Ghost Mushroom<br/>phasing active?}
    P -->|Yes| Z[Return false<br/>pipes/gates ignored]
    P -->|No| D{For each pipe}
    D --> E{AABB overlap<br/>with top pipe?}
    E -->|Yes| F{Inside a hole?}
    F -->|Yes| D
    F -->|No| C
    E -->|No| G{AABB overlap<br/>with bottom pipe?}
    G -->|Yes| F
    G -->|No| D
    D -->|Done| H{For each gate}
    H --> I{Same AABB + hole check}
    I -->|Collision| C
    I -->|No collision| H
    H -->|Done| J[Return false]
```

### Impossible Mode Flow

```mermaid
sequenceDiagram
    participant P as Player
    participant G as Game Loop
    participant Gate as Gate System
    participant Phys as Physics

    Note over Gate: Spawn guard: no new gate while<br/>one is still unpassed (hasPendingGate)
    P->>G: Fly through red gate gap
    G->>Gate: checkGatePass() → true
    Gate->>G: Activate impossible mode
    G->>G: gravityReversed = true
    G->>G: impossibleTimer = 10s
    loop Each Frame
        G->>Phys: updateKiro(kiro, dt, reversed=true)
        Phys->>Phys: Apply negative gravity
        G->>G: impossibleTimer -= dt
    end
    G->>G: Timer expires → gravityReversed = false
```

### Ghost Mushroom Powerup Flow

```mermaid
sequenceDiagram
    participant P as Player
    participant G as Game Loop
    participant Pow as Powerup System
    participant Col as Collision Detector

    P->>G: Overlap Ghost Mushroom sprite
    G->>Pow: checkPowerupCollision() → index
    Pow->>G: Remove powerup, ghostPhaseTimer = 6s
    loop Each Frame while timer > 0
        G->>Col: checkCollision(..., phasing=true)
        Col->>Col: Skip pipe/gate checks, ground still solid
        G->>G: Render pulsing translucent glow on Yoshi
        G->>G: ghostPhaseTimer -= dt
    end
    G->>G: Timer expires → normal collisions resume
```

### Laser System Flow

```mermaid
sequenceDiagram
    participant P as Player
    participant I as Input Handler
    participant L as Laser System
    participant Pipes as Pipes/Gates

    P->>I: Press L key
    I->>L: fireLaser(gameState)
    L->>L: Check laserReady
    alt Ready
        L->>L: laserReady = false
        L->>L: rechargeTimer = 20s
        L->>L: beamTimer = 0.3s
        L->>Pipes: Add holes at beam Y position
        loop Each Frame
            L->>L: rechargeTimer -= dt
        end
        L->>L: Timer expires → laserReady = true
    else Recharging
        L->>L: Ignore
    end
```

## Credits

Character, enemy, powerup, and background art generated with [Higgsfield](https://higgsfield.ai). Voice lines are an original TTS recreation, not sourced from any copyrighted game audio. Built with [Kiro](https://kiro.dev).

## License

See [LICENCE.md](LICENCE.md).
