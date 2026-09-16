# MotionScript

MotionScript is a code-first animation tool for Roblox Studio.

It lets you create Roblox animations by writing simple animation code instead of manually placing every keyframe in the Animation Editor.

## Why use MotionScript?

MotionScript is useful for:

- creating animations faster
- generating animations with AI
- editing animation timing directly in code
- making repeatable animation setups
- previewing animations directly on an R6 rig
- controlling easing, FPS, timing, positions, and rotations
- exporting animations as Roblox `KeyframeSequence` objects

## Example

```text
animation Walk {
    duration 1
    framerate 30
    loop true
    priority Movement

    defaults {
        easing Linear InOut
    }

    frame 0f {
        right_arm rotation 25 0 0
        left_arm rotation -25 0 0
        right_leg rotation -20 0 0
        left_leg rotation 20 0 0
    }

    frame 15f {
        right_arm rotation -25 0 0
        left_arm rotation 25 0 0
        right_leg rotation 20 0 0
        left_leg rotation -20 0 0
    }

    frame 30f {
        copy 0f
    }
}
