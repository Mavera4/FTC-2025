# FTC Team 21311 — Lion Robotics Gold
**Clear Lake, Iowa** | Season: DECODE 2025–26

> Programmer & Co-Captain: Arman Nikoueiha

---

## Robot Overview

Our robot is built around a **turreted flywheel shooter**, a **spindexer sorting system**, and a **vectored intake** — all coordinated by a sensor-fusion autonomous pipeline. The software stack is written in Java using the FTC SDK.

---

## Software Highlights

### Turret Auto-Aiming Algorithm
The turret rotates independently from the drivetrain, allowing the robot to shoot while driving or defending. The targeting algorithm continuously calculates the required turret angle using the robot's live odometry position and the known goal coordinates:

```
θ_turret = arctan((140 - y) / (140 - x)) - θ_heading
```

Odometry drift is corrected in real-time using **Limelight camera relocalization** off April Tags on the goals, keeping the turret locked on target throughout the match.

---

### Shooting While Moving
To counter defense and reduce cycle times, we implemented a predictive shooting algorithm that accounts for the robot's velocity vector. Rather than stopping to shoot, the turret pre-compensates for translational motion:

```
θ_turret_adj = arctan((140 - (y + vy*d)) / (140 - (x + vx*d))) - (θ_heading + v_heading * d)
```

This lets the robot score while being pushed — a meaningful competitive advantage in tight matches.

---

### Flywheel Speed Control (Bang-Bang Controller)
We built a **Bang-Bang flywheel velocity controller** adapted from our VEX Spin Up robot. Accurate flywheel speed is critical for shot consistency at varying distances, so we:

1. Collected flywheel velocity data at multiple distances
2. Fit a **linear regression** to the data
3. Used the model to auto-select target RPM based on distance to goal:

```
distanceToTarget = sqrt(x² + y²)
flywheelTargetVelocity = 12 * distanceToTarget + 2300
```

We shared this algorithm with four other teams in our league (23791, 26744, 26743, 6546).

---

### Automated Hood Angle Control
The shooter hood angle adjusts automatically based on the flywheel's current angular velocity, ensuring consistent shot arc regardless of spin-up state. This reduced driver workload and improved scoring consistency from multiple field positions.

---

### Spindexer Sorting System
The spindexer is a horizontal three-slot carousel that sorts artifacts by color before feeding them into the shooter. Key software components:

- **Dual color sensors** identify artifact color at intake and track which color is in each slot
- Slot positions and colors are stored in an **ordered list** (indexed array) for fast lookup
- The system ensures correct shot pattern for maximum ranking points in autonomous

---

### Pedro Pathing — Autonomous Navigation
We migrated from Roadrunner to **Pedro Pathing** (an Advanced Reactive Vector Follower) for our autonomous routine. Pedro generates **Bézier curve paths** from start, control, and end points, then follows them using **PIDF loops** controlling both angular and translational velocity.

Key advantage over Roadrunner: Pedro is significantly more robust to localization errors, which made our autonomous far more consistent match-to-match.

---

### Launch Zone Detection
We implemented launch zone detection to trim our average autonomous cycle from ~8s to ~7.5s — saving enough time for one additional shot cycle worth ~9 points per match. The algorithm uses the robot's geometry (corner offset `L`, angle `θ`) to precisely determine when the robot has entered the scoring zone.

---

### Sensor Fusion (10 Sensors)
| Sensor | Purpose |
|---|---|
| Limelight Camera | Artifact tracking (auto), April Tag relocalization (tele-op) |
| 2x Color Sensors | Artifact color identification for spindexer sorting |
| 2x Motor Encoders | Turret positioning, flywheel speed control, hood angle |
| 2x Odometry Wheels | Dead-wheel position tracking in autonomous |
| Axon Servo Encoder | Spindexer slot positioning |
| Pinpoint Computer | High-refresh-rate odometry fusion for turret tracking |

---

### Object-Oriented Architecture
All major mechanisms (turret, spindexer, shooter, intake) are implemented as **reusable Java classes** with well-defined action methods. This kept the codebase modular and made it easy to call subsystem actions from both autonomous and tele-op OpModes without code duplication.

---

## Stack
- **Language:** Java
- **Framework:** FTC SDK
- **Autonomous:** Pedro Pathing (Bézier curve + PIDF)
- **Vision:** Limelight (April Tags + object detection)
- **Odometry:** Pinpoint Computer + 2x dead wheels

---

*Team 21311 — Lion Robotics Gold | Clear Lake, Iowa*
