---
title: BearPlatform
permalink: /platform/
nav_title: Platform
nav_order: 3
layout: docs
eyebrow: Robot software
subtitle: A Kotlin library for programming FTC robots, made by Beargineers for students and open to every team.
---

BearPlatform is what Beargineers program their robots with. It sits on top of FIRST's FTC SDK: the SDK talks to the
hubs, and BearPlatform gives robot code a shape that students can read, change and reuse from one season to the next.

- **A robot is made of parts.** A drivetrain, an intake, a claw: each is a small class with its own settings, and
  parts are made of parts.
- **Settings change on the robot.** Servo positions, PID gains and field positions are declared right where the code
  uses them, and tuned live from a laptop on the robot's WiFi, without rebuilding the app.
- **Robot code reads top to bottom.** An autonomous is a list of steps that wait for each other; button handlers and
  every-loop work run alongside, and none of it needs threads.
- **Real units.** Distances, angles and speeds carry their units, so `30.cm` can't be mistaken for 30 inches, and
  turning the robot never needs angle wrapping by hand.
- **Driving comes with it.** A mecanum drive, goBILDA Pinpoint odometry, field-centric TeleOp and driving to poses on the
  field.
- **You can see what the robot does.** Driver Station lines, live graphs in the browser, logs, and buttons that start
  actions for testing.

## What robot code looks like

A part of the robot, with settings that can be tuned on the robot:

```kotlin
class Claw(parent: Component) : Component(parent) {
    val servo = PositionServo(this, "servo", device = "claw")
    val openPosition by config(0.6)                 // the setting claw.openPosition
    val closedPosition by config(0.3)
    val closeTime by config(300.milliseconds)

    fun open() { servo.position = openPosition }
    fun close() { servo.position = closedPosition }

    /** Closes, and returns once the claw holds on */
    suspend fun grab() {
        close()
        delay(closeTime)
    }
}
```

The robot, made of its parts:

```kotlin
class MyRobot(host: RobotHost) : Robot(host) {
    val localizer = PinpointLocalizer(this)         // goBILDA Pinpoint odometry
    val drive = MecanumDrive(this)
    val follower = PathFollower(this, drive, localizer)
    val claw = Claw(this)
}
```

An autonomous, one step after another:

```kotlin
@Autonomous(name = "Auto")
class Auto : CoroutineOpMode() {
    private val robot by robot(::MyRobot)

    override suspend fun CoroutineScope.onStart() {
        robot.claw.grab()
        robot.follower.driveTo(Pose(60.cm, 0.cm, 90.degrees))
        robot.claw.open()
    }
}
```

And a TeleOp, where buttons start actions and the sticks drive every loop:

```kotlin
@TeleOp(name = "Driving")
class Driving : CoroutineOpMode() {
    private val robot by robot(::MyRobot)

    override suspend fun CoroutineScope.onStart() {
        controller1.rightBumper.onPress { robot.claw.grab() }
        controller1.leftBumper.onPress { robot.claw.open() }
        everyTick { robot.drive.driveWith(controller1.leftStick, controller1.rightStick.x) }
    }
}
```

## Where to start

- **A new robot project:** [Getting started]({{ '/platform/getting-started/' | relative_url }}) sets one up from the
  [Quickstart](https://github.com/Beargineers/Quickstart), which drives a mecanum robot out of the box.
- **The ideas everything builds on:** [Robots and parts]({{ '/platform/robots/' | relative_url }}),
  [Opmodes]({{ '/platform/opmodes/' | relative_url }}) and
  [Robot code over time]({{ '/platform/robot-code-over-time/' | relative_url }}). Read them in that order; the other pages
  can be read as needed.
- **New to Kotlin:** start with [Kotlin for FTC]({{ '/learn/kotlin/kotlin-for-ftc/' | relative_url }}).
- **Every class and function:** the API reference for
  [nl.beargineers.platform:ftc](https://javadoc.io/doc/nl.beargineers.platform/ftc) and
  [nl.beargineers.platform:core](https://javadoc.io/doc/nl.beargineers.platform/core).
- **The source, issues and contributing:** [github.com/Beargineers/BearPlatform](https://github.com/Beargineers/BearPlatform).

## Versions

BearPlatform is on Maven Central as `nl.beargineers.platform:ftc`. While its version starts with `0.`, a release may
change the API, but only between seasons: during a season, releases fix and add. These pages describe the latest
release. It is free to use, under the MIT license.
