---
title: Control
summary: PID controllers and feedforward with gains tuned live, and helpers for steady decisions and smooth changes.
---

## PID

A PID controller turns an error, how far something is from its target, into an output, usually a motor power:

```kotlin
class Turret(parent: Component) : Component(parent) {
    val motor = Motor(this, "motor", ticksPerTurn = 384.5, gearRatio = 121.0 / 16)
    val gains by config(PidGains(p = 0.01, d = 0.0005))       // turret.gains = p=0.01, i=0, d=0.0005
    private val pid = Pid { gains }
    var target = 0.degrees

    override fun update() {
        motor.power = pid.update((target - motor.position).inDegrees)
    }
}
```

`update(error)` is called once per loop with `target - measured`, in any unit; the gains are per that unit. The gains
are read on every update, so changing them on the settings page takes effect right away (and clears the integral).

When a sensor measures how fast the error changes, like a motor's velocity, pass it: `pid.update(error, rate)`. The
derivative then doesn't jump with sensor noise or a moving target.

Options, all named arguments of `Pid(...)`:

| Option                 | Is                                                                              | Default      |
|------------------------|---------------------------------------------------------------------------------|--------------|
| `outputLimit`          | the output stays within ±this                                                   | 1.0          |
| `integralLimit`        | the integral term alone never gives more than ±this                             | `outputLimit`|
| `integralZone`         | the integral only builds up while the error is smaller than this                | no limit     |
| `derivativeSmoothing`  | smooths the derivative, which otherwise jumps with every encoder tick: roughly how long a change takes to show | none |
| `maxStep`              | after a pause in updates longer than this, the controller starts over           | 250 ms       |

`proportionalTerm`, `integralTerm` and `derivativeTerm` are the parts of the last output, to show or graph while
tuning, and `reset()` forgets the past.

## Feedforward

Feedforward is the power a mechanism needs to move at a velocity, worked out from how it behaves: `kS` to overcome
friction and start moving, plus `kV` per unit of velocity, plus `kA` per unit of acceleration. Measure them once, and a
PID only has to correct what is left, which makes it much easier to tune.

```kotlin
class Flywheel(parent: Component) : Component(parent) {
    val motor = Motor(this, "motor", ticksPerTurn = 28.0)
    val feedforward by config(Feedforward(kS = 0.05, kV = 0.00017))    // flywheel.feedforward = kS=0.05, kV=0.00017
    val gains by config(PidGains(p = 0.0005))
    private val pid = Pid { gains }
    var targetRpm = 0.0

    override fun update() {
        motor.power = feedforward.power(targetRpm) + pid.update(targetRpm - motor.velocity.inRpm)
    }
}
```

The velocity can be in any unit, like rpm here; the constants are per that unit. For just holding a speed, a motor's
`velocityControl` may be enough (see [Motors and servos]({{ '/platform/motors-and-servos/' | relative_url }})).

## Steady decisions and smooth changes

**`Hysteresis(on, off)`** makes an on/off decision that doesn't flicker around its threshold. With `on` above `off`,
it turns on at `on` or above and back off only at `off` or below, like running a flywheel at full power while it is far
below its target speed. With `on` below `off` it works the other way around, like a ball being in when a distance sensor
sees it closer than 13 cm, and out only once it is farther than 17:

```kotlin
private val ballIn = Hysteresis(on = 13.0, off = 17.0)
...
val hasBall = ballIn.update(distance.inCm)
```

**`Debouncer(delay)`** turns a flickering condition into a steady one: the result changes only after the condition has
stayed changed for `delay`. Like "a target is in view" from a camera that misses it for a frame now and then:

```kotlin
private val targetSeen = Debouncer(100.milliseconds)
...
val seen = targetSeen.update(camera.hasTarget)
```

**`SlewRateLimiter(ratePerSecond)`** limits how fast a value may change, like a power ramping up to a new setting rather
than jumping, which is easier on gears and keeps a tall robot from tipping:

```kotlin
private val ramp = SlewRateLimiter(ratePerSecond = 2.0)       // from 0 to full power in half a second
...
motor.power = ramp.update(wantedPower)
```

All three keep their result in `value`, and take `update()` once per loop.
