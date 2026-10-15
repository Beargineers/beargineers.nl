---
title: Getting started
summary: Start a robot project from the Quickstart, or add BearPlatform to a project you already have.
---

## What you need

- **Android Studio**, Narwhal 3 Feature Drop or later, as FIRST's SDK 12.0 asks.
- **A robot with a REV Control Hub.** The Quickstart's starter code drives a mecanum robot with goBILDA Pinpoint
  odometry; for other robots, keep the project and change the robot class.
- **Some Kotlin.** Robot code with BearPlatform is Kotlin. [Kotlin for FTC]({{ '/learn/kotlin/kotlin-for-ftc/' | relative_url }})
  is a good first path.

## Start from the Quickstart

The [Quickstart](https://github.com/Beargineers/Quickstart) is FIRST's FtcRobotController with a `TeamCode` written
in Kotlin on BearPlatform, and without FIRST's Java samples, which don't use it.

1. **Fork** the Quickstart on GitHub, then clone your fork. A fork keeps the way to updates open (see
   [Updates](#updates)).
2. Open the project folder in Android Studio and let it sync.
3. In the robot configuration on the Driver Station, name the drive motors `leftFront`, `rightFront`, `leftBack` and
   `rightBack`, and the Pinpoint `pinpoint`.
4. Measure where the odometry pods are, and set `localizer.forwardPodLeft` and `localizer.strafePodForward` in
   `TeamCode/src/main/res/raw/config.properties` (see [Localization]({{ '/platform/localization/' | relative_url }})).
5. Connect to the robot and run **TeamCode** in Android Studio, which installs the app.
6. On the Driver Station, start **Driving**:
   - With the robot on a box, wheels off the ground, press **Back**: each wheel turns for a second and should turn so
     it would push the robot forward. The left motors start reversed (`R`), the right ones forward (`F`): flip one
     that turns backward, like `drive.leftFront.direction = F`. A different wheel turning than the Driver Station
     names means a motor is plugged into another port than configured.
   - Push the robot by hand and watch **Pose**: x should grow when it moves forward, y when it moves to its left.
   - Drive! The left stick moves, the right stick turns.
7. Put the robot on the floor with some room, facing away from you, and start **Auto**: it drives a 60 cm square.

Settings like a motor's direction don't need a rebuild: with the laptop on the robot's WiFi,
<http://192.168.43.1:8080/bear/config> changes them live (see [Settings]({{ '/platform/settings/' | relative_url }})).

### What's in the project

| In `TeamCode/src/main`                       | What                                                                       |
|----------------------------------------------|----------------------------------------------------------------------------|
| `java/.../MyRobot.kt`                        | The robot: the parts it is made of                                         |
| `java/.../Driving.kt`                        | The TeleOp                                                                 |
| `java/.../Auto.kt`                           | An autonomous                                                              |
| `java/.../Claw.kt`                           | An example part to start your own from; not on the robot until you add it  |
| `res/raw/config.properties`                  | The robot's settings                                                       |

Rename them as you like: the opmodes' names on the Driver Station come from their `@TeleOp` and `@Autonomous`
annotations.

## Add BearPlatform to a project you have

A project made from FIRST's FtcRobotController needs three things.

The Kotlin plugin, in the `plugins` block of the project's `build.gradle`, next to the Android ones:

```groovy
plugins {
    id 'com.android.application' version '8.13.2' apply false
    id 'com.android.library' version '8.13.2' apply false
    id 'org.jetbrains.kotlin.android' version '2.2.20' apply false
}
```

In `TeamCode/build.gradle`: the plugin applied, Kotlin set to the Java 8 bytecode the Control Hub runs, and the
library:

```groovy
apply plugin: 'com.android.application'
apply plugin: 'org.jetbrains.kotlin.android'

// ...what was there before...

kotlin {
    compilerOptions {
        jvmTarget = org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_1_8
    }
}

dependencies {
    implementation project(':FtcRobotController')
    implementation 'nl.beargineers.platform:ftc:0.2.0'
}
```

And a settings file, `TeamCode/src/main/res/raw/config.properties`, which may start empty. Java opmodes and BearPlatform
opmodes can live side by side in one project.

## Updates

- **BearPlatform:** change the version in `TeamCode/build.gradle`. Release notes say what changed and how to update
  robot code.
- **FTC SDK and the Quickstart:** when FIRST releases a new SDK, it comes to the Quickstart too. Pull it into your fork:

  ```
  git pull https://github.com/Beargineers/Quickstart.git master
  ```
