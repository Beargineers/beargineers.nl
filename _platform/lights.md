---
title: Indicator lights
summary: Lights that tell the drivers what the robot is up to, with signals that take turns by priority.
---

## Signals

A light on the robot can tell the drivers a lot: the intake is full, a target is in view, the shooter is ready. Many
parts of the robot have something to say, so each asks through a signal of its own, and the light shows the one with
the highest priority:

```kotlin
val fullSignal = robot.light.signal(priority = 10)
val newBallSignal = robot.light.signal(priority = 40)

everyTick {
    fullSignal.show(if (robot.intake.balls == 3) RgbColor.GREEN else RgbColor.OFF)
}
// A ball came in: blinks orange, hiding the green for a moment
robot.intake.beam.onTurnOn { newBallSignal.blink(RgbColor.ORANGE, times = 3) }
```

A signal shows something until told otherwise, or:

| Call                                          | Shows                                                        |
|-----------------------------------------------|--------------------------------------------------------------|
| `show(state)`                                 | `state`, until told otherwise                                |
| `show(state, lasting = 2.seconds)`            | `state` for a while, then nothing                            |
| `blink(state, period = 250.milliseconds, times = 3)` | `state` and off in turns; without `times`, until told otherwise |
| `show(pattern)`                               | states one after another, see below                          |
| `clear()`                                     | nothing                                                      |

Of the signals showing something, the highest priority wins; of equal ones, the one asked last. Asking a signal again
for what it shows already changes nothing, so a blink asked for every loop keeps its rhythm instead of starting over.
`isShowing` tells whether a signal has something to show, whether or not a higher one hides it.

A `Pattern` shows states one after another, each for its time, over and over or a number of `times`:

```kotlin
private val chase = Pattern("G00" to 150.milliseconds, "0G0" to 150.milliseconds, "00G" to 150.milliseconds)
...
leds.signal().show(chase)
```

Make a pattern once, in a property, rather than in every loop.

Lights are off from INIT, and again when the opmode stops. Only changes are sent to the hub, as each is a command to
it.

## goBILDA's RGB light

goBILDA's RGB indicator light plugs into a servo port and is configured as a servo, named `light` unless told
otherwise:

```kotlin
val light = RgbLight(this)
```

Its colors are `RgbColor.OFF`, `RED`, `ORANGE`, `YELLOW`, `SAGE`, `GREEN`, `AZURE`, `BLUE`, `INDIGO`, `VIOLET` and
`WHITE`. A change of color is one command to the hub, like a servo's position.

## REV's LEDs

REV's red/green LED indicators each take a digital port pair, configured as two LED devices: `<name>red` on the port
that lights red, `<name>green` on the other. `RevLeds` is a row of them, `led1`, `led2` and `led3` unless told
otherwise:

```kotlin
val leds = RevLeds(this)                                          // led1red, led1green, led2red, ...
val twoLeds = RevLeds(this, "status", listOf("left", "right"))
```

Its states are strings with a letter per LED: `R` red, `G` green, `A` amber (both), `0` off. `"GG0"` lights the first
two green. Only LEDs that change are sent.

## Lights of your own

Any light can be an indicator: extend `Indicator<S>`, with `S` what it shows, and say how to `display` a state. The
signals, priorities, blinking and patterns come with it.

```kotlin
class Blinkin(parent: Component) : Indicator<BlinkinPattern>(parent, "blinkin", BlinkinPattern.BLACK) {
    val driver by hardware<RevBlinkinLedDriver>("blinkin")

    override fun display(state: BlinkinPattern) = driver.setPattern(state)
}
```
