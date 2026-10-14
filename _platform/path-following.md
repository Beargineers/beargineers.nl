---
title: Path following
summary: Driving to poses on the field and along paths of them, and tuning how the robot gets there.
---

## Driving to a pose

`PathFollower` drives the robot to poses on the field, using where its localizer says the robot is. Create it after the
drive and the localizer:

```kotlin
class MyRobot(host: RobotHost) : Robot(host) {
    val localizer = PinpointLocalizer(this)
    val drive = MecanumDrive(this)
    val follower = PathFollower(this, drive, localizer)
}
```

Then:

```kotlin
follower.driveTo(SPIKE_1)                                               // drives there and stops
follower.driveTo(BASKET, speed = 0.5)                                   // at half power at most
follower.drivePath(Waypoint(A), Waypoint(B, speed = 0.5), Waypoint(C))  // through A and B, stops at C
follower.driveTo(localizer.pose + RobotOffset(forward = 30.cm))         // 30 cm ahead
```

Each returns when the robot has arrived: within `follower.tolerance` (2 cm) and `follower.headingTolerance` (1°), and
slower than `follower.stoppedSpeed` (10 cm/s) and `follower.stoppedTurnSpeed` (20°/s), so it doesn't carry on past the
target. They return `false` when the robot gave up: when it gets no closer for `follower.stallTime` (500 ms), like when
pushing against a wall. Slow progress isn't a stall.

The poses are where the robot's center should be, and which way it should face, in field coordinates (see
[Units and geometry]({{ '/platform/units-and-geometry/' | relative_url }})). Poses for one alliance are converted for the
other with `field.forAlliance(pose, alliance)`.

## Paths

A path is a list of waypoints. The robot drives through each, without stopping, and stops at the last:

```kotlin
follower.drivePath(
    Waypoint(Pose(60.cm, 0.cm, 0.degrees)),
    Waypoint(Pose(60.cm, 60.cm, 90.degrees), speed = 0.6),
    Waypoint(Pose(0.cm, 60.cm, 180.degrees), tolerance = 1.cm),
)
```

A point on the way counts as passed within `follower.passTolerance` (8 cm) and `follower.passHeadingTolerance` (5°),
looser than where it stops. A waypoint's own `tolerance` and `headingTolerance` override both. `speed` limits the power,
from 0.0 to 1.0.

`drivePath(waypoints, stopAtEnd = false)` drives through the last waypoint too and leaves the motors running, for
whatever drives next.

## Holding a pose

`follower.hold(pose)` keeps the robot at a pose until it is stopped, like while shooting from a spot: whenever
something pushes the robot further than the tolerance away, it drives back. It never returns: stop it like any action.

```kotlin
coroutineScope {
    val hold = launch { robot.follower.hold(robot.localizer.pose) }
    robot.shooter.shoot()
    hold.cancel()
}
```

## Driving and the drivers

Following a path takes the drivetrain over (see
[Robot code over time]({{ '/platform/robot-code-over-time/' | relative_url }})): a new drive action, or the sticks pushed
past halfway, stop it, and the motors stop with it. So a TeleOp button can drive the robot to the basket, and the
drivers can take over any time.

## How it works

The follower looks at how far the robot still has to go in its own frame: forward, sideways and around. Three PID
controllers turn those into powers for the drivetrain. Their D terms use the velocity the localizer measures, so late
readings don't make them jump.

Small powers don't move a robot against its wheels' friction, so until the robot is within tolerance, the follower adds
friction power toward the target: `follower.frictionPower` forward (0.08), `follower.frictionStrafePower` sideways
(0.14, mecanum wheels need more) and `follower.frictionTurnPower` (0.1). With a drive that has velocity control, they
can be zero.

## Tuning

The defaults were tuned on a 12 kg mecanum robot with `drive.nominalVoltage = 12`: 60 cm in 0.9 s and a quarter turn in
0.7 s, within 1 cm and 1°. Another robot may need other numbers.

1. **Friction powers:** raise a power slowly until the robot moves. Offer commands for it in a tuning opmode (see
   [Telemetry]({{ '/platform/telemetry/' | relative_url }})) and watch the robot.
2. **Gains:** offer drives as commands, like *forward 60 cm* and *back*, and run them from the graphs page while
   changing `follower.forwardGains`, `follower.strafeGains` and `follower.headingGains` on the settings page. The graphs
   `forward error cm`, `strafe error cm` and `heading error deg` show how the robot got there, next to the powers they
   turn into, `forward power`, `strafe power` and `turn power`. The log says how each drive ended.

```kotlin
@TeleOp(name = "Tune paths")
class TunePaths : CoroutineOpMode() {
    private val robot by robot(::MyRobot)

    override suspend fun CoroutineScope.onStart() {
        robot.localizer.pose = Pose.ZERO
        command("forward 60 cm") { robot.follower.driveTo(Pose(60.cm, 0.cm, 0.degrees)) }
        command("back") { robot.follower.driveTo(Pose.ZERO) }
        command("quarter turn") { robot.follower.driveTo(robot.localizer.pose.turnBy(90.degrees)) }
    }
}
```

| Setting                                 | Is                                                         | Default                |
|-----------------------------------------|------------------------------------------------------------|------------------------|
| `follower.forwardGains`                 | power per cm still to go forward                           | `p=0.03, i=0.002, d=0.005` |
| `follower.strafeGains`                  | power per cm still to go sideways                          | `p=0.045, d=0.006`     |
| `follower.headingGains`                 | power per degree still to turn                             | `p=0.04, i=0.015, d=0.004` |
| `follower.frictionPower`                | power that just overcomes friction driving forward         | 0.08                   |
| `follower.frictionStrafePower`          | the same, sideways                                         | 0.14                   |
| `follower.frictionTurnPower`            | the same, turning                                          | 0.1                    |
| `follower.tolerance`, `headingTolerance`| how close counts as arrived, where the robot stops          | 2 cm, 1°               |
| `follower.passTolerance`, `passHeadingTolerance` | how close counts as passed, on the way            | 8 cm, 5°               |
| `follower.stoppedSpeed`, `stoppedTurnSpeed` | arrived only once slower than these                    | 10 cm/s, 20°/s         |
| `follower.stallTime`                    | gives up when getting no closer for this long              | 500 ms                 |
