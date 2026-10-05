---
title: Kotlin Practice
published: true
summary: Ten beginner exercises to learn Kotlin by writing small, FTC-inspired programs.
---

Learn a little Kotlin, write a little code, and check what happens. These exercises start with the basics and gradually combine them into useful robot-programming ideas. Allow around 10–20 minutes per exercise; take longer when a concept is new.

## Before you start

Open [Kotlin Playground](https://play.kotlinlang.org/) and replace its example with one starter below at a time. Choose **JVM** if asked for a target, then press **Run**. You can also use a Kotlin project in your IDE. No robot, Android Studio, or FTC SDK is needed: all inputs are ordinary variables and all results are printed text.

If Kotlin is completely new to you, begin with the tour linked on [Kotlin for FTC]({{ '/learn/kotlin/kotlin-for-ftc/' | relative_url }}). The starters below deliberately leave work for you to do. Change the input values and run again to check every example, rather than printing the expected answers directly.

The robot situations are simplified practice examples. The numbers and rules here are chosen for learning, rather than taken from a competition game.

## Choose an exercise

1. [Introduce your team](#1-introduce-your-team) — variables and strings
2. [Estimate wheel travel](#2-estimate-wheel-travel) — numbers and arithmetic
3. [Stop at an obstacle](#3-stop-at-an-obstacle) — `if` and `else`
4. [Check whether the robot is ready](#4-check-whether-the-robot-is-ready) — Boolean logic
5. [Choose a starting route](#5-choose-a-starting-route) — `when`
6. [Add a slow driving mode](#6-add-a-slow-driving-mode) — functions
7. [Print a countdown](#7-print-a-countdown) — loops and ranges
8. [Review practice scores](#8-review-practice-scores) — lists and totals
9. [Handle a missing reading](#9-handle-a-missing-reading) — nullable values
10. [Describe a robot](#10-describe-a-robot) — data classes

## 1. Introduce your team

**Practice:** `val`, `var`, `String`, `Int`, and string templates.

Print the team name and number on one line. Then increase `completedLaps` by one and print the new lap count. Use the variables in your messages so changing a value changes the output too.

```kotlin
fun main() {
    val teamName = "Beargineers"
    val teamNumber = 27628
    var completedLaps = 2

    // Print the team introduction.
    // Increase the lap count, then print it.
}
```

**Check your result:**

```text
Team 27628: Beargineers
Completed laps: 3
```

**Hint:** Put `$teamName` inside a string to insert its value. A `val` cannot be reassigned; a `var` can.

**Try more:** Change `completedLaps` to a `val` and try to increase it. Read the error, then explain why the team name and the lap count use different declarations.

## 2. Estimate wheel travel

**Practice:** `Double`, multiplication, and units.

A wheel travels one circumference in one complete rotation. Calculate how far it travels in `rotations` rotations and print the distance in centimetres. Assume the wheel does not slip.

```kotlin
fun main() {
    val circumferenceCm = 31.4
    val rotations = 2.5

    // Calculate distanceCm and print it.
}
```

**Check your result:** `Distance: 78.5 cm`. With `rotations = 0.0`, the distance should be `0.0 cm`; with `rotations = 1.0`, it should be `31.4 cm`.

**Hint:** Use `*` to multiply. Decimal literals such as `2.5` have type `Double`.

**Try more:** Calculate rotations from a distance of `157.0` cm. You should get `5.0`. Then compare `5 / 2` with `5.0 / 2.0` and explain the difference.

## 3. Stop at an obstacle

**Practice:** comparisons and `if` / `else`.

A simulated robot should print `STOP` when an obstacle is **20 cm away or closer**, and `DRIVE` otherwise. Assume the distance is non-negative.

```kotlin
fun main() {
    val distanceCm = 35

    // Decide whether to print STOP or DRIVE.
}
```

**Check your result:** Run the program with each input below. The value exactly on the boundary matters.

| `distanceCm` | Expected output |
| --- | --- |
| `35` | `DRIVE` |
| `21` | `DRIVE` |
| `20` | `STOP` |
| `8` | `STOP` |
| `0` | `STOP` |

**Hint:** `<=` means “less than or equal to”. Only one of the two branches should run.

**Try more:** Store the stopping distance in a variable so you can change it in one place.

## 4. Check whether the robot is ready

**Practice:** `Boolean`, `&&`, and `!`.

Print `READY` only when the robot has been initialized **and** stop has **not** been requested. Print `WAIT` in every other case.

```kotlin
fun main() {
    val isInitialized = true
    val stopRequested = false

    // Check both conditions before printing READY or WAIT.
}
```

**Check your result:**

| `isInitialized` | `stopRequested` | Expected output |
| --- | --- | --- |
| `true` | `false` | `READY` |
| `false` | `false` | `WAIT` |
| `true` | `true` | `WAIT` |
| `false` | `true` | `WAIT` |

**Hint:** `&&` requires both sides to be true. `!` reverses a Boolean value.

**Try more:** Add `hasGamePiece`. Print `READY` only when the original conditions hold and the robot also has a game piece.

## 5. Choose a starting route

**Practice:** `when` and a default branch.

Use `startPosition` to select a printed instruction. This is a pretend route selector; it does not move a robot.

```kotlin
fun main() {
    val startPosition = "left"

    // Use when to print the instruction for this position.
}
```

**Check your result:**

| `startPosition` | Expected output |
| --- | --- |
| `"left"` | `Turn right` |
| `"centre"` | `Drive forward` |
| `"right"` | `Turn left` |
| `"unknown"` (or any other value) | `Stay still` |

**Hint:** Start with `when (startPosition)`. Each branch uses `->`; an `else` branch handles unrecognized values.

**Try more:** Make `"LEFT"` and `"Left"` behave like `"left"` using `.lowercase()`.

## 6. Add a slow driving mode

**Practice:** function parameters, return values, and reusing logic.

Complete `drivePower`. In normal mode it should return the joystick value unchanged. In slow mode it should return **half** that value. For this exercise, inputs are always between `-1.0` and `1.0`; negative values represent driving backwards.

```kotlin
fun drivePower(stick: Double, slowMode: Boolean): Double {
    // Replace this placeholder with your calculation and return value.
    return 0.0
}

fun main() {
    println(drivePower(0.8, false))
    println(drivePower(0.8, true))
    println(drivePower(-0.6, true))
    println(drivePower(0.0, true))
}
```

**Check your result:** The four lines should be `0.8`, `0.4`, `-0.3`, and `0.0`, in that order. Also try `-1.0` and `1.0` with both modes.

**Hint:** Use `return` to send a value back to the caller. `println()` inside the function is not a substitute for returning the result.

**Try more:** Add a third parameter, `slowFactor: Double`, so the caller can choose a factor such as `0.25` instead of always using half power.

## 7. Print a countdown

**Practice:** a `for` loop and a descending range.

Print a countdown from `5` to `1`, then print `GO!` once. Use a loop instead of five separate print statements. This exercise prints immediately; it does not need a timer or a delay.

```kotlin
fun main() {
    val start = 5

    // Print the countdown using a loop.
    // Print GO! after the loop.
}
```

**Check your result:**

```text
5
4
3
2
1
GO!
```

With `start = 3`, the output should begin at `3`. With `start = 0`, it should contain only `GO!`.

**Hint:** `start downTo 1` describes a descending range that includes `1` when `start` is at least `1`.

**Try more:** Rewrite it with a `while` loop. Decide which variable must change each time so the loop finishes.

## 8. Review practice scores

**Practice:** `List<Int>`, iteration, an accumulator, and decimal division.

The team recorded scores from four practice runs of a made-up challenge. Use a loop to calculate their total. Then print the number of runs, the total, and the average. Assume the list contains at least one score.

```kotlin
fun main() {
    val scores = listOf(12, 18, 15, 21)
    var total = 0

    // Add each score to total in a loop.
    // Calculate the average, then print the three results.
}
```

**Check your result:**

```text
Runs: 4
Total: 66
Average: 16.5
```

Try `listOf(10, 11)` as well: the total should be `21` and the average `10.5`.

**Hint:** Loop with `for (score in scores)`. Use `scores.size` for the number of runs and convert the total with `.toDouble()` before dividing to keep the fractional part.

**Try more:** Count how many runs scored at least `18` points. For the original list, the answer is `2`. Then handle `emptyList<Int>()` by printing `No practice runs` instead of calculating an average.

## 9. Handle a missing reading

**Practice:** nullable types and a `null` check.

A simulated distance sensor sometimes has no reading. If a reading exists, print its distance. Otherwise, print `Sensor unavailable`. A distance of `0` is a real reading and must not be treated as missing.

```kotlin
fun main() {
    val distanceCm: Int? = null

    // Check whether there is a reading before using it.
}
```

**Check your result:**

| `distanceCm` | Expected output |
| --- | --- |
| `null` | `Sensor unavailable` |
| `40` | `Distance: 40 cm` |
| `0` | `Distance: 0 cm` |

**Hint:** `Int?` means an integer or `null`. Start with `if (distanceCm != null)`. You do not need `!!` for this exercise.

**Try more:** Combine this with exercise 3: print `STOP` for a missing reading or a distance of `20` cm or less, and `DRIVE` for a larger distance.

## 10. Describe a robot

**Practice:** a `data class`, constructor parameters, and properties.

Create a data class named `Robot` with two properties: `name: String` and `wheelCount: Int`. Then create two instances and print a description of each. Put the class above `main()`.

```kotlin
// Define your Robot data class here.

fun main() {
    // Create a robot named Alpha with 4 wheels.
    // Create a robot named Beta with 6 wheels.
    // Print each robot's name and wheel count using its properties.
}
```

**Check your result:**

```text
Alpha has 4 wheels
Beta has 6 wheels
```

**Hint:** The declaration starts with `data class Robot(...)`. Put `val` before each property in the parentheses. Read an instance's properties with a dot, such as `robot.name`; inside a string, use `${robot.name}`.

**Try more:** Put both robots in a list and print their descriptions with one loop. Add a third robot and confirm it appears without adding another print statement.

## Put it together: a tiny driver simulator

Ready for a longer challenge? Reuse the ideas from exercises 3, 6, 8, and 9. Write a function `choosePower(distanceCm: Int?, stick: Double, slowMode: Boolean): Double` that returns:

- `0.0` when the distance is missing or is `20` cm or less;
- the result of your `drivePower` function otherwise.

Use `listOf(50, 25, 20, null, 40)` as a sequence of distance readings. Loop over the readings and print one chosen power for each, using a joystick value of `0.8` and slow mode enabled. The results should be `0.4`, `0.4`, `0.0`, `0.0`, `0.4`.

This is a console simulation. You are practicing how to separate a decision into a function and check it with sample inputs before learning to connect it to robot hardware.

## Before you move on

For each exercise, explain your code to a teammate, change an input, and predict the new output before running it. If the result surprises you, trace the values one line at a time. Keep your completed programs so you can reuse the ideas in the next exercise.

For a refresher, the official Kotlin Tour covers [basic types](https://kotlinlang.org/docs/kotlin-tour-basic-types.html), [control flow](https://kotlinlang.org/docs/kotlin-tour-control-flow.html), [functions](https://kotlinlang.org/docs/kotlin-tour-functions.html), [collections](https://kotlinlang.org/docs/kotlin-tour-collections.html), [null safety](https://kotlinlang.org/docs/kotlin-tour-null-safety.html), and [classes](https://kotlinlang.org/docs/kotlin-tour-classes.html).
