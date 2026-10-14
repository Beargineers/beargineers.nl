---
title: Motors and servos
summary: Motors with positions and speeds in real units, servos that turn to a position or at a power, and any other device.
---

Motors and servos are parts of the robot like any other: they belong to the part that uses them, and their settings
start with that part's path.

## Motors

```kotlin
class Turret(parent: Component) : Component(parent) {
    // goBILDA 435 RPM: 384.5 ticks per turn of its shaft, from goBILDA's product page. The turret is centered by hand
    // once, and autonomous and TeleOp share that zero.
    val motor = Motor(this, "motor", device = "turret", ticksPerTurn = 384.5, gearRatio = 121.0 / 16, keepZero = true)
}
```

`Motor(this, "motor", device = "turret")` is the part `turret.motor`, and the motor `turret` in the robot configuration;
without `device`, the name is the motor's name in the robot configuration too.

| Property                | Is                                                                                         |
|-------------------------|--------------------------------------------------------------------------------------------|
| `power`                 | from -1.0 to 1.0; set it to drive the motor                                                |
| `position`              | how far the output turned since INIT or the last `resetPosition()`, as an `Angle`          |
| `velocity`              | how fast the output turns, as an `AngularVelocity`: `motor.velocity.inRpm`                 |
| `ticks`                 | encoder ticks since INIT or the last `resetPosition()`                                     |
| `device`                | FIRST's `DcMotorEx`, for anything the platform doesn't cover                               |

`resetPosition()` makes the current position the new zero.

Settings, all with the constructor arguments as defaults, so they can be fixed on the robot without rebuilding:

| Setting                       | Is                                                                                    |
|-------------------------------|---------------------------------------------------------------------------------------|
| `turret.motor.direction`      | `F` or `R`: which way positive power turns                                            |
| `turret.motor.brake`          | whether the motor holds its position at zero power, instead of coasting; on by default |
| `turret.motor.ticksPerTurn`   | encoder ticks per turn of the motor's own output shaft                                |
| `turret.motor.gearRatio`      | turns of the motor's shaft per turn of what it drives, for gears after the motor      |
| `turret.motor.keepZero`       | keep the position's zero from one opmode to the next                                  |
| `turret.motor.velocityControl`| let the hub hold the motor's speed                                                    |
| `turret.motor.powerTolerance` | how much the power must change before it is sent to the hub (0.005)                   |

### Positions and speeds

`position` and `velocity` are measured at the output, after the gears, in real units. They need `ticksPerTurn`: the
encoder resolution "at the output shaft" on the motor's product page, like 384.5 for a goBILDA 435 RPM motor or 1425.1
for a 117 RPM one. Without it they fail, saying what to set; a motor whose encoder isn't used doesn't need it.

The motor type picked in the robot configuration isn't used for this, because it is wrong for most current motors: the
SDK's goBILDA type always has the 60 RPM model's numbers, and its REV UltraPlanetary type assumes a 20:1 gearbox.

With `gearRatio`, positions are of what the motor drives: a turret turned by a 16-tooth gear on the motor meshing
with a 121-tooth gear has `gearRatio = 121.0 / 16`, and `position` is the turret's angle.

### Keeping the zero

A motor's zero is where it was at INIT. With `keepZero`, it is where it was at the first INIT after the robot app
started, and the opmodes after share it: for a turret or an arm set to a known position once, before the match, and
used by autonomous and TeleOp alike. A hub that lost power counts from zero again, which the platform can't tell.

### Holding a speed

With `velocityControl`, the hub holds the motor's speed with its encoder: `power` then asks for a share of the motor's
top speed, which the motor keeps whatever the load, friction or battery, even at powers too small to move it
otherwise. The top speed comes from the motor type in the robot configuration; for goBILDA and REV motors it is right
whatever the gearbox.

It needs the encoder plugged in. Without it the hub never sees the motor move and keeps raising the power, up to full:
so when the encoder doesn't move for half a second under power, the motor is stopped and the Driver Station says why.

### Writes to the hub

Every power sent to the hub takes time on the connection all devices share (see
[Performance]({{ '/platform/performance/' | relative_url }})). So a power is only sent when it differs from the last one
sent by at least `powerTolerance`, and code like a PID that nudges the power every loop doesn't write every loop.
Stopping and full power are always sent exactly. `sentPower` is what the hub has.

Motors stop when the opmode ends.

## Servos that turn to a position

```kotlin
class Shooter(parent: Component) : Component(parent) {
    val latch = PositionServo(this, "latch")
    val latchOpen by config(0.3)
    val latchClosed by config(0.65)

    fun open() { latch.position = latchOpen }
    fun close() { latch.position = latchClosed }
}
```

`position` goes from 0.0 to 1.0; values outside are clamped. The servo holds still until the first position is set,
and a position is only sent to the hub when it changes. Keep the positions as settings of the part that uses the servo,
like `latchOpen` above, so they are tuned live.

Settings: `shooter.latch.direction` (`F` or `R`), and `pwmMin` and `pwmMax`, the pulse widths in microseconds for
positions 0 and 1, for servos that need other ones than the SDK's 600 and 2400.

## Servos that turn at a power

A continuous rotation servo, like on an intake's rollers, turns at a `power` from -1.0 to 1.0:

```kotlin
val roller = CRServo(this, "roller")
...
roller.power = 1.0
```

Like a motor's, its power is only sent when it differs from what the hub has by at least `powerTolerance` (0.005), and it
stops when the opmode ends. Settings: `direction`, `powerTolerance`, and `pwmMin` and `pwmMax`: many continuous
rotation servos reach full speed only at 500 and 2500.

## Any other device

For a device the platform has no class for, take FIRST's own from the robot configuration:

```kotlin
class Vision(parent: Component) : Component(parent) {
    val limelight by hardware<Limelight3A>()                    // the device "limelight"
    val imu by hardware<IMU>("controlHubImu")                   // named differently
}
```

The device is named like the property unless a name is given. It is looked up on first use, which has to be in
`init()` or later: hardware isn't available in constructors. A device the robot configuration doesn't have stops the
opmode when it is first used, naming it; the platform's own classes use theirs in `init()`, so they fail INIT.

Every platform class gives its SDK device as `device` too, like `motor.device`, for what the platform doesn't cover.
