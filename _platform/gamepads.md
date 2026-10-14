---
title: Gamepads
summary: Buttons, triggers and sticks, read once per loop, with handlers for presses, taps and holds.
---

## The two controllers

Every opmode has `controller1` and `controller2`, the Driver Station's gamepads 1 and 2. They are read once at the start
of every loop, so all code in a loop sees the same state: a press is noticed by everything that looks for it in the
loop where it happened.

| Buttons                                   | Triggers                        | Sticks                     |
|-------------------------------------------|---------------------------------|----------------------------|
| `a`, `b`, `x`, `y`                        | `leftTrigger`, `rightTrigger`   | `leftStick`, `rightStick`  |
| `dpadUp`, `dpadDown`, `dpadLeft`, `dpadRight` |                             |                            |
| `leftBumper`, `rightBumper`               |                                 |                            |
| `leftStickButton`, `rightStickButton`     |                                 |                            |
| `back`, `start`, `guide`, `touchpad`      |                                 |                            |

PlayStation-style buttons are the same as their Xbox-style ones: `a` is cross, `b` circle, `x` square, `y` triangle.

## Buttons

Check a button in the moment:

| Property        | Is                                                  |
|-----------------|-----------------------------------------------------|
| `isDown`        | whether it is down                                  |
| `wasPressed`    | whether it went down in this loop                   |
| `wasReleased`   | whether it went up in this loop                     |
| `heldFor`       | how long it has been down; zero when it is up       |

Or have it start robot code:

```kotlin
override suspend fun CoroutineScope.onStart() {
    controller1.rightBumper.onPress { robot.scoreSample() }    // pressing again starts it over
    controller1.leftBumper.onRelease { robot.claw.open() }
    controller1.a.onTap { robot.intake.toggle() }               // a short press
    controller1.a.whileHeld(after = 300.milliseconds) {         // a long one; cancelled on release
        robot.intake.reverse()
    }
}
```

- **`onPress { }`** starts the code on every press. Pressing again while it still runs stops it and starts it over.
- **`onRelease { }`** does the same on every release.
- **`onTap(within = 300.milliseconds) { }`** starts the code when the button is released less than `within` after
  being pressed.
- **`whileHeld(after = ...) { }`** runs the code while the button is down, starting once it has been down for `after`,
  and stops it on release. Together with `onTap`, one button does one thing on a tap and another while held.

Handlers last as long as the code that set them up: handlers set up in `onInit` stop at START, handlers set up in
`onStart` work until the opmode ends (see [Robot code over time]({{ '/platform/robot-code-over-time/' | relative_url }})).
In INIT, buttons can pick an alliance or a starting position:

```kotlin
override suspend fun CoroutineScope.onInit() {
    controller1.x.onPress { alliance = Alliance.BLUE }
    controller1.b.onPress { alliance = Alliance.RED }
    everyTick { show("Alliance (X blue, B red)", alliance) }
}
```

Robot code can also wait for a button: `controller1.a.awaitPress()` and `awaitRelease()` return at the next press or
release. A press in the current loop, before the wait started, doesn't count.

## Triggers and sticks

A trigger is an `Axis`, with its `value` from 0.0 to 1.0. A stick is two axes, `x` and `y`, from -1.0 to 1.0: `x`
positive to the right and `y` positive when pushed forward. (FIRST's SDK has `y` the other way around.) A stick's
`magnitude` is how far it is pushed in any direction, from 0.0 to 1.0.

```kotlin
everyTick {
    robot.arm.power = controller2.leftStick.y.value
    robot.intake.speed = controller2.rightTrigger.value
}
```

Values close to zero read as zero, so a stick at rest doesn't creep: the deadband is the setting `controller1.deadband`
(and `controller2.deadband`), 0.05 by default. The rest of the range is stretched, so values still grow smoothly from
zero.

A trigger or a stick axis can be a button too, down while pushed further than a threshold:

```kotlin
controller1.rightTrigger.asButton(threshold = 0.5).onPress { robot.shooter.shoot() }
```

## Driving

The drivetrains take the sticks directly, in a loop:

```kotlin
everyTick { robot.drive.driveWith(controller1.leftStick, controller1.rightStick.x) }
```

See [Drivetrains]({{ '/platform/drivetrains/' | relative_url }}) for field-centric driving and tank controls.

## The SDK's gamepad

`controller1.gamepad` is FIRST's `Gamepad`, for what the platform doesn't cover, like rumble or the light on
PlayStation controllers:

```kotlin
// The drivers feel it when the intake gets full
var wasFull = false
everyTick {
    val full = robot.intake.isFull
    if (full && !wasFull) controller1.gamepad.rumble(300)
    wasFull = full
}
```
