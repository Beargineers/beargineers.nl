---
title: Autonomous Patterns
published: false
summary: How Beargineers thinks about configurable autonomous routines, field state, sensors, and time budgets.
---

Autonomous code should express decisions clearly enough that the team can change strategy without rewriting the whole program.

## Concepts

- Program catalogues for choosing a routine during init.
- Field positions and no-fly zones as named data.
- Time budgets, cancellation, and safe final actions.
- Sensors as feedback, not decoration.
- Telemetry that explains what the robot believes is happening.

## From the DECODE season

Beargineers used configurable autonomous sequences so the drive team could adapt to alliance partners and field conditions. Some of the ideas from [Software & Control]({{ '/portfolio-archive/2025-26/decode/software/' | relative_url }}) should become reusable examples here.
