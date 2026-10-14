---
title: Units and geometry
summary: Distances, angles and speeds that carry their units, poses on the field, and the season's field.
---

## Units

A bare number like `30` could be centimeters, inches or degrees. BearPlatform's numbers carry their units:

```kotlin
val reach = 30.cm + 4.inch                   // a Distance
val turn = 90.degrees                        // an Angle
val speed = 2.tiles / 1.5.seconds            // a LinearVelocity
val spin = 435.rpm                           // an AngularVelocity
```

They are read back in the unit wanted: `reach.inCm`, `turn.inRadians`, `spin.inDegreesPerSecond`. Mixing them up
doesn't compile: a distance can't be added to an angle. They cost no more than plain `Double`s.

| Type              | Made with                                              | Read with                                    |
|-------------------|--------------------------------------------------------|----------------------------------------------|
| `Distance`        | `10.cm`, `5.mm`, `1.5.meters`, `4.inch`, `2.tiles`     | `inCm`, `inMm`, `inMeters`, `inInches`, `inTiles` |
| `Angle`           | `90.degrees`, `1.5.radians`, `0.25.rotations`          | `inDegrees`, `inRadians`, `inRotations`      |
| `Heading`         | `135.degrees.toHeading()`                              | `inDegrees`, `inRadians`                     |
| `LinearVelocity`  | `50.cm.perSecond`, `distance / duration`               | `inCmPerSecond`, `inInchesPerSecond`         |
| `AngularVelocity` | `90.degrees.perSecond`, `435.rpm`, `angle / duration`  | `inDegreesPerSecond`, `inRadiansPerSecond`, `inRpm` |
| `Duration`        | `300.milliseconds`, `1.5.seconds` (Kotlin's own)       | `inWholeMilliseconds`, `toDouble(DurationUnit.SECONDS)` |

A tile is a standard FTC field tile, 24 inches. Arithmetic works as expected: `speed * 2.seconds` is a distance,
`reach / 2` is half of it, `reach / 1.cm` is a plain number, and `abs`, `sin`, `cos`, `hypot` and `atan2` take units
too.

### Angles and headings

An `Angle` is an amount of turning, and never wraps around: `270.degrees + 180.degrees` is 450°, which is what a
turret, an arm or counting rotations needs. Positive is counterclockwise.

A `Heading` is a direction on the field, like the way the robot faces, and always stays within (-180°, 180°]:

```kotlin
val facing = robot.localizer.pose.heading
val turned = facing + 90.degrees              // wraps around by itself
val turn = target.heading - facing            // the shortest turn there: an Angle within (-180°, 180°]
```

So turning the robot never needs angle wrapping by hand. Most places that take a heading, like
`Pose(x, y, 135.degrees)`, take an angle too and convert it.

## Points, poses and offsets

| Type          | Is                                                                                         |
|---------------|--------------------------------------------------------------------------------------------|
| `Point`       | a place on the field: `Point(30.cm, -10.cm)`                                               |
| `Pose`        | a place and the way the robot faces there: `Pose(30.cm, -10.cm, 90.degrees)`              |
| `RobotOffset` | a move or a place in the robot's own frame: `forward`, `right`, and `turn` counterclockwise |
| `PoseVelocity`| how fast the robot moves along field x and y, and turns                                   |

```kotlin
val next = pose + RobotOffset(forward = 30.cm, turn = 90.degrees)  // 30 cm ahead, then a quarter turn left
val camera = (pose + RobotOffset(forward = 20.cm, right = -5.cm)).point   // where a camera mounted there is
val toGo = target.relativeTo(pose)            // forward, right and turn the robot still has to go
val face = pose.headingTo(goal)               // the heading that faces the goal from here
val far = pose.distanceTo(goal)
```

A point is also a vector on the field: `b - a` is the move from `a` to `b`, with its `length` and `direction`. Points
and poses can be `shiftBy` field distances, poses `turnBy` an angle, and `lerp` goes part of the way to another one.

## Field coordinates

Poses on the field use FTC's field coordinate system:

- the origin is the center of the field;
- heading 0 faces +x, and +y is 90° counterclockwise from it;
- angles and turns are positive counterclockwise.

Which wall +x points to changes from season to season, so each season's code describes its field:

```kotlin
// DECODE's field: +x points toward the audience, the red drivers stand at the audience's left, and
// blue's half of the field is a mirror image of red's across the x axis
val DecodeField = Field(
    xAxis = FieldDirection.TOWARD_AUDIENCE,
    symmetry = FieldSymmetry.FLIP_Y,
    redStation = FieldDirection.AUDIENCE_LEFT,
)
```

| `Field` property     | Is                                                                                       |
|----------------------|------------------------------------------------------------------------------------------|
| `xAxis`              | where +x points, as the audience sees the field                                          |
| `symmetry`           | how one alliance's side maps onto the other's: `FLIP_Y`, `FLIP_X` or `ROTATE_180`        |
| `referenceAlliance`  | the alliance poses in code are written for, red by default                               |
| `redStation`, `blueStation` | the walls the drivers stand at; field-centric driving needs it                    |
| `tileSize`           | 24 inches                                                                                |

Then:

- **`field.forAlliance(pose, alliance)`** converts a pose written for the reference alliance to the other's, so
  autonomous paths are written once;
- **`field.driversFace(alliance)`** is the way the alliance's drivers look, for field-centric driving;
- **`field.contains(point)`** and **`field.clamp(point)`** keep points within the walls.

### Tiles

Tiles are named the way the audience sees the field: columns A to F from left to right, rows 1 to 6 from the
audience's side to the far side. `field.tile("D4")` is the center of D4, and a suffix picks a spot on the tile, as the
audience sees it: `T`op, `B`ottom, `L`eft, `R`ight, `C`enter, one or two of them in any order. `tile("F6TR")` is the top
right corner of F6, and `tile("C3L")` the middle of its left edge.

```kotlin
val start = DecodeField.tile("F6BL").withHeading(135.degrees)
```

Settings can be tiles too, like `AutoPositions.START = F6BL:135`, once the season's field is set with
`Config.field = DecodeField` (see [Settings]({{ '/platform/settings/' | relative_url }})).

## Robot coordinates

Moves relative to the robot use `RobotOffset`: `forward` along its heading, `right` to its right side, and `turn`
counterclockwise. The drivetrains take powers the same way: forward, right, and turn counterclockwise.
