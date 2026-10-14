---
title: Robot code over time
summary: Steps that wait for each other, things that run alongside, and actions that take over a part of the robot.
---

## Steps, one after another

Robot code is mostly about time: drive there, then open the claw, then wait for the shooter to spin up. With
BearPlatform such code reads exactly like that, top to bottom:

```kotlin
override suspend fun CoroutineScope.onStart() {
    robot.follower.driveTo(SPIKE_1)                       // returns when the robot is there
    robot.claw.grab()                                     // returns when the claw holds on
    waitUntil(1.seconds) { robot.shooter.isReady }        // waits at most a second
    robot.shooter.shoot()
}
```

Functions that take time are `suspend` functions. While one waits, the opmode keeps looping: sensors are read, other
code runs, the Driver Station is updated. It is Kotlin's coroutines underneath, run by the robot's loop: everything
runs on one thread, one step at a time, so robot code never needs threads or locks.

### Waiting

| Code                                       | Waits                                                                 |
|--------------------------------------------|-----------------------------------------------------------------------|
| `nextTick()`                               | until the next loop                                                   |
| `waitUntil { intake.isFull }`              | until the condition is true, checking it once per loop                |
| `waitUntil(2.seconds) { intake.isFull }`   | the same, giving up after 2 seconds; returns whether it became true   |
| `delay(500.milliseconds)`                  | for a while                                                           |
| `withTimeout(3.seconds) { ... }`           | runs the block, and stops it after 3 seconds with an exception; `withTimeoutOrNull` returns null instead |
| `button.awaitPress()`                      | until a button is pressed (see [Gamepads]({{ '/platform/gamepads/' | relative_url }})) |

A loop that waits for something does `nextTick()` in every round:

```kotlin
suspend fun Arm.moveTo(angle: Angle) = takeOver("move to $angle") {
    while (abs(motor.position - angle) > 2.degrees) {
        motor.power = pid.update((angle - motor.position).inDegrees)
        nextTick()
    }
    motor.power = 0.0
}
```

Never wait with `Thread.sleep` or a loop without `nextTick()`: the robot's whole loop would wait with it, and nothing
else, not even the gamepads, would be read.

## Things at the same time

`race` runs steps at the same time and goes on with the first one to finish, stopping the others:

```kotlin
// Drives toward the spike, but stops as soon as the intake is full
race({ robot.follower.driveTo(SPIKE_1) }, { waitUntil { robot.intake.isFull } })
```

`launch` starts something that runs alongside the code after it, and `coroutineScope` waits until everything started
inside it is done:

```kotlin
coroutineScope {
    launch { robot.arm.moveTo(90.degrees) }    // the arm moves while the robot drives
    robot.follower.driveTo(BASKET)
}                                               // here both are done
```

### Every loop

`everyTick { }` sets up something to do once every loop, like driving with the sticks or showing a line, and returns
right away, like a button handler: code after it runs too.

```kotlin
override suspend fun CoroutineScope.onStart() {
    everyTick { show("Pose") { robot.localizer.pose } }     // shows the pose while the robot drives
    robot.follower.driveTo(SPIKE_1)
    robot.follower.driveTo(BASKET)
}
```

The block runs once right away, then once per loop. It lasts as long as the code that set it up: one set up in
`onInit` stops at START, one in `onStart` runs until the opmode ends, one in a button handler stops when the handler is
started over. `everyTick` returns a `Job`, and `job.cancel()` stops it earlier.

## Named steps

`withName` writes a step's start and end to the robot's log, which makes logs of a match readable:

```kotlin
suspend fun MyRobot.scoreSample() = withName("Score sample") {
    follower.driveTo(BASKET)
    arm.moveTo(120.degrees)
    claw.open()
}
```

The log then has *Score sample: started*, and *finished*, *cancelled* or *failed* with the reason.

## Actions

Most parts of the robot can only do one thing at a time: a drivetrain can't drive to two places, and an arm can't go up
and down at once. Code that makes a part do something over time takes the part over:

```kotlin
suspend fun Arm.moveTo(angle: Angle) = takeOver("move to $angle") {
    // ...
}
```

`takeOver` stops whatever action had the part before, waits until it has cleaned up, and runs the new one: **the latest
request wins**. Pressing "score" while the robot drives to the spike stops the drive to the spike. The stopped action
ends with a `Preempted` exception, which stops the code that ran it too, so a step after it doesn't run as if the
action had finished. The text, like `"move to $angle"`, is only a label for the log and for `currentAction`.

On any part:

- `isBusy` tells whether an action has it, and `currentAction` which one;
- `cancelAction()` stops the action that has it.

A few things to know:

- An action that takes over the same part again from inside, like `driveTo` inside `drivePath`, keeps control and
  stops nothing. So do coroutines it starts with `launch` or `everyTick`: they are part of the action, and the action
  keeps the part until they are done too.
- Two parts are two separate things to take over: the arm moving doesn't stop the drivetrain.
- The sticks stop an action that drives the robot when pushed past halfway (see
  [Drivetrains]({{ '/platform/drivetrains/' | relative_url }})).

### Cleaning up

An action that leaves the hardware running, like a motor holding a speed, cleans up in `finally`, which runs however
the action ends: finished, stopped by another action, or by the end of the opmode.

```kotlin
suspend fun Flywheel.holdSpeed() = takeOver("hold speed") {
    try {
        while (true) {
            motor.power = feedforward.power(targetRpm) + pid.update(targetRpm - motor.velocity.inRpm)
            nextTick()
        }
    } finally {
        motor.power = 0.0
    }
}
```

Such an action loops by itself with `nextTick()` rather than with `everyTick`: `everyTick` returns right away, so a
`finally` after it would run right away too, while the loop goes on.

## Handlers and their lifetime

Button handlers (`onPress` and the others, see [Gamepads]({{ '/platform/gamepads/' | relative_url }})), `everyTick`
blocks and commands all live as long as the code that set them up. That is what makes the opmode's phases clean:

- set up in `onInit`, they stop at START;
- set up in `onStart`, they run until the opmode ends;
- set up inside an action or a handler, they stop with it.

## Testing robot code

Robot code runs the same on a laptop as on the robot, with a clock the test controls, so steps that take seconds are
tested in milliseconds. The platform's own tests do this; see the BearPlatform repository for examples.
