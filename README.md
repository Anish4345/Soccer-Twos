Soccer Twos — 3D Multi-Agent Simulation
A 2v2 multi-agent soccer simulation built entirely in Python using pygame.
Inspired by the Unity ML-Agents Soccer Twos environment, this project recreates the game logic and renders it in a fully interactive 3D perspective view — with zero OpenGL or external 3D libraries.

Project Overview
PropertyDetailsTypeRule-Based Multi-Agent SimulationEnvironment2v2 Soccer (Team Alpha vs Team Beta)RenderingHand-rolled painter's-algorithm 3D → 2D projectionDependencypygame onlyPlatformPython 3.x, Jupyter Notebook / .py script
This project was built as part of a Final Round Task Assignment that required:

Creating a game environment modelled after the Unity ML-Agents Soccer Twos spec
Developing agents that can play the game autonomously


Game Rules

Teams: Team Alpha (green) vs Team Beta (red) — 2 agents per team
Goal: Get the ball into the opponent's goal; defend your own
Episode length: 1200 steps per episode
Scoring: Goals are tracked per episode; wins, losses, and draws are recorded across episodes
Reset: Ball and agents reset to starting positions after every goal or episode end


Agent Design
Agents use rule-based AI (not reinforcement learning). Each agent has a role and a simple decision loop:
Roles
RoleBehaviourStrikerAlways chases the ball; attacks the opponent's goal when closeDefenderHolds a weighted midpoint between the ball and own goal
State Machine
Each agent tracks one of three states displayed as a colour dot above their head:
StateColourMeaningchaseGreyMoving toward the ballattackYellowClose to ball, driving toward opponent's goalsupportBlueDefending — holding position near own goal
AI Logic (agent_ai)

Determine which teammate is nearest to the ball
If nearest (or striker role): chase ball → switch to attack mode when close
If not nearest (defender role): hold a support position between ball and own goal
Compute angle error → rotate toward target → move forward or reverse


Architecture
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

Controls
InputActionLeft Mouse DragOrbit camera around fieldScroll WheelZoom in / outSPACEPause / Resume simulationRStart a new gameESCQuit

Environment Specification
Matches the Unity ML-Agents Soccer Twos parameters:
ParameterValueField size500 × 300 unitsGoal height90 unitsAgent radius9.0 unitsBall radius7.5 unitsMax steps/episode1200FPS60Window1280 × 720Teams2 (Alpha, Beta)Agents per team2 (1 Striker + 1 Defender)

Installation & Running
1. Install dependency
bashpip install pygame
2. Run from Jupyter Notebook
Open Soccer_Twos.ipynb and run all cells.
3. Run as a Python script
If you export the notebook:
bashjupyter nbconvert --to script Soccer_Twos.ipynb
python Soccer_Twos.py

File Structure
Soccer-Twos/
│
├── Soccer_Twos.ipynb       # Main simulation notebook
└── README.md               # This file

Key Implementation Details
3D Rendering (No OpenGL)
The entire 3D view is built using a hand-rolled perspective projection:

A Camera class manages orbital rotation (theta), elevation (phi), and zoom
World coordinates are projected to screen using a view-space transform
Objects are depth-sorted using the painter's algorithm (back-to-front draw order) to handle occlusion correctly

Physics

Ball friction: velocity multiplied by 0.994 per step
Wall collisions: elastic bounce with 0.75 damping
Ball-agent collision: impulse-based response with a goal-directed kick bias — when an agent touches the ball, a force component pushes the ball toward the opponent's goal

Multi-Episode Tracking
The HUD displays a running log of goals, episode results (win/loss/draw), and cumulative stats across all episodes in the current session.

HUD Display
ALPHA  2  :  1  BETA
Ep 04   Step 0847/1200   Wins  A:2  B:1  D:0
─────────────────────────────────────────────
[Ep03 S0612] GOAL! ALPHA → 2-1
[Ep03 S0000] GOAL! BETA  → 1-1

Comparison with Unity ML-Agents Soccer Twos
FeatureUnity ML-AgentsThis ProjectAgents4 (2v2)4 (2v2) Agent controlPPO (trained RL)Rule-based AIObservation space336-dim ray-castGeometric computationActions3 discrete branchesForward/back + rotate3D renderingUnity EngineCustom pygame renderer Reward functionTime-penalty basedWin/loss trackingDependencyUnity + ML-Agentspygame only 

References

Unity ML-Agents Toolkit — Soccer Twos
Pygame Documentation
Task Assignment: Final Round — Soccer Twos Multi-Agent Game
