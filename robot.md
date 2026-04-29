---
layout: portfolio
title: Robot
nav_title: Robot
nav_order: 4
---

During this season we went through three complete robot
rebuilds. Each of the rebuilds was started when we realized
that no incremental improvements can adequately improve
our robot’s scoring throughput.

## Alpha
Our first iteration was inspired by publicly available design by team 23070 which was a quick way to start the season, providing a bedrock to test hardware and software.
This build was a great starting point with such a strong pros as:
- Reliable artifact uninterrupted path
- Separate intake and feeder motors, makes sure that artifacts will not be fired when not needed
- Side panels made out of a strong material
But there were also a number of cons:
- Hand cut panels lack precision, which makes shooter shoot to the side and have more friction
- Too much pressure on the artifact causes backspin
- For localization, motor encoders were used, which were not precise enough
- For pose correction, a webcam was used, which also lacked precision
- For intake and feeder low RPM motors were used, which had unnecessary torque and not enough speed
- Hard to change number plates which were secured with tape

![Alpha]({{ '/assets/images/alpha.jpg' | relative_url }})

## Beta
For regional qualifier we decided to mostly use the same design but completely re-build the robot using laser-cutter for precision. There were however some important changes made:
- ### Positioning:
    - goBILDA Pinpoint localizer used for position tracking
    - Limelight - used for position correction by tracking AprilTags
- ### Intake
    - We've designed and 3D printed a one-piece wide intake with gutters and ramp
    - We've used faster 1100RPM motors for both intake and feeder for more reliable pickup and faster shooting sequence
- ### Flywheel
    - We've implemented adjustable flywheel artifact compression so we could find an optimal balance between range and backspin

With this robot we have won Benelux championship.
![Beta]({{ '/assets/images/beta.jpg' | relative_url }})
Photo (c) FTC Benelux

## Gamma
For the FIRST championship in Houston we have decided to completely re-design the robot and start from a blank page.

### Sleek body design
Makes for a robot with no wires showing, and sides without any bumps, so when the robot comes into contact with another robot or the wall, it does not stick, and instead slides. It incorporates the number plate holder with no compromise to the flatness of the body.

### Turret
Allows the robot to shoot into the goal regardless of its heading. However, it complicates the design, reduces reliability, and requires a dedicated motor for turret movement.

### Single motor for intake and feeder
As one motor is required for the turret, the feeder motor was compromised. In order for the artifacts to not go into the flywheel before shooting is required, a latch was put in place for stopping the artifact, actuated by a servo.

### Front facing camera
A Logitech camera in a custom Limelight-style housing makes the vision modules easily interchangeable allowing to choose the best tech for recognizing artifacts on the floor and automatically collecting them.

![Gamma]({{ '/assets/images/gamma.jpg' | relative_url }})
