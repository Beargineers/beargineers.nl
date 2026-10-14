---
title: Opmodes
summary: What an opmode does from INIT to STOP, and where robot code goes in it.
---

## An opmode

Opmodes are what the drivers pick on the Driver Station. With BearPlatform they extend `CoroutineOpMode`, and carry
FIRST's `@TeleOp` or `@Autonomous` annotation as usual:

```kotlin
@TeleOp(name = "Driving")
class Driving : CoroutineOpMode() {
    private val robot by robot(::MyRobot)

    override suspend fun CoroutineScope.onInit() {
        everyTick { show("Ready when you are") }
    }

    override suspend fun CoroutineScope.onStart() {
        controller1.a.onPress { robot.claw.grab() }
        everyTick { robot.drive.driveWith(controller1.leftStick, controller1.rightStick.x) }
    }

    override fun onStop() {
        // after everything has stopped, rarely needed
    }
}
```

- **`val robot by robot(::MyRobot)`** declares the opmode's robot. It is created at INIT, not with the opmode, so a
  robot that can't be created or set up, like one whose motor is missing from the robot configuration, fails INIT with
  the reason on the Driver Station. Use it from
  `onInit` and `onStart` on. For an app serving several robots, `robot(TeamRobots)` creates whichever robot this one is
  (see [Robots and parts]({{ '/platform/robots/' | relative_url }})).
- **`onInit`** runs from INIT until START: for menus, picking a starting position, or showing that the robot is ready.
  START stops it, along with every handler and every-loop block it set up.
- **`onStart`** runs from START until the opmode ends. An autonomous puts its steps here; a TeleOp sets up its buttons
  and its every-loop driving.
- **`onStop`** runs at the very end, after robot code and the robot's parts have stopped.

Both `onInit` and `onStart` are coroutines: they can wait, and what they set up runs alongside them. See
[Robot code over time]({{ '/platform/robot-code-over-time/' | relative_url }}).

## INIT

Pressing INIT on the Driver Station:

1. creates the robot and its parts;
2. loads the robot's settings (see [Settings]({{ '/platform/settings/' | relative_url }})). A value that can't be read,
   like `drive.maxPower = fast`, fails INIT and lists every such value, rather than failing in the middle of a match;
3. initializes the parts: their `init()` sets up the hardware;
4. starts `onInit`, and loops until START.

Until START, the Driver Station also shows what needs attention: settings nobody uses (often typos, with a "did you
mean" hint), warnings from the robot's parts, and a battery below `battery.low` (12 V).

## Every loop

From INIT to STOP, the opmode loops, and every loop:

1. reads the gamepads, digital sensors and the newest readings of slow sensors, so everything in the loop sees the same
   state;
2. updates the robot's parts, parents before their parts, in the order they were created;
3. runs robot code that is due: button handlers, every-loop blocks, actions;
4. sends what was shown to the Driver Station and the robot's graphs page.

A loop that takes less than `loop.minTime` (5 ms) sleeps for the rest: looping faster than sensors change only takes
time from the hubs. The first lines on the Driver Station tell how long loops take, like
*Loop: 2.4 ms average, 6.1 ms longest*, and how the battery is doing. [Performance]({{ '/platform/performance/' | relative_url }})
explains what makes loops slow.

## STOP

When the opmode ends, by STOP on the Driver Station or the end of autonomous, robot code is stopped and runs its
cleanup (`finally` blocks), then the robot's parts stop: motors and servos the platform provides stop by themselves.
Then `onStop` runs.

The robot's pose carries over to the next opmode: TeleOp starts where autonomous ended (see
[Localization]({{ '/platform/localization/' | relative_url }})).

## Showing what happens

Opmodes show lines on the Driver Station, record values for the live graphs, write to the log, and offer commands that
start actions from a laptop:

```kotlin
everyTick {
    show("Pose") { robot.localizer.pose }          // computed only when lines are sent
    graph("shooter rpm", robot.shooter.rpm)
}
command("forward 60 cm") { robot.follower.driveTo(Pose(60.cm, 0.cm, 0.degrees)) }
```

[Telemetry]({{ '/platform/telemetry/' | relative_url }}) has all of it.

## Opmodes for both alliances

Opmodes that differ only by alliance are one abstract class with a subclass for each:

```kotlin
abstract class Driving(private val alliance: Alliance) : CoroutineOpMode() {
    private val robot by robot(::MyRobot)

    override suspend fun CoroutineScope.onStart() {
        val driversFace = Field2026.driversFace(alliance)
        everyTick {
            robot.drive.driveWith(controller1.leftStick, controller1.rightStick.x, robot.localizer.pose.heading, driversFace)
        }
    }
}

@TeleOp(name = "Red driving")
class RedDriving : Driving(Alliance.RED)

@TeleOp(name = "Blue driving")
class BlueDriving : Driving(Alliance.BLUE)
```

The season's field (here `Field2026`) knows where each alliance's drivers stand; see
[Units and geometry]({{ '/platform/units-and-geometry/' | relative_url }}).
