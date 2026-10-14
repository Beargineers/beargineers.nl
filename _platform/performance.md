---
title: Performance
summary: What makes loops slow, how the platform keeps them fast, and how to find where the time goes.
---

Most robots never need this page: the platform's defaults keep loops fast. It is for when the Driver Station says
loops are slow, or a robot has many sensors.

## The hubs' one connection

Every command to the hubs holds their one connection until the hub answers: a motor power, a servo position, a bulk
read, each step of an I2C read. There is one connection for the Control Hub and an Expansion Hub together, and each
command holds it for about 1.4 ms. That, not the code, is what makes loops slow:

| Command                                   | Takes about                              |
|-------------------------------------------|------------------------------------------|
| a motor power                             | 1.6 ms                                   |
| a bulk read of one hub                    | 2 ms                                     |
| a Pinpoint read                           | 5.5 ms                                   |
| a 2m distance sensor read                 | 9 ms                                     |
| a color sensor read (with the distance)   | 11 ms (16 ms), on an Expansion Hub       |

An Expansion Hub is reached through the Control Hub, over RS485, and costs more: on one of our robots, a servo write to
it took 5.5 ms, and its I2C reads took nearly twice as long as the Control Hub's. **Put the devices written or read
most often on the Control Hub.**

## What the platform does about it

- **One bulk read per hub per loop.** Motor encoders, digital and analog inputs come from the hub's bulk read, taken
  again every loop, but only when something reads it: everything in a loop sees the same values, and a hub nothing
  reads costs nothing.
- **Only changes are sent.** A motor's power is sent only when it differs from what the hub has by at least
  `powerTolerance` (0.005), so a PID nudging the power every loop doesn't write every loop; stopping and full power are
  always sent exactly. Servo positions and lights are sent only when they change.
- **Slow sensors are read on a thread of their own.** I2C sensors, like the Pinpoint, distance and color sensors, are
  read by the device reader, one at a time, each at its own interval, so the loop never waits for them. Each loop takes
  their newest readings.
- **Lines are collected every 100 ms**, not every loop, and values are turned into text only then
  (`telemetry.updateInterval`).
- **Loops don't spin.** A loop that takes less than `loop.minTime` (5 ms) sleeps for the rest: looping faster than
  sensors change only takes CPU from the threads that talk to the hubs.
- **The battery is read a few times a second**, not every loop: each reading is a request of its own.

## Reading slow sensors

Every read of the device reader takes connection time from everything else, so:

- read as rarely as the robot can do with: each sensor's `readInterval` is a setting;
- pause sensors nothing needs: `ballSensor.reading = isIntaking`.

When the reads take more than `reader.budget` percent of the time (50%), the Driver Station says so while they do,
naming the reads that take the most, and the log gets the first warning and any clearly worse one. Only what lasts
three seconds counts, as a slow read now and then is normal: a 2m sensor's first read takes up to 300 ms. On one of our
robots, the Pinpoint every 15 ms took about 37% of the time, and each distance sensor every 50 ms about 17%.

The Pinpoint's `readInterval` is 15 ms: on a mecanum robot, path following was as fast and as accurate reading it every
10, 15 or 20 ms, with the connection 55, 37 and 28% busy reading.

### Slow devices of your own

A sensor the platform has no class for, but slow to read, is read the same way by extending `BackgroundSensor`: say how
to read it, and offer what the reading means. This is roughly how the platform's own `DistanceSensor` is made:

```kotlin
class LaserRange(parent: Component, name: String) :
    BackgroundSensor<Double>(parent, name, readInterval = 40.milliseconds) {

    val device by hardware<Rev2mDistanceSensor>(name)

    /** On the device reader's thread: keep it to the read itself */
    override fun read(): Double = device.getDistance(DistanceUnit.CM)

    /** This loop's distance, or null while not reading */
    val distance: Distance? get() = value?.cm
}
```

`read()` runs on the device reader's thread, so it only reads; everything else happens on the loop's thread. The part
gets `reading` and the `readInterval` setting with it.

## Finding where the time goes

`telemetry.profile = yes` graphs where each loop's time goes, on the graphs page:

| Graph                         | Is                                                                      |
|-------------------------------|-------------------------------------------------------------------------|
| `loop ms`                     | every loop's time; always on                                            |
| `time <part> ms`              | each of the robot's top-level parts' updates, robot code, motor writes and how many per loop, the battery, telemetry |
| `alloc <part> KB`             | what each part allocates, which makes garbage to collect                |
| garbage collection            | how often the app collects garbage, and how long it takes               |
| `reader busy %`               | how busy the device reader keeps the connection, with each read's time and rate |

With profiling on, a loop that takes unusually long is logged with where its time went, and so are long pauses
between loops. Profiling costs a few clock readings per loop, so leave it off for matches.
