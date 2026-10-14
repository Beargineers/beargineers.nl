---
title: Telemetry
summary: Driver Station lines, live graphs in the browser, logs, commands that start actions from a laptop, and the robot's web pages.
---

## Driver Station lines

Opmodes and parts of the robot show lines on the Driver Station:

```kotlin
show("Intake", intake.mode)                                   // Intake: ON
show("Pose") { localizer.pose }                               // computed only when lines are sent
show("Press A to start the intake")                           // a line of its own
```

A line is shown for the loop it was shown in, so code that shows something does it every loop, in `everyTick` or a
part's `update()`. Loops are much faster than anyone can read, so lines are only collected every
`telemetry.updateInterval` (100 ms): in the loops between, `show()` does nothing and values are never turned into text.
For a value that is costly to make, pass a block, like the pose above: it runs only in loops that collect lines.

The Driver Station's first lines are the platform's own: how long loops take, like *Loop: 2.4 ms average, 6.1 ms
longest*, and the battery, like *Battery: 12.6 V, lowest 12.1 V*.

## Graphs

`graph(name, value)` records a number for the robot's live graphs:

```kotlin
graph("shooter rpm", shooter.motor.velocity.inRpm)
graph("target rpm", targetRpm)
```

With the laptop on the robot's WiFi, <http://192.168.43.1:8080/bear/graphs> draws them, one chart per value, and shows
the Driver Station lines next to them. It needs no internet, and it keeps showing the last opmode after it stops.
Graphs keep the last 30 seconds or so of every value, at most one sample per 10 ms.

The platform graphs some values itself: `loop ms` (every loop's time, where a loop longer than 10 ms always shows up),
`battery`, the robot's pose (`x cm`, `y cm`, `heading deg`), and the path follower's errors and powers.

## Logs

`log("...")` writes a message to the robot's log, from a part with the part's path in front:

```kotlin
log("ball count reset")                                       // intake: ball count reset
```

The log also gets:

- named steps: `withName("Score")` writes when a step starts, finishes, fails or gets cancelled (see
  [Robot code over time]({{ '/platform/robot-code-over-time/' | relative_url }}));
- the Driver Station lines, once a second (`telemetry.logInterval`), so a match's log shows what the drivers saw;
- warnings of the robot's parts, settings problems, and the lowest battery voltage of the opmode.

The log is the SDK's robot log: `/sdcard/robotControllerLog.txt` on the Control Hub, and logcat in Android Studio,
where the platform's lines carry the tag `BearPlatform`. FIRST's match logs keep a copy for every match:
<http://192.168.43.1:8080/bear/matchLogs> downloads them as a zip and deletes them from the robot.

## Commands

For testing and tuning, an opmode can offer commands: actions started from a laptop, by buttons on the graphs page.

```kotlin
override suspend fun CoroutineScope.onStart() {
    command("forward 60 cm") { robot.follower.driveTo(Pose(60.cm, 0.cm, 0.degrees)) }
    command("open claw") { robot.claw.open() }
}
```

A command is offered as long as the code that offered it runs: commands offered in `onInit` go away at START, all of
them when the opmode ends. Starting a command that still runs starts it over, like pressing a button again. Someone
should be at the Driver Station, ready to stop the robot.

Scripts start commands too, with `POST http://192.168.43.1:8080/bear/command?name=forward%2060%20cm`, and read
everything the graphs page shows, as JSON, from `/bear/graphs/data`: together they let a script run a drive, read what
the robot did, change settings and try again. BearPlatform's `scripts/command.sh "forward 60 cm"` starts one.

## Battery

Every opmode watches the battery: the line on the Driver Station, a `battery` graph, a warning at INIT below
`battery.low` (12 V), and the lowest voltage of the opmode in the log. The battery is read a few times a second, as each
reading is a request to the hub of its own.

## The robot's web pages

The Control Hub serves these pages itself, so they work whenever the robot is on, with an opmode running or not:

| Page                                         | Does                                                                     |
|----------------------------------------------|--------------------------------------------------------------------------|
| `http://192.168.43.1:8080/bear/config`       | shows and edits the settings, live (see [Settings]({{ '/platform/settings/' | relative_url }})) |
| `/bear/config.properties`                    | the settings text: `GET` reads it, `POST` with `text/plain` replaces it  |
| `/bear/graphs`                               | live graphs, Driver Station lines, buttons for commands                  |
| `/bear/graphs/data`                          | the same as JSON, for scripts                                            |
| `/bear/command?name=...`                     | `POST` starts a command                                                  |
| `/bear/matchLogs`                            | downloads FIRST's match logs as a zip, and deletes them from the robot   |

Over USB instead of WiFi, run `adb forward tcp:8080 tcp:8080` and use `http://localhost:8080`.

## Scripts

BearPlatform's repository has scripts for these pages, in `scripts/`. They talk to the robot on its WiFi, or over USB
with `ROBOT=http://localhost:8080`:

| Script                                  | Does                                                                      |
|-----------------------------------------|---------------------------------------------------------------------------|
| `updateConfig.sh [config.properties]`   | sends a settings file to the robot; without one, the changed `config.properties` of the project |
| `command.sh "name"`                     | starts a command; without a name, lists the ones offered now              |
| `downloadMatchLogs.sh [folder]`         | downloads the match logs into a new folder, and deletes them from the robot |
