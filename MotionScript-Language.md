# MotionScript 2.0

MotionScript creates Roblox KeyframeSequences for R6 and R15. Source is data, not executable Lua. Preview and export use the same compiler. Existing R6 animations still default to R6 when `rig` is omitted.

## Complete structure

```text
animation Reach {
    rig R15
    framerate 60
    duration 2
    loop false
    priority Action

    defaults {
        easing Sine InOut
    }

    pose Ready {
        reset
        right_lower_arm rotation 15 0 0
    }

    frame 0f {
        use Ready
    }
    frame 30f {
        right_upper_arm rotation 70 -10 10
        right_lower_arm rotation 30 0 0
        marker ReachStart right
    }
    frame 60f {
        copy 0f
    }
}
```

Use one command per line and put closing braces on their own lines. Metadata and preset definitions can appear anywhere directly inside the animation; their position does not affect interpretation. Keywords are lowercase. Part names and easing values are case insensitive; preset names are case sensitive. Surrounding Markdown code fences are accepted. `--` starts a comment outside quoted marker text.

`framerate` is an integer from 1 to 240. `30f` means 30 source frames; `0.5s` and `0.5` mean half a second. `duration` accepts the same units. Omit duration to end at the final key. A longer declared duration adds a final hold; a shorter duration extends to the last key with a warning. One pose at zero plus a positive duration makes a static hold. Keys may be authored out of time order. Equal-time blocks merge in source order, with later commands taking precedence.

Priorities: Core, Idle, Movement, Action, Action2, Action3, Action4. The UI speed multiplier is baked into exported keyframe times, including markers. It does not change source frame numbers.

## Rigs and joint names

R6: `torso`, `head`, `right_arm`, `left_arm`, `right_leg`, `left_leg`.

R15:

```text
lower_torso
  upper_torso
    head
    right_upper_arm -> right_lower_arm -> right_hand
    left_upper_arm  -> left_lower_arm  -> left_hand
  right_upper_leg -> right_lower_leg -> right_foot
  left_upper_leg  -> left_lower_leg  -> left_foot
```

R15 aliases: `torso`/`body` = lower_torso, `waist` = upper_torso, `right_arm`/`left_arm` = corresponding upper arm, `right_leg`/`left_leg` = upper leg, `right_elbow`/`left_elbow` = lower arm, `right_wrist`/`left_wrist` = hand, `right_knee`/`left_knee` = lower leg, and `right_ankle`/`left_ankle` = foot. Canonical names are preferred. Compact spellings such as `RightUpperArm` are also accepted.

Each name controls the Motor6D connecting that part to its parent. Preview requires a matching selected rig in Workspace. Changing `rig R6` to `rig R15` selects a different skeleton; it does not automatically retarget an existing animation. Standard R15 joint bases are used for compilation without a selected reference rig. Preview and Insert KFS use the selected rig's actual C0 rotation to convert parent axes into joint axes; keep that same rig selected when exporting customized rigs. R6 retains its original transform convention.

## Transforms and state

```text
right_upper_arm rotation 80 -10 15
lower_torso position 0 -0.1 -0.05
right_upper_arm rotate 5 0 -2
lower_torso move 0 0.02 0
right_hand weight 0.7
right_hand reset
```

`rotation` sets XYZ Euler angles in degrees relative to the neutral joint. `position` sets a translation in studs. They replace those fields, not the entire pose. `rotate` adds to the stored Euler angles, and `move` adds to the stored position; they are not local matrix multiplications. Both relative commands are frame-only. `part reset` resets that part's position, rotation, and weight. `reset` resets and keys every part. Weight is 0..1, persists, and exports as Roblox Pose.Weight.

An unmentioned part is not keyed at that time. Each joint interpolates between its own keys, allowing an arm to move on a different rhythm from the head or wrist. For a newly keyed part, unmentioned transform fields retain their last authored values. A first key later than zero gets an implicit neutral key at zero. A last key earlier than duration gets a final hold. Never-keyed joints are not forced to neutral; necessary parent poses are structural with zero weight.

To hold a limb before a later movement, explicitly repeat its current pose at the end of the hold. An empty frame or a marker alone does not create a hold key for every joint. `copy` keys all joints, so it is appropriate for full-body returns and loop closure.

Axes are relative to the joint's parent part in its neutral pose, not world coordinates. With a standard upright avatar, +X is right, +Y is up, and -Z is forward. Rotation uses `CFrame.Angles(X,Y,Z)` order. These are not three interchangeable anatomical labels:

- Arms/legs hanging down: positive X raises them forward, negative X moves them backward. A forward arm raise is approximately `90 0 0`.
- The right arm swings outward with positive Z; the left arm swings outward with negative Z.
- Positive X bends an upright torso backward; negative X leans it forward. Positive head X looks up and negative head X looks down.
- A forward elbow bend commonly uses positive X; a knee bend toward the back uses negative X. Stay within plausible ranges and check the actual rig.
- Torso and upper-limb rotations carry their descendants. Account for inherited movement instead of doubling it at the wrist or head.

Interpolation uses CFrames and shortest rotation paths. For a full spin, use intermediate rotations spaced less than 180 degrees apart. Writing 0 then 360 alone will not create a visible revolution. Body translation does not move HumanoidRootPart through the world; runtime root motion needs separate game logic.

## Reusable poses and mirroring

```text
pose RightReach {
    right_upper_arm rotation 80 -8 10
    right_lower_arm rotation 25 0 0
    right_hand rotation -5 4 -6
}
frame 20f {
    use RightReach
    right_hand rotate 2 0 0
}
frame 50f {
    mirror RightReach
}
```

Presets can contain absolute rotation, position, per-part easing, weight, per-part reset, and reset. They cannot contain use/mirror, copy, relative commands, frames, or markers. Presets can be defined after their use and are validated even when unused.

`use Name` applies the named partial pose at the current frame. Commands run in source order, so following commands refine the result. `mirror Name` swaps left/right parts, negates position X, and negates rotation Y/Z across the avatar's sagittal plane. Central parts stay central with reflected values. A mirror is an authored symmetry operation, not an IK or contact solver.

`copy 0f` starts a frame from an earlier frame's complete resolved position/rotation/weight state. Copy is applied before that frame's other commands, regardless of where its line appears. It does not copy markers or easing. Missing, same-time, and future references are errors.

## Easing and markers

Easing precedence is per-part > frame > defaults. A key's easing controls its outgoing segment to the next key of that joint. Easing overrides belong to that key; they do not persist like position/rotation/weight.

Native Pose easings: Linear, Constant, Elastic, Cubic, Bounce, CubicV2. Additional sampled easings: Sine, Quad, Quart, Quint, Expo, Circular, Back. Directions: In, Out, InOut. Native curves stay native. Additional curves are sampled at the source FPS and exported as Linear poses, so preview and exported animation share the same curve. Their export is an approximation; use 60 or 120 FPS for sharp, short motion. Back can intentionally overshoot. Unknown styles produce an error rather than silently turning into Linear.

```text
defaults {
    easing Sine InOut
}
frame 18f {
    easing Quad Out
    right_hand easing Back Out
    marker Contact right_hand
    marker Note "left--right"
}
```

Markers become KeyframeMarkers in the exported sequence. Marker names are words using letters, digits, or underscores; values may contain text. Runtime consumers can listen for these markers. The plugin does not run gameplay actions from markers.

## Creating advanced animations with an AI

Start by identifying the rig, action, direction, duration, contact points, and whether it loops. Plan anticipation, weight transfer, acceleration, the main action, overshoot, follow-through, and recovery. Assign times before writing source.

Use as many meaningful poses as the action needs. A 2–3 second performance can use 15–30 or more source keys, with denser spacing around fast changes. Do not add random keys just to meet a number, and do not optimize for short code. Use partial per-joint keys so wrists, elbows, head, and torso can lead or lag one another. Favor asymmetry, readable silhouettes, believable balance, and deliberate arcs. Coordinate leg, knee, ankle, and pelvis movement to suggest contact; this version does not solve foot planting automatically.

For a loop, close the body pose with `copy 0f` and consider both pose and velocity across the seam. Matching endpoints alone does not guarantee matching velocity. A differing endpoint produces a warning. Keep marker placement intentional so events are not duplicated at both loop boundaries.

Before returning source, check every part name, angle sign, time, copy reference, duration, and easing. Explain any requested feature the language cannot express. There is no IK, automatic retargeting, arbitrary Lua execution, procedural noise, clip layering, or custom Bezier syntax in this version.

Study the complete examples in `examples/`: R15_WeightShift_Reach, R15_Mirrored_Boxing, and R6_WeightShift_Wave. They demonstrate timing and language features; their physical appearance has not been visually reviewed in Studio.

## Limits and editing

Limits keep generated input bounded: 1 MB of source, 4,096 source frame blocks, 600 seconds, 20,000 compiled keys, and 120,000 compiled poses including structural ancestors. Oversized sampled curves are rejected with a diagnostic. Position magnitude per component is limited to 10,000 studs; rotation to 36,000 degrees. These are parser limits, not anatomical recommendations.

Timeline dragging moves a source frame and its copy references together, including all blocks merged at that time. It rejects stale source, collisions, and broken copy dependencies before updating the editor. Recompile after manually editing source. Named presets stay in source; baked easing samples do not appear as editable source diamonds.

Version 2 requires complete animation blocks and valid commands. Old AI fragments with missing braces or unsupported easing used to be silently repaired; they now need correction. Valid existing R6 syntax remains supported.
