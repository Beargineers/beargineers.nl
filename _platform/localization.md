---
title: Localization
summary: Knowing where the robot is on the field, with goBILDA's Pinpoint and two odometry pods.
---

## The localizer

A localizer knows where the robot is: its `pose`, where its center is on the field and which way it faces, and its
`velocity`. Create it first on the robot, so it updates before anything that drives by it:

```kotlin
class MyRobot(host: RobotHost) : Robot(host) {
    val localizer = PinpointLocalizer(this)
    val drive = MecanumDrive(this)
}
```

Every loop, `localizer.pose` is where the robot is, as a `Pose` with `x`, `y` and `heading` in field coordinates (see
[Units and geometry]({{ '/platform/units-and-geometry/' | relative_url }})). Setting it tells the localizer where the
robot is, like at the start of autonomous:

```kotlin
robot.localizer.pose = Field2026.tile("A1BL").withHeading(0.degrees)
```

**The pose carries over from one opmode to the next**, while the robot app runs: TeleOp starts where autonomous ended,
unless it sets the pose itself. After the app restarts, the pose starts at zero.

The pose is graphed as `x cm`, `y cm` and `heading deg` on the robot's graphs page.

## goBILDA's Pinpoint

`PinpointLocalizer` uses goBILDA's Pinpoint odometry computer with two odometry pods, one measuring forward motion and
one sideways. The Pinpoint is named `pinpoint` in the robot configuration. It is read on a thread of its own every
`localizer.readInterval` (15 ms), so the loop never waits for it.

INIT calibrates the Pinpoint's gyro, which takes a quarter second with the robot standing still. Until then, and while
the Pinpoint reports a problem, like a pod not plugged in, the pose stays where it was and the Driver Station shows
the problem.

### Setting it up

1. **Where the pods are**, measured from the robot's center, with goBILDA's signs. Both are required:

   ```properties
   # The forward pod's distance left of the center; negative when it is right of it
   localizer.forwardPodLeft = -3.2cm
   # The strafe pod's distance forward of the center; negative when it is behind it
   localizer.strafePodForward = -0.9cm
   ```

2. **Which pods:** `localizer.pods = FOUR_BAR` (the default) or `SWINGARM`, goBILDA's two kinds. For other pods,
   `localizer.ticksPerMm` sets their resolution instead.
3. **Which way they count:** start an opmode that shows the pose, like the Quickstart's Driving, and push the robot by
   hand. x should grow when it moves forward, and y when it moves to its left. A pod counting the wrong way gets
   `localizer.forwardPodDirection = REVERSED` or `localizer.strafePodDirection = REVERSED`.
4. **The gyro, for the best accuracy:** turn the robot ten full turns by hand and look at the heading. If it reads a
   bit more or less than the turns made, set `localizer.yawScalar` to the old value times the real turns over the turns
   read; the Pinpoint's own is around 1.002.

These settings can be changed on the robot's settings page while the robot runs, and the Pinpoint takes them right
away.

## Field-centric driving

Field-centric TeleOp drives by the localizer's heading: see [Drivetrains]({{ '/platform/drivetrains/' | relative_url }}).

## Localizers of your own

A localizer is a part of the robot that extends `Localizer`, with a `pose` that can be read and set, and a `velocity`.
To carry the pose over between opmodes, it starts from `Localizer.lastPose` in `init()` and saves its pose there in
`stop()`.
