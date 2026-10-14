---
title: Robots and parts
summary: A robot is a tree of components, each with its name, settings and place in the robot's life.
---

## Parts are components

Every part of the robot is a `Component`: a subsystem like the intake, or a piece of one, like the shooter's latch.
A component is created with its parent, and that makes it a part of the parent:

```kotlin
class Shooter(parent: Component) : Component(parent) {
    val latch = PositionServo(this, "latch")
    val flywheel = Motor(this, "flywheel")
    val kV by config(0.00036)                  // the setting shooter.kV
}
```

At the top of the tree is the `Robot`:

```kotlin
class MyRobot(host: RobotHost) : Robot(host) {
    val localizer = PinpointLocalizer(this)    // created first, so it updates first
    val drive = MecanumDrive(this)
    val shooter = Shooter(this)
    val leftFlywheel = Flywheel(this, "left")  // two of the same kind need names
    val rightFlywheel = Flywheel(this, "right")
}
```

The opmode creates the robot (see [Opmodes]({{ '/platform/opmodes/' | relative_url }})), and with it all of its parts.

## Names and paths

A component's name is its class name, starting with a small letter: a `Shooter` is `shooter`. Two parts of the same
parent can't have the same name, so the second `Flywheel` needs one of its own, `Flywheel(this, "right")`. Names may
have letters, digits, `_` and `-`.

The names from the robot down to a part make its path: the shooter's latch is `shooter.latch`. The robot's own name
isn't part of it, which keeps paths short. Paths show up everywhere:

- **Settings** of a part start with its path: `val open by config(0.3)` in the latch is the setting `shooter.latch.open`
  (see [Settings]({{ '/platform/settings/' | relative_url }})). Settings declared in the robot class itself start with
  `robot.`
- **Hardware** parts like motors and servos are named in the robot configuration on the Driver Station by their name,
  or by the `device` given to them: `PositionServo(this, "servo", device = "claw")` is the part `claw.servo`, and the
  servo `claw` in the robot configuration.
- **Log messages** of a part start with its path.

## A part's life

A component takes part in the robot's life by overriding three functions:

```kotlin
class Intake(parent: Component) : Component(parent) {
    val roller = CRServo(this, "roller")
    var on = false

    override fun init() { /* once, before the first loop: set up what needs the hardware */ }

    override fun update() {
        // every loop, before robot code runs: read sensors, apply what was asked for
        roller.power = if (on) 1.0 else 0.0
    }

    override fun stop() { /* once, at the end: leave the hardware safe */ }
}
```

- **`init()`** runs once, at INIT. Hardware is available from here on, not in constructors: create parts in the
  constructor, use their devices from `init()`.
- **`update()`** runs once every loop, before robot code (handlers, actions, everyTick blocks) runs, so robot code
  sees fresh readings.
- **`stop()`** runs once when the opmode ends. Motors and servos the platform provides stop by themselves.

Parents go before their parts, and siblings in the order they were created: create the localizer first, so everything
that drives by the robot's pose sees the new one in the same loop. `stop()` goes the other way around.

## Reporting from a part

A component can show lines on the Driver Station, record values for the live graphs, and write to the log (see
[Telemetry]({{ '/platform/telemetry/' | relative_url }})):

```kotlin
override fun update() {
    show("Intake", if (on) "on" else "off")
    graph("roller power", roller.power)
}
```

`warn("...")` reports a problem worth fixing that doesn't stop the robot, like a doubtful setting: the Driver Station
shows it during INIT, and it goes to the log.

## Actions

Most parts can only do one thing at a time: an arm can't go up and down at once, and a drivetrain can't drive to two
places. Functions that make a part do something over time take it over:

```kotlin
suspend fun Arm.moveTo(angle: Angle) = takeOver("move to $angle") {
    target = angle
    waitUntil { isAt(angle) }
}
```

Taking over stops whatever action had the part before, so the latest request wins. `isBusy`, `currentAction` and
`cancelAction()` tell about and stop the action in control. [Robot code over time]({{ '/platform/robot-code-over-time/' | relative_url }})
explains actions in full.

## One app for several robots

A team with more than one robot, or a practice bot next to the competition robot, can have one app for all of them.
Which robot it is comes from the robot configuration picked on the Driver Station:

```kotlin
object TeamRobots : RobotChoice<TeamRobot>(
    "competition" to ::CompetitionRobot,
    "practice" to ::PracticeRobot,
)
```

Opmodes then create whichever robot this one is with `val robot by robot(TeamRobots)`, and the settings file can have a
section for each robot (see [Settings]({{ '/platform/settings/' | relative_url }})). A robot configuration the app
doesn't know fails INIT, listing the ones it does. An opmode for one of the robots only, like a tuning opmode for a
mechanism the other robot doesn't have, uses `robot(TeamRobots.only<CompetitionRobot>())`, which fails INIT on the
other robot, saying so.

`TeamRobot` is what both robots have in common: a base class with what the shared opmodes need, where each robot
says what its parts are:

```kotlin
abstract class TeamRobot(host: RobotHost) : Robot(host) {
    abstract val localizer: Localizer
    abstract val drive: OmniDrivetrain
    abstract val follower: PathFollower
    abstract val intake: Intake
}

class PracticeRobot(host: RobotHost) : TeamRobot(host) {
    override val localizer = PinpointLocalizer(this)
    override val drive = MecanumDrive(this)
    override val follower = PathFollower(this, drive, localizer)
    override val intake = RollerIntake(this)
}
```
