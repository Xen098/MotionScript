# MotionScript Language Documentation

MotionScript is a code-based animation language designed for creating Roblox animations with simple, readable instructions.

MotionScript currently supports **Roblox R6 rigs**.

Instead of manually working with `Motor6D`, `CFrame`, or Roblox's internal joint axes, MotionScript lets you describe animation using body parts, rotations, positions, frames, and easing.

---

# Animation Structure

Every MotionScript animation begins with an `animation` block.

```text
animation Walk {
    duration 1
    framerate 30
    loop true
    priority Movement
}
```

The animation name comes after `animation`.

```text
animation Walk {
}

animation Run {
}

animation SwordSlash {
}
```

Using one-word names such as `Walk`, `SmoothRun`, or `SwordSlash` is recommended.

---

# Supported Rig

MotionScript currently supports:

```text
R6
```

The supported body parts are:

```text
torso
head

right_arm
left_arm

right_leg
left_leg
```

Example:

```text
frame 0f {
    torso rotation 0 0 0
    head rotation 0 0 0

    right_arm rotation 20 0 0
    left_arm rotation -20 0 0

    right_leg rotation -20 0 0
    left_leg rotation 20 0 0
}
```

MotionScript automatically converts these simple body-part rotations into the correct R6 joint space.

You do not need to manually work with Roblox shoulder or hip `Motor6D` orientations.

---

# Duration

`duration` controls the total animation length in seconds.

```text
duration 1
```

means:

```text
1 second
```

Examples:

```text
duration 0.5
duration 0.8
duration 1
duration 1.5
duration 2
```

The final frame should normally match the duration.

At 30 FPS:

```text
duration 1
```

would normally finish at:

```text
frame 30f
```

---

# Frame Rate

`framerate` controls frame-based timing.

```text
framerate 30
```

The commonly supported values are:

```text
12
15
24
30
60
120
```

`30 FPS` is the recommended default.

Frame rate does not change Roblox's animation format. It determines how MotionScript converts frame numbers into seconds.

At 30 FPS:

```text
frame 0f   = 0 seconds
frame 15f  = 0.5 seconds
frame 30f  = 1 second
```

At 60 FPS:

```text
frame 30f = 0.5 seconds
frame 60f = 1 second
```

At 120 FPS:

```text
frame 30f  = 0.25 seconds
frame 60f  = 0.5 seconds
frame 120f = 1 second
```

---

# Frames

A frame describes a pose at a specific point in the animation.

Frame notation:

```text
frame 15f {
}
```

You may also use seconds directly:

```text
frame 0.5 {
}
```

Both are valid.

For AI-generated animations, frame notation is usually easier to manage:

```text
frame 0f {
}

frame 7f {
}

frame 15f {
}

frame 22f {
}

frame 30f {
}
```

---

# Looping

Use:

```text
loop true
```

for animations that repeat.

Examples include:

```text
Walk
Run
Idle
Breathing
Swimming
Crawling
```

Use:

```text
loop false
```

for animations that normally play once.

Examples include:

```text
Attack
Jump
Landing
Pickup
Wave
Reaction
Interaction
```

---

# Animation Priority

MotionScript supports Roblox animation priorities.

```text
priority Movement
```

Supported priorities:

```text
Core
Idle
Movement
Action
Action2
Action3
Action4
```

Recommended usage:

| Animation Type | Priority |
| --- | --- |
| Idle | `Idle` |
| Walk | `Movement` |
| Run | `Movement` |
| Crawl | `Movement` |
| Attack | `Action` |
| Interaction | `Action` |
| Emote | `Action` |

Example:

```text
animation Walk {
    duration 1
    framerate 30
    loop true
    priority Movement
}
```

---

# Default Easing

The standard MotionScript default is:

```text
defaults {
    easing Linear InOut
}
```

`Linear InOut` should normally be used unless another easing style is intentionally wanted.

Supported easing styles:

```text
Linear
Constant
Elastic
Cubic
Bounce
CubicV2
```

Supported directions:

```text
In
Out
InOut
```

Example:

```text
defaults {
    easing Linear InOut
}
```

Unsupported easing styles such as:

```text
Sine
Quad
Quart
Quint
Back
Circular
Exponential
```

are not MotionScript easing styles.

The MotionScript plugin may automatically replace unsupported styles with:

```text
Linear
```

For example:

```text
easing Sine Out
```

may be repaired to:

```text
easing Linear Out
```

---

# Rotation

Rotation syntax:

```text
body_part rotation X Y Z
```

Example:

```text
right_arm rotation 30 0 0
```

MotionScript uses easy anatomical rotation axes.

```text
X = forward / backward
Y = twist
Z = side-to-side
```

Examples:

```text
right_arm rotation 30 0 0
```

moves the right arm forward/backward.

```text
head rotation 0 20 0
```

turns the head.

```text
torso rotation 0 0 5
```

leans the torso sideways.

You may combine axes:

```text
right_arm rotation -35 4 6
```

---

# Position

Body parts can also receive position offsets.

Syntax:

```text
body_part position X Y Z
```

Example:

```text
torso position 0 -0.04 0
```

Position values are measured in Roblox studs.

Position and rotation can be used together:

```text
frame 8f {
    torso position 0 -0.04 0
    torso rotation 3 0 0
}
```

Small torso position changes are useful for adding weight to walks, runs, landings, and other movement.

---

# Stateful Frames

MotionScript is stateful.

A body part keeps its previous pose until you change it again.

Example:

```text
frame 0f {
    right_arm rotation -30 0 0
    left_arm rotation 30 0 0
}

frame 8f {
    right_arm rotation -10 0 0
}
```

At `8f`, the left arm is still:

```text
left_arm rotation 30 0 0
```

because it was never changed.

You do not need to rewrite every body part on every frame.

This also helps MotionScript create smoother animations because unchanged limbs are not unnecessarily re-keyed.

---

# Frame Easing

You may override easing for an entire frame.

```text
frame 10f {
    easing CubicV2 Out

    right_arm rotation -40 0 0
    left_arm rotation 40 0 0
}
```

Everything explicitly keyed in that frame uses the frame easing unless the body part has its own easing override.

---

# Body-Part Easing

You can override easing for one body part.

```text
frame 10f {
    right_arm rotation -40 0 0
    right_arm easing CubicV2 Out

    left_arm rotation 40 0 0
}
```

Only `right_arm` receives that pose-level easing.

---

# Copying Frames

`copy` copies the complete resolved pose of an earlier frame.

Example:

```text
frame 30f {
    copy 0f
}
```

This is especially useful for looping animations.

Example:

```text
frame 0f {
    right_arm rotation 25 0 0
    left_arm rotation -25 0 0
}

frame 15f {
    right_arm rotation -25 0 0
    left_arm rotation 25 0 0
}

frame 30f {
    copy 0f
}
```

The final pose becomes identical to the starting pose.

Do not copy a future frame.

Valid:

```text
frame 30f {
    copy 0f
}
```

Invalid:

```text
frame 0f {
    copy 30f
}
```

---

# Markers

MotionScript supports keyframe markers.

Syntax:

```text
marker Name Value
```

Example:

```text
frame 8f {
    marker Footstep Left

    left_leg rotation 5 0 0
}
```

Another example:

```text
frame 12f {
    marker Hit Sword
}
```

Markers can later be used for gameplay events such as:

```text
Footsteps
Attack hits
Particles
Sounds
Landing effects
Interactions
```

---

# Making Animations Smooth

Smoothness does not come only from easing.

The most important parts of a smooth animation are:

1. Good key poses.
2. Good timing.
3. Intermediate poses.
4. Natural movement between body parts.
5. Avoiding unnecessary repeated keys.

For example, this is a very basic walk:

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
```

It works, but additional intermediate poses can make it feel smoother.

```text
frame 0f
frame 7f
frame 15f
frame 22f
frame 30f
```

The intermediate poses describe how the character moves between the main extremes.

---

# Smooth Walk Example

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

        torso rotation 2 0 0
    }

    frame 7f {
        right_arm rotation 10 0 0
        left_arm rotation -10 0 0

        right_leg rotation -8 0 0
        left_leg rotation 8 0 0

        torso position 0 -0.025 0
    }

    frame 15f {
        right_arm rotation -25 0 0
        left_arm rotation 25 0 0

        right_leg rotation 20 0 0
        left_leg rotation -20 0 0

        torso rotation 2 0 0
    }

    frame 22f {
        right_arm rotation -10 0 0
        left_arm rotation 10 0 0

        right_leg rotation 8 0 0
        left_leg rotation -8 0 0

        torso position 0 -0.025 0
    }

    frame 30f {
        copy 0f
    }
}
```

---

# Walking and Running

For natural walking and running, arms and opposite legs normally move against each other.

Example:

```text
right_arm forward
left_leg forward

left_arm backward
right_leg backward
```

Then halfway through the cycle they reverse.

Avoid moving all limbs in the same direction at the same time unless that movement is intentional.

---

# Torso Movement

The torso affects much of the visible R6 character because the head and limbs are connected through it.

Large torso rotations can make the entire character appear tilted.

For normal walking or running, torso rotation should usually remain subtle.

Example:

```text
torso rotation 2 0 0
```

or:

```text
torso rotation -3 0 0
```

Values around roughly:

```text
-4 to +4 degrees
```

are usually enough for normal movement.

More extreme values can be used for stylized animations, attacks, dodges, leaning, or dramatic poses.

---

# Head Movement

Head motion should usually be subtle unless the animation specifically focuses on the head.

Example:

```text
head rotation -2 3 0
```

Small head movement can make an animation feel less robotic.

Avoid moving the head exactly opposite to the torso on every frame unless that is intentional.

---

# Avoid Repeated Identical Keys

Do not repeatedly write:

```text
frame 5f {
    right_arm rotation 20 0 0
}

frame 10f {
    right_arm rotation 20 0 0
}

frame 15f {
    right_arm rotation -20 0 0
}
```

unless you intentionally want the arm to hold still between `5f` and `10f`.

Repeated identical keys create a deliberate pause or hold.

For continuous movement, only key the limb when its motion should actually change.

---

# Intermediate Poses

Intermediate frames are especially useful when a movement changes direction.

For example:

```text
frame 0f {
    right_arm rotation 30 0 0
}

frame 7f {
    right_arm rotation 12 0 0
}

frame 15f {
    right_arm rotation -30 0 0
}
```

This gives the animation more control over how the arm passes through the center.

The same idea works for legs, torso movement, head movement, attacks, jumping, and other actions.

---

# One-Shot Animation Structure

For actions such as attacks, a useful structure is:

```text
Anticipation
↓
Action
↓
Recovery
```

Example:

```text
animation Punch {
    duration 0.6
    framerate 30
    loop false
    priority Action

    defaults {
        easing Linear InOut
    }

    frame 0f {
        right_arm rotation 0 0 0
    }

    frame 5f {
        right_arm rotation 35 0 0
        torso rotation -3 0 0
    }

    frame 10f {
        right_arm rotation -70 0 0
        torso rotation 4 0 0

        marker Hit Punch
    }

    frame 18f {
        right_arm rotation 0 0 0
        torso rotation 0 0 0
    }
}
```

---

# Loop Structure

A looping animation should normally end on the same pose where it started.

Example:

```text
animation Idle {
    duration 2
    framerate 30
    loop true
    priority Idle

    defaults {
        easing Linear InOut
    }

    frame 0f {
        torso position 0 0 0
    }

    frame 30f {
        torso position 0 -0.025 0
    }

    frame 60f {
        copy 0f
    }
}
```

---

# Full Syntax Reference

```text
animation AnimationName {
    duration NUMBER
    framerate NUMBER
    loop true/false
    priority PRIORITY

    defaults {
        easing STYLE DIRECTION
    }

    frame TIME {
        body_part rotation X Y Z
        body_part position X Y Z

        easing STYLE DIRECTION
        body_part easing STYLE DIRECTION

        marker NAME VALUE

        copy EARLIER_FRAME
    }
}
```

---

# Supported Values

## Rig

```text
R6
```

## Body Parts

```text
torso
head
right_arm
left_arm
right_leg
left_leg
```

## Easing Styles

```text
Linear
Constant
Elastic
Cubic
Bounce
CubicV2
```

## Easing Directions

```text
In
Out
InOut
```

## Priorities

```text
Core
Idle
Movement
Action
Action2
Action3
Action4
```

## Recommended Frame Rates

```text
12
15
24
30
60
120
```

---

# Recommended Default

When creating a new animation, this is a good starting template:

```text
animation NewAnimation {
    duration 1
    framerate 30
    loop false
    priority Movement

    defaults {
        easing Linear InOut
    }

    frame 0f {
        torso rotation 0 0 0

        head rotation 0 0 0

        right_arm rotation 0 0 0
        left_arm rotation 0 0 0

        right_leg rotation 0 0 0
        left_leg rotation 0 0 0
    }

    frame 30f {
        copy 0f
    }
}
```

From there, add meaningful intermediate frames to create the desired motion.
