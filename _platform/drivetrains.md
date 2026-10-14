---
title: Drivetrains
summary: Mecanum and tank drivetrains, driving with the sticks, robot-centric and field-centric.
---

## Two kinds

Drivetrains come in two kinds, which take different commands and suit different controls:

- **`OmniDrivetrain`** moves in any direction while turning: mecanum, X and swerve drives. Path following and
  field-centric driving need one. `MecanumDrive` is one.
- **`TankDrivetrain`** drives forward and back and turns, with the wheels of each side turning together: tank and
  six-wheel drives. It can't move sideways.

Both are parts of the robot, with the actions of any part: driving to a target takes the drivetrain over, and the
sticks stop it when pushed far (see below).

## Mecanum drive

```kotlin
class MyRobot(host: RobotHost) : Robot(host) {
    val drive = MecanumDrive(this)       // motors leftFront, rightFront, leftBack, rightBack
}
```

The four motors are named `leftFront`, `rightFront`, `leftBack` and `rightBack` in the robot configuration, and are
the drive's parts: `drive.leftFront` is a [motor]({{ '/platform/motors-and-servos/' | relative_url }}) with its own
settings. The left motors run reversed by default, as they are mounted mirrored to the right ones.

**Checking the wheels:** `drive.checkWheels()` turns the wheels one at a time for a second each, and the Driver Station
shows which. Each should turn so it would push the robot forward. One that turns backward needs its direction flipped,
like `drive.leftFront.direction = F`; a different wheel turning than the one shown means the motors are plugged into
other ports than configured. Put it on a button: `controller1.back.onPress { robot.drive.checkWheels() }`.

| Setting               | Is                                                                                      |
|-----------------------|-----------------------------------------------------------------------------------------|
| `drive.maxPower`      | scales every power down: 0.3 for the first tries (1.0)                                  |
| `drive.nominalVoltage`| scales powers by this over the battery's voltage, so the robot drives the same on a full battery as on a tired one: with 12 here, a power of 0.5 is 0.46 on 13 V. Path following stays tuned as the battery runs down. Full power stays full power. |
| `drive.strafeScale`   | multiplies the sideways power, as mecanum wheels slip more sideways: raise it until a sideways push of the stick is as fast as a forward one (1.0) |
| `drive.leftFront.direction` and the others | each motor's settings                                              |

`MecanumDrive(this, velocityControl = true)` lets the hub hold each wheel's speed with its encoder: powers then ask for
speeds, so even small ones move the robot, whatever the battery. It needs the drive encoders plugged in (see
[Motors and servos]({{ '/platform/motors-and-servos/' | relative_url }})). `ticksPerTurn` is only needed to read the
wheels' positions or speeds: 384.5 for goBILDA 435 RPM motors, 537.7 for 312 RPM.

## Driving with powers

Code drives an omni drivetrain with powers relative to the robot, each from -1.0 to 1.0:

```kotlin
drive.drive(forward = 0.5, right = 0.2, turn = 0.1)    // turn is counterclockwise, like all angles
drive.stopMoving()
```

When that needs more than full power from a wheel, all powers are scaled down together: the robot moves the same way,
only slower. Code that drives for a while takes the drivetrain over and stops it at the end:

```kotlin
suspend fun MyRobot.backOff() = drive.takeOver("back off") {
    try {
        drive.drive(forward = -0.3)
        delay(500.milliseconds)
    } finally {
        drive.stopMoving()
    }
}
```

To drive to places on the field, use [path following]({{ '/platform/path-following/' | relative_url }}).

## Driving with the sticks

Robot-centric: the left stick moves the robot forward and sideways, the right stick turns it, clockwise when pushed
right.

```kotlin
everyTick { robot.drive.driveWith(controller1.leftStick, controller1.rightStick.x) }
```

Field-centric: pushing the stick forward drives away from the drivers, whichever way the robot faces. It needs the
robot's heading from its [localizer]({{ '/platform/localization/' | relative_url }}), and the way the drivers look:

```kotlin
val driversFace = Field2026.driversFace(alliance)
everyTick {
    robot.drive.driveWith(controller1.leftStick, controller1.rightStick.x, robot.localizer.pose.heading, driversFace)
}
```

`Field2026.driversFace(alliance)` comes from the season's field (see
[Units and geometry]({{ '/platform/units-and-geometry/' | relative_url }})). Without one, use the heading the robot had
when it pointed away from the drivers, like at the start of autonomous, and let a button set it again:

```kotlin
var driversFace = Heading.ZERO                                   // the way the robot faced when its pose was zeroed
controller1.y.onPress { driversFace = robot.localizer.pose.heading }   // the robot faces away from me now
```

### Sticks and actions

When an action drives the robot, like driving to a target, the sticks leave it alone until one is pushed past halfway:
then the action stops, and the sticks drive. A thumb resting on a stick doesn't stop it. The threshold is
`driveWith(..., takeOverAbove = 0.5)`.

So TeleOp can have both: a button starts a drive to the basket, and the drivers take over any time.

## Tank drives

A tank drive is a `TankDrivetrain`, driven with powers forward and turning:

```kotlin
drive.drive(forward = 0.5, turn = 0.2)     // the left wheels get forward - turn, the right ones forward + turn
```

In TeleOp, with tank controls, each stick drives its side:

```kotlin
everyTick { robot.drive.driveWith(controller1.leftStick.y, controller1.rightStick.y) }
```

The platform doesn't have an FTC tank drive class yet; one is a few lines:

```kotlin
class TankDrive(parent: Component) : TankDrivetrain(parent, "drive") {
    val left = Motor(this, "left", direction = DcMotorSimple.Direction.REVERSE)
    val right = Motor(this, "right")

    override fun drive(forward: Double, turn: Double) {
        val l = forward - turn
        val r = forward + turn
        val scale = maxOf(1.0, abs(l), abs(r))     // keep the direction when a side would need more than full power
        left.power = l / scale
        right.power = r / scale
    }
}
```

Path following and field-centric driving need to move sideways, so they work with omni drivetrains only.
