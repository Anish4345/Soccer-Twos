# Soccer Twos — 3D Multi-Agent Simulation

A **2v2 multi-agent soccer simulation** built entirely in Python using `pygame`.  
Inspired by the [Unity ML-Agents Soccer Twos](https://github.com/Unity-Technologies/ml-agents) environment, this project recreates the game logic and renders it in a fully interactive **3D perspective view** — with zero OpenGL or external 3D libraries.

---

## Project Overview

| Property | Details |
|---|---|
| **Type** | Rule-Based Multi-Agent Simulation |
| **Environment** | 2v2 Soccer (Team Alpha vs Team Beta) |
| **Rendering** | Hand-rolled painter's-algorithm 3D → 2D projection |
| **Dependency** | `pygame` only |
| **Platform** | Python 3.x, Jupyter Notebook / `.py` script |

This project was built as part of a **Final Round Task Assignment** that required:
- Creating a game environment modelled after the Unity ML-Agents Soccer Twos spec
- Developing agents that can play the game autonomously

---

## Game Rules

- **Teams:** Team Alpha (green) vs Team Beta (red) — 2 agents per team
- **Goal:** Get the ball into the opponent's goal; defend your own
- **Episode length:** 1200 steps per episode
- **Scoring:** Goals are tracked per episode; wins, losses, and draws are recorded across episodes
- **Reset:** Ball and agents reset to starting positions after every goal or episode end

---

## Agent Design

Agents use **rule-based AI** (not reinforcement learning). Each agent has a role and a simple decision loop:

### Roles
| Role | Behaviour |
|---|---|
| **Striker** | Always chases the ball; attacks the opponent's goal when close |
| **Defender** | Holds a weighted midpoint between the ball and own goal |

### State Machine
Each agent tracks one of three states displayed as a colour dot above their head:

| State | Colour | Meaning |
|---|---|---|
| `chase` | Grey | Moving toward the ball |
| `attack` | Yellow | Close to ball, driving toward opponent's goal |
| `support` | Blue | Defending — holding position near own goal |

### AI Logic (`agent_ai`)
1. Determine which teammate is **nearest** to the ball
2. If nearest (or striker role): chase ball → switch to attack mode when close
3. If not nearest (defender role): hold a support position between ball and own goal
4. Compute angle error → rotate toward target → move forward or reverse

---

## Architecture

```
Soccer_Twos.ipynb
│
├── Camera                  # 3D orbit camera with perspective projection
│   ├── _basis()            # Compute eye/forward/right/up vectors
│   ├── project()           # World → screen coordinates
│   ├── depth()             # Depth for painter's sort
│   └── screen_radius()     # Project sphere radius to screen
│
├── Rendering
│   ├── draw_field()        # Striped grass, boundary lines, centre circle
│   ├── draw_goal()         # 3D goal with net, posts, crossbars (depth-sorted)
│   ├── draw_agent()        # Cylindrical agent body, direction arrow, state dot
│   ├── draw_ball()         # Sphere with spinning pentagon panels + highlight
│   └── draw_hud()          # Score, episode, step count, goal flash, event log
│
├── Simulation
│   ├── Ball                # dataclass: position, velocity, spin
│   ├── Agent               # dataclass: position, velocity, angle, team, role
│   ├── agent_ai()          # Rule-based decision logic → (fwd, rot) action
│   ├── apply_action()      # Apply movement and rotation to agent
│   ├── update_ball()       # Ball physics: friction, wall bounce
│   ├── agent_ball_col()    # Ball-agent collision + goal-directed kick impulse
│   ├── agent_sep()         # Agent-agent separation (overlap resolution)
│   └── check_goal()        # Detect when ball crosses goal line
│
└── main()                  # Game loop: event handling → simulation → render
```

---

## Controls

| Input | Action |
|---|---|
| **Left Mouse Drag** | Orbit camera around field |
| **Scroll Wheel** | Zoom in / out |
| **SPACE** | Pause / Resume simulation |
| **R** | Start a new game |
| **ESC** | Quit |

---

## Environment Specification

Matches the Unity ML-Agents Soccer Twos parameters:

| Parameter | Value |
|---|---|
| Field size | 500 × 300 units |
| Goal height | 90 units |
| Agent radius | 9.0 units |
| Ball radius | 7.5 units |
| Max steps/episode | 1200 |
| FPS | 60 |
| Window | 1280 × 720 |
| Teams | 2 (Alpha, Beta) |
| Agents per team | 2 (1 Striker + 1 Defender) |

---

## Installation & Running

### 1. Install dependency

```bash
pip install pygame
```

### 2. Run from Jupyter Notebook

Open `Soccer_Twos.ipynb` and run all cells.

### 3. Run as a Python script

If you export the notebook:

```bash
jupyter nbconvert --to script Soccer_Twos.ipynb
python Soccer_Twos.py
```

---

## File Structure

```
Soccer-Twos/
│
├── Soccer_Twos.ipynb       # Main simulation notebook
└── README.md               # This file
```

---

## Key Implementation Details

### 3D Rendering (No OpenGL)
The entire 3D view is built using a **hand-rolled perspective projection**:
- A `Camera` class manages orbital rotation (theta), elevation (phi), and zoom
- World coordinates are projected to screen using a view-space transform
- Objects are depth-sorted using the **painter's algorithm** (back-to-front draw order) to handle occlusion correctly

### Physics
- Ball friction: velocity multiplied by `0.994` per step
- Wall collisions: elastic bounce with `0.75` damping
- Ball-agent collision: impulse-based response with a **goal-directed kick bias** — when an agent touches the ball, a force component pushes the ball toward the opponent's goal

### Multi-Episode Tracking
The HUD displays a running log of goals, episode results (win/loss/draw), and cumulative stats across all episodes in the current session.

---

## HUD Display

```
ALPHA  2  :  1  BETA
Ep 04   Step 0847/1200   Wins  A:2  B:1  D:0
─────────────────────────────────────────────
[Ep03 S0612] GOAL! ALPHA → 2-1
[Ep03 S0000] GOAL! BETA  → 1-1
...
```

---

## Comparison with Unity ML-Agents Soccer Twos

| Feature | Unity ML-Agents | This Project |
|---|---|---|
| Agents | 4 (2v2) | 4 (2v2)  |
| Agent control | PPO (trained RL) | Rule-based AI |
| Observation space | 336-dim ray-cast | Geometric computation |
| Actions | 3 discrete branches | Forward/back + rotate |
| 3D rendering | Unity Engine | Custom pygame renderer  |
| Reward function | Time-penalty based | Win/loss tracking |
| Dependency | Unity + ML-Agents | `pygame` only  |

---

## References

- [Unity ML-Agents Toolkit — Soccer Twos](https://github.com/Unity-Technologies/ml-agents/blob/main/docs/Learning-Environment-Examples.md#soccer-twos)
- [Pygame Documentation](https://www.pygame.org/docs/)
- Task Assignment: *Final Round — Soccer Twos Multi-Agent Game*

---
