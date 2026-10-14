---
title: Sensors
summary: Switches and beam breaks that work like buttons, analog sensors, and distance and color sensors read without slowing the loop.
---

## Switches and beam breaks

Digital sensors are on or off: limit switches, touch sensors, beam breaks. They are read at the start of every loop,
with the gamepads, and work like gamepad buttons:

```kotlin
class Intake(parent: Component) : Component(parent) {
    val beam = DigitalInput(this, "beam", onWhenLow = true)   // most beam breaks pull their line low when broken
    var balls = 0

    suspend fun countBalls() = beam.onTurnOn { balls++ }
}
```

| Check in the moment | Wait or react                                              |
|---------------------|------------------------------------------------------------|
| `isOn`              | `awaitOn()`, `awaitOff()`                                  |
| `turnedOn`, `turnedOff`: in this loop | `onTurnOn { }`, `onTurnOff { }`          |
| `onFor`: how long it has been on | `whileOn(after = ...) { }`                    |

The handlers behave like the gamepad's (see [Gamepads]({{ '/platform/gamepads/' | relative_url }})).

Settings: `intake.beam.onWhenLow`, whether the sensor is on when its line reads low, like a REV touch sensor when
pressed and most beam breaks when broken; and `intake.beam.debounce`, how long a change has to last to count, for a
switch that bounces or a beam that flickers (`20ms`).

## Analog sensors

Potentiometers, absolute encoders and other sensors that answer with a voltage:

```kotlin
val armAngle = AnalogInput(this, "armAngle")
...
val turned = armAngle.fraction * 270.degrees
```

`voltage` is in volts, from 0 to `maxVoltage`, 3.3 V on REV hubs, and `fraction` is the share of it, from 0 to 1.

Digital and analog sensors and motor encoders come with the hub's bulk read, taken once per loop when something reads
them: everything in a loop sees the same values, and a sensor nothing reads costs nothing.

## Distance sensors

REV's 2m distance sensor and others the SDK sees as a distance sensor:

```kotlin
class Intake(parent: Component) : Component(parent) {
    val ball = DistanceSensor(this, "ball")
    private val holding = Hysteresis(on = 13.0, off = 17.0)

    val hasBall: Boolean get() = holding.value

    override fun update() {
        ball.distance?.let { holding.update(it.inCm) }
    }
}
```

`distance` is a `Distance`, or null when nothing is within `maxRange` (200 cm), when the sensor doesn't answer, and
before its first reading. Turning distances into decisions is the part's job: here a ball is in once closer than
13 cm, and out only once farther than 17, so the answer doesn't flicker (see [Control]({{ '/platform/control/' | relative_url }})).

## Color sensors

REV's Color Sensor V3 and others the SDK sees as a normalized color sensor:

```kotlin
val sample = ColorSensor(this, "sample", readDistance = true)
...
val color = sample.color
val purple = color != null && color.value > 0.01 && color.hue in 210.0..250.0
```

`color` is an `Rgb`: `red`, `green` and `blue` from 0 to 1, `hue` in degrees (red 0, yellow 60, green 120, blue 240),
`saturation` from 0 (gray) to 1, and `value`, the brightest channel. What a sensor sees is dim, so tell colors apart by
hue and saturation, which don't depend on the light, and check that something is there first, by `value` or the
distance: with nothing in front of the sensor, hue and saturation are noise. On one of our robots, DECODE's game
pieces held against the sensor read:

|                 | hue            | saturation     | value       | distance                 |
|-----------------|----------------|----------------|-------------|--------------------------|
| green artifact  | 162°           | 0.73           | 0.05–0.08   | 0.6 cm                   |
| purple artifact | 230°           | 0.48           | 0.04–0.05   | 0.7–1 cm                 |
| nothing         | ~174° (noise)  | ~0.46 (noise)  | 0.001       | 5–7 cm (the robot's own parts) |

REV's sensor lights what it sees with its own LED, so it sees colors well only within a centimeter or two. With
`readDistance`, each read also takes the sensor's distance: good for telling whether something is in front of it, not
for measuring. `distance` is null from `maxRange` (15 cm), the sensor's answer for nothing.

## Slow sensors

Distance and color sensors are I2C devices, slow to read: a read takes several messages to the hub, about 9 ms for a
2m distance sensor. So they are read on a thread of their own every `readInterval` (50 ms), and the loop never waits
for them: every loop takes the newest reading. `readInterval` is a setting, like `intake.ball.readInterval = 60ms`.

Every read takes the hubs' connection from motors and other sensors, so turn a sensor's `reading` off while nothing
needs it:

```kotlin
shooterBall.reading = isShooting
```

While off, it has no reading; turned on again, it is read right away and has none until that reading.
[Performance]({{ '/platform/performance/' | relative_url }}) explains the hubs' connection, and how to read other slow
devices this way.
