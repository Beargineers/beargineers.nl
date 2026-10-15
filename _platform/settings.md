---
title: Settings
summary: Values that change without rebuilding the app, declared where the code uses them and edited live on the robot.
---

## Declaring a setting

Servo positions, PID gains, motor directions, positions on the field: values like these get tuned again and again.
In BearPlatform they are settings, declared right where the code uses them:

```kotlin
class Claw(parent: Component) : Component(parent) {
    val openPosition by config(0.6)
    val closedPosition by config(0.3)
    val closeTime by config(300.milliseconds)
}
```

`config(0.6)` makes `openPosition` a setting with the default 0.6. If the robot's settings file has a value for it,
that value is used instead. Read the property as any other: it always has the current value, and changes made on the
robot take effect right away, even while an opmode runs.

- **Required settings** have no default: `val goal by config<Point>()`. INIT fails until the settings file sets it.
- **Optional settings** may be left out: `val cameraOffset by optionalConfig<Distance>()` is null unless the file sets
  it.

## Keys

Every setting has a key. In a part of the robot, it is the part's path and the property's name: `openPosition` in the
`claw` is `claw.openPosition`, and in a part of the claw, `claw.wrist.openPosition`. Settings of the robot class itself
are `robot.<name>`. Elsewhere, the key starts with the name of the class it is declared in, like `AutoPositions.START`,
and top-level properties have just their name.

`config(0.6, key = "...")` picks the key by hand. A class that isn't a part can pick the start of its keys by
implementing `ConfigScope`.

## The settings file

The settings are in the app's `res/raw/config.properties` (in the Quickstart, `TeamCode/src/main/res/raw/`), one
`key = value` per line:

```properties
# Comments go on their own lines: text after a value becomes part of the value
claw.openPosition = 0.62
claw.closeTime = 250ms
drive.leftFront.direction = F
follower.forwardGains = p=0.03, i=0.002, d=0.005
```

Only settings that differ from their defaults need a line. Every opmode loads the file at INIT and checks it against the
settings its robot declares:

- a value that can't be read, like `claw.closeTime = soon`, fails INIT, listing every such value with what was expected;
- a key nothing uses shows on the Driver Station until START, usually a typo, with a *did you mean* hint.

A key that only other opmodes of the app use shows up as unused too, since an opmode knows only its own robot's
settings.

## Values

Values are written the way people write them, with units:

| Type                     | Written as                                                  | Plain numbers are |
|--------------------------|-------------------------------------------------------------|-------------------|
| number (`Double`)        | `0.25`                                                      |                   |
| whole number (`Int`)     | `3`                                                         |                   |
| true/false (`Boolean`)   | `true`, `yes`, `on`, `1` or `false`, `no`, `off`, `0`       |                   |
| text (`String`)          | anything                                                    |                   |
| `Duration`               | `300ms`, `1.5s`                                             | (a unit is needed)|
| `Distance`               | `30cm`, `12in`, `5mm`, `1.5tiles`, `2ft`                    | cm                |
| `Angle`, `Heading`       | `90deg`, `1.57rad`, `0.25rot`                               | degrees           |
| `LinearVelocity`         | `30cm/s`, `12in/s`                                          | cm/s              |
| `AngularVelocity`        | `90deg/s`, `435rpm`                                         | degrees/s         |
| `Point`                  | `10cm, -20cm`, or a tile like `F6TR`                        | cm                |
| `Pose`                   | `10cm, -20cm, 90deg`, or a tile and a heading like `F6BL:135` | cm, degrees     |
| `PidGains`               | `p=0.02, i=0, d=0.001`, or `0.02, 0, 0.001`                 |                   |
| `Feedforward`            | `kS=0.04, kV=0.0085, kA=0`                                  |                   |
| any `enum`               | its name, or a unique beginning: `F` for `FORWARD`          |                   |

Named numbers, like PID gains, can be left out: `p=0.02, d=0.001` has no I. Tile codes need the season's field: set
`Config.field = Field2026` before settings are read, like in the robot's constructor (see
[Units and geometry]({{ '/platform/units-and-geometry/' | relative_url }})).

## Several robots in one file

An app that serves several robots (see [Robots and parts]({{ '/platform/robots/' | relative_url }})) has one settings
file with a section for each robot, named after its robot configuration on the Driver Station:

```properties
# Every robot's settings
AutoPositions.START = F6BL:135

[competition]
localizer.forwardPodLeft = -3.2cm
localizer.strafePodForward = -0.9cm

[practice, practice2]
localizer.forwardPodLeft = 7cm
localizer.strafePodForward = 2cm

[*]
# every robot's again
```

Settings before the first section are every robot's; a section's settings apply only to the robots it names, and
override every robot's. `[*]` goes back to every robot's. Only every robot's settings and the active robot's sections
are checked, so one robot's settings never show up as unused on the other.

## Editing on the robot

With the laptop on the robot's WiFi, <http://192.168.43.1:8080/bear/config> shows every setting the robot has, with its
default, which section each value comes from, and problems like typos. Edits saved there take effect right away, even
in a running opmode, and stay on the robot until the next install of the app. The page works with no opmode running,
as the Control Hub itself serves it.

Edits on the robot are lost when a new build is installed, which starts from the build's own settings file. To keep
them, use the page's **Download** link and copy the file into the project.

From a script, `POST http://192.168.43.1:8080/bear/config.properties` with the text as `text/plain` replaces the
settings, and `GET` reads them. Over USB, run `adb forward tcp:8080 tcp:8080` and use `localhost:8080` instead. The
Gradle task `sendConfig` sends the project's `config.properties` this way, from Android Studio or a terminal (see
[Gradle tasks]({{ '/platform/telemetry/#gradle-tasks' | relative_url }})).

## Settings of your own types

Config reads the types above, and every `enum`. Teach it another type with `Config.registerType`, giving a name for
messages, how to read the text, and how to write the value back. Register it before any setting of the type is
declared, like at the start of the robot's constructor:

```kotlin
data class Range(val min: Double, val max: Double)

Config.registerType(ConfigType("range", parse = { text ->
    val (min, max) = text.split("..").map { it.trim().toDouble() }
    Range(min, max)
}, format = { "${it.min}..${it.max}" }))

val armLimits by config(Range(0.1, 0.9))       // arm.armLimits = 0.15..0.85
```

A `parse` that can't read the text throws an exception with a short explanation, which the Driver Station shows.
