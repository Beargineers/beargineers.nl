---
title: Why coroutines
summary: Robot code is about time. How BearPlatform's coroutines compare to state machines and composable actions.
---

## Robot code is about time

An autonomous is a story told over time: drive to the spike, collecting on the way, then drive to the basket and
shoot. TeleOp is full of the same: hold the button to spin up, release to fire, then retract.

The FTC SDK runs robot code in a loop, many times a second. Each pass reads the sensors and gamepads, decides what to
do, and must end quickly, or the robot stops reacting: a `sleep(2000)` in the middle freezes everything else for two
seconds, gamepads included. So "drive there, *then* open the claw" can't be written as the two lines it is.

FTC programmers have found two ways around it. BearPlatform takes a third.

## State machines

The classic answer, taught in [Game Manual 0](https://gm0.org/en/latest/docs/software/concepts/finite-state-machines.html):
an enum says which step the robot is at, and every loop does a bit of the current step and decides whether to move on.

Say the robot should drive to a spike with its intake on, and give up on it after 3 seconds; if it collected something,
drive to the basket and shoot; if not, try the next spike:

```kotlin
enum class Step { COLLECT, TO_BASKET, SHOOT, DONE }

var step = Step.COLLECT
var spike = 0
val stepTimer = ElapsedTime()

override fun start() = stepTimer.reset()

override fun loop() {
    when (step) {
        Step.COLLECT -> {
            intake.on()
            follower.followTowards(spikes[spike])
            if (follower.isAt(spikes[spike]) || intake.isFull || stepTimer.seconds() > 3) {
                intake.off()
                when {
                    intake.isFull -> step = Step.TO_BASKET
                    ++spike < spikes.size -> stepTimer.reset()
                    else -> step = Step.DONE
                }
            }
        }
        Step.TO_BASKET -> {
            follower.followTowards(BASKET)
            if (follower.isAt(BASKET)) {
                shooter.startShot()
                step = Step.SHOOT
            }
        }
        Step.SHOOT -> if (shooter.isDone) step = Step.DONE
        Step.DONE -> follower.stop()
    }
}
```

It works, and there is no magic in it. But the story is cut into pieces: the order of the steps lives in the
transitions, the loop over spikes is a counter, the timeout a timer to reset at the right moment, and every new step
or decision means more states. Two things at once, like an arm moving while the robot drives, need a second machine
and some flags between them, and stopping the routine halfway, when the driver takes over, means putting every
machine back in order by hand.

## Composable actions

The modern answer, from Road Runner's [Actions](https://rr.brott.dev/docs/v1-0/actions/), FTCLib's
[command-based programming](https://docs.ftclib.org/ftclib/command-base/command-system) and
[Defined](https://defined.teamundefined.com/): each step is an action object that advances a bit every loop and knows
when it is done, and actions combine into sequences and parallel groups that a scheduler runs. Libraries add what
matters on a robot: actions claim the subsystems they use, so a new request cancels the old one (Defined's "newest
request wins"), they clean up when cancelled, and some keep running all match, like Defined's monitors.

The same routine, sketched with actions (the names differ between libraries):

```kotlin
fun trySpikes(index: Int): Action =
    if (index == spikes.size) Instant {}
    else Sequential(
        Instant { intake.on() },
        Race(DriveTo(spikes[index]), WaitUntil { intake.isFull }, Wait(3.0)),
        Instant { intake.off() },
        If({ intake.isFull },
            then = Sequential(DriveTo(BASKET), Shoot()),
            otherwise = trySpikes(index + 1)),
    )
```

That reads much more like the story, and actions compose: `DriveTo` and `Shoot` are reused everywhere. The price is
that the program is now a structure of objects rather than code. Each step is a class with its start, update, end and
cancel; the `if` became an `If` action, the loop over spikes a recursion that builds actions; and anything an action
computes, the next one has to find through a shared variable. When something goes wrong, the stack trace points into
the scheduler, not to the line of the routine that misbehaved.

## Coroutines

Kotlin can do something Java can't: a `suspend` function can pause in the middle, at a `nextTick()`, a `waitUntil` or
a `driveTo`, and continue later, right where it stopped, with all its local variables. BearPlatform's opmodes resume
paused functions once every loop. So the routine is written as the story it is:

```kotlin
suspend fun MyRobot.collectAndScore(spikes: List<Pose>) {
    for (spike in spikes) {
        intake.on()
        // Drive to the spike, but stop as soon as the intake is full, and give up after 3 seconds
        withTimeoutOrNull(3.seconds) {
            race({ follower.driveTo(spike) }, { waitUntil { intake.isFull } })
        }
        intake.off()
        if (intake.isFull) {
            follower.driveTo(BASKET)
            shooter.shoot()
            return
        }
    }
}
```

The loop is a `for`, the decision an `if`, the result a variable, and stopping early a `return`. `driveTo` and `shoot`
are themselves just suspend functions, made the same way. Nothing blocks: while this function waits for the robot to
reach the spike, the loop goes on, reading sensors, running TeleOp handlers or other routines. And it all still runs
on one thread, so there are no threads, locks or races to worry about.

It is not magic, either: the Kotlin compiler turns every suspend function into a state machine, the one you would
otherwise write by hand, with the current step and the local variables kept for when it resumes. You get to write the
story; the compiler writes the machine.

### What actions give, coroutines give too

| | State machine | Composable actions | Coroutines in BearPlatform |
|---|---|---|---|
| Steps in order | states and transitions | `Sequential(...)` | lines, one after another |
| Waiting | a check in a state | `Wait`, `WaitUntil` actions | `delay`, `waitUntil` |
| Things at once | more machines, flags | `Parallel(...)` | `launch` |
| First one wins | combined conditions | `Race(...)` | `race` |
| A time limit | a timer, reset by hand | a timeout on the action | `withTimeout` |
| Decisions and loops | more states, counters | `If` actions, recursion | `if`, `for`, `while` |
| Cleanup when stopped | in every way out | a cancel hook | `try`/`finally` |
| One part, one action | flags by hand | requirements, slots | `takeOver` |
| Every-loop behavior | in `loop()` | monitors, default commands | `everyTick` |
| A new behavior | new states | a new action class | a new suspend function |

So the good ideas of action libraries are all here: a part taken over by a new action stops the old one, which cleans
up in its `finally`; things run side by side without threads; `everyTick` plays the role of a monitor. See
[Robot code over time]({{ '/platform/robot-code-over-time/' | relative_url }}) for all of it.

## Things to know

- **It's Kotlin.** Suspend functions are a Kotlin feature, so robot code on BearPlatform is Kotlin. Java opmodes can
  live in the same project, but can't be written this way.
- **Never block.** `Thread.sleep()`, or a loop that doesn't call `nextTick()`, holds up the whole robot, as in any
  loop-based code. Wait with `delay`, `waitUntil` or `nextTick()`.
- **There's little to learn.** Students don't need to know coroutines in depth: `suspend`, the platform's handful of
  functions for waiting, `race` and `launch`, and `try`/`finally` cover robot code. Kotlin's own
  [coroutines guide](https://kotlinlang.org/docs/coroutines-overview.html) goes further, for the curious; most of it,
  like threads and dispatchers, BearPlatform handles for you.
