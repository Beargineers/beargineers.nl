---
layout: portfolio
title: Robot
nav_title: Robot
nav_order: 4
subtitle: Integrated design, control, and sensing stack
---

## Design Brainstorm {#design-brainstorm}
![Ramp]({{ '/assets/images/ramp.jpg' | relative_url }})
Front-mounted ramp intake concept for smooth cycles and minimal jams.

- Ramp guides artifacts into short rollers so the driver can approach at angles.
- Gentle incline plus compression keeps artifacts controlled while entering the robot.
- Fast, reliable pickup with easy integration to feeder and shooter systems.

## Design Process {#design-process}
![Iterations]({{ '/assets/images/iterations.jpg' | relative_url }})
Scrimmage feedback led to reliability upgrades and tighter manufacturing.

- Rebuilt ramp around artifact dimensions to stop bounce-outs on fast intakes.
- Added dead-wheel odometry and a camera for automated alignment and distance-aware shooter control.
- Enlarged intake window to contain artifacts during aggressive driving.
- Shifted from hand-cut parts to laser-cut components for precision and strength.

## Chassis {#chassis}
4-motor mecanum drivetrain prioritized alignment speed over pushing power.

- 96mm mecanum wheels (X-pattern), 1:1 drive, tight wheelbase to cut rotational inertia.
- Low, centered mass to reduce pitching while accelerating and shooting.
- Strafing keeps turn corrections short; pushing power tradeoff mitigated with driver practice.

## Intake {#intake}
Fast ground pickup with positive control into the feeder.

- Rubberized rollers pull artifacts via friction to avoid over-compression jams.
- Mid-feeder wheel on a 1000 RPM motor pushes artifacts toward the shooter.
- Tuned belt path maintains wrap and tension through the pivot arc without sharp bends.
- 3D-printed ramp guides misaligned artifacts; tensioner tested for balanced grip and speed.

## Shooter {#shooter}
Tuned flywheel with adjustable pressure and fast feeding.

- PIDF control (no integral) for quick target-speed convergence while targets change.
- Distance-to-speed table (10 points) simplified into a linear formula for live adjustments.
- Adjustable contact part balances backspin vs. bounce-outs.
- Grippy feeder wheel on a 1000 RPM motor queues three artifacts rapidly.

## Code & Automation {#code}
Custom stack for parallel actions in auto and tele-op.

- Wrote our own framework for flexibility and deeper system understanding.
- Phase-based autonomous: main loop runs tasks in parallel (drive + flywheel tuning, etc.).
- Robot entities store geometry/tuning; shared main code carries versions forward.
- FollowPath PD controller accepts waypoints for smooth arrivals.
- Tele-op buttons trigger path moves; auto-shoot when flywheel speed, heading, and position are valid.

## Autonomous Strategy {#autonomous}
Pattern-aware routines built from modular building blocks.

- Preloaded artifacts stay in order; auto-shoot motif matches without opening the ramp for RP.
- Spike pickup order favors the right spike first to hit counts divisible by three.
- End every routine outside the shooting zone, finishing near the ramp for tele-op handoff.
- “Autonomous maker” composes new autos from menu blocks; preset mirrored routines cover far/close shooting.

## Sensors {#sensors}
Vision, odometry, and artifact counting for consistent cycles.

- Moved from webcam to **Limelight** to offload processing and improve accuracy.
- **Gobilda PinPoint** dead wheels plus IMU give slip-proof localization when vision is unavailable.
- Distance sensor counts artifacts entering from the ramp and halts intake after three; drivers still monitor holes in artifacts.
