# Spline Motion Warping

**Spline Motion Warping** extends Unreal Engine's built-in [Motion Warping](https://dev.epicgames.com/documentation/en-us/unreal-engine/motion-warping-in-unreal-engine) system with a new root motion modifier: **Spline Warp**. Instead of warping in a straight line to a target, the character smoothly follows any curve you draw with a Spline Component.

Use it for traversal abilities (rolls along curved paths, dashes around corners), cinematic movement (characters following scripted paths), or any gameplay where linear warping is too rigid.

---

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Setup Guide](#setup-guide)
  - [Step 1 -- Motion Warping Basics](#step-1----motion-warping-basics)
  - [Step 2 -- Place a Spline in the Level](#step-2----place-a-spline-in-the-level)
  - [Step 3 -- Register the Spline as a Warp Target](#step-3----register-the-spline-as-a-warp-target)
  - [Step 4 -- Select Spline Warp in the Montage](#step-4----select-spline-warp-in-the-montage)
  - [Step 5 -- Play the Montage](#step-5----play-the-montage)
- [Spline Warp Properties](#spline-warp-properties)
- [Rotation Modes](#rotation-modes)
- [Blend In](#blend-in)
- [Debug Visualization](#debug-visualization)
- [Blueprint Factory Function](#blueprint-factory-function)
- [Example Project](#example-project)
- [Video](#video)
- [Troubleshooting](#troubleshooting)
- [Support](#support)

---

## Requirements

| Requirement | Details |
|---|---|
| **Engine Version** | Unreal Engine 5.6 or higher |
| **Engine Plugin** | **MotionWarping** must be enabled (ships with UE) |
| **Root Motion** | Your animation must have root motion data |
| **Platforms** | Win64, Mac, Linux |

---

## Installation

1. Download the plugin zip for your engine version.
2. Extract the `SplineMotionWarping` folder into your project's `Plugins/` directory, so the path is: `YourProject/Plugins/SplineMotionWarping/SplineMotionWarping.uplugin`
3. Open your project. The plugin will compile automatically.
4. Verify it appears under **Edit > Plugins** (search for "Spline Motion Warping").

![Plugin enabled in the Plugins window](images/plugins-window.png)

---

## Setup Guide

### Step 1 -- Motion Warping Basics

Spline Motion Warping builds on top of Epic's Motion Warping system. If you are new to Motion Warping, start with Epic's official documentation:

> **[Motion Warping in Unreal Engine -- Epic Documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/motion-warping-in-unreal-engine)**

That guide covers:
- Enabling the Motion Warping plugin
- Adding the **Motion Warping Component** to your character
- Creating an **Animation Montage** with a **Motion Warping notify state**
- Using **Add or Update Warp Target** in Blueprints

Once you have a working Motion Warping setup (even a simple Skew Warp to a target point), you are ready to swap in a spline path. The following steps show what is **different** for Spline Warp.

### Step 2 -- Place a Spline in the Level

The spline defines the curved path the character will follow during the warp.

1. In the **Place Actors** panel, search for **Spline** (or create an Actor with a Spline Component).
2. Place the spline actor in your level.
3. Select the spline and adjust its control points in the viewport to shape the desired path.

![Spline placed in level with control points visible](images/spline-in-level.png)

> **Important:** The spline must **not** be a child of the character performing the warp. If the spline moves with the character, the path shifts during traversal and produces erratic results. Always use a standalone spline actor in the world.

### Step 3 -- Register the Spline as a Warp Target

This is the key difference from standard Motion Warping. Instead of passing a target **transform** (location/rotation), you pass the **Spline Component** itself:

1. In your character's Blueprint, get a reference to the **Motion Warping Component**.
2. Call **Add or Update Warp Target from Component**.
3. Set **Name** to a unique identifier (e.g., `SplinePath`). This must match the notify in Step 4.
4. Connect your **Spline Component** reference to the **Component** pin.

![Add or Update Warp Target from Component node with Spline Component connected](images/register-warp-target.png)

> Register the warp target **before** the montage reaches the warp notify window. A common pattern is to call this immediately before `Play Montage`.

### Step 4 -- Select Spline Warp in the Montage

1. Open your **Animation Montage**.
2. If you don't already have one, add a **Motion Warping** notify state on the Notifies track (right-click > Add Notify State > Motion Warping).
3. Position the notify window to cover the section of the animation where the character should follow the spline.
4. Select the notify state and look at the **Details** panel.
5. Set **Root Motion Modifier** to **Spline Warp** (instead of Skew Warp or Scale).
6. Set **Warp Target Name** to the same name used in Step 3 (e.g., `SplinePath`).

![Montage Details panel showing Spline Warp selected as Root Motion Modifier](images/montage-spline-warp.png)

You will also see the Spline Warp-specific properties described in the next section.

### Step 5 -- Play the Montage

Trigger the montage as you normally would (e.g., via `Play Montage` or through the Gameplay Ability System). When playback reaches the warp window, the character will follow the spline path instead of warping in a straight line.

![Character following a curved spline path during a roll animation](images/spline-warp-result.png)

---

## Spline Warp Properties

When you select Spline Warp as the Root Motion Modifier, the following properties appear in the notify state's Details panel:

| Property | Type | Default | Description |
|---|---|---|---|
| **Spline Rotation Mode** | Enum | `FaceSplineTangent` | How the character's rotation is driven during the warp. See [Rotation Modes](#rotation-modes). |
| **Z Axis Only** | Bool | `true` | Constrain rotation to yaw only. Prevents pitch and roll changes from spline curvature. |
| **Blend In Ratio** | Float (0--1) | `0.0` | Fraction of the warp window spent easing from raw root motion onto the spline. See [Blend In](#blend-in). |
| **Draw Debug** | Bool | `false` | Enable runtime debug visualization. See [Debug Visualization](#debug-visualization). |
| **Debug Draw Duration** | Float | `0.0` | How long debug lines persist (seconds). 0 = single frame. |

The standard Motion Warping properties are also available:

| Property | Type | Default | Description |
|---|---|---|---|
| **Warp Translation** | Bool | `true` | Warp the character's position along the spline. |
| **Warp Rotation** | Bool | `true` | Warp the character's facing direction. |
| **Ignore Z Axis** | Bool | `false` | Ignore the spline's vertical position; use the raw root motion height instead. |

---

## Rotation Modes

The **Spline Rotation Mode** controls how the character faces during the warp:

### FaceSplineTangent *(Default)*
The character rotates to face the direction of travel along the spline. As the spline curves, the character turns to follow.

**Best for:** Traversal abilities -- rolls, dashes, vaults -- where the character should look where they are going.

### FaceTarget
The character rotates to face the **end point** of the spline throughout the entire warp.

**Best for:** Strafing movements where the character keeps facing a fixed point while moving along a curved path.

### Locked
The character's rotation is frozen to whatever it was when the warp began.

**Best for:** Backward movements, side-steps, or any action where the initial facing must be preserved.

### UseWarpRotation
Falls back to the standard Motion Warping rotation logic. The rotation is computed from the animation's root motion delta toward the warp target.

**Best for:** When you want standard rotation behavior but with spline-based translation.

![Side-by-side comparison of the four rotation modes on the same spline path](images/rotation-modes.png)

---

## Blend In

The **Blend In Ratio** controls how the character transitions from raw root motion onto the spline at the start of the warp window:

| Value | Behavior |
|---|---|
| `0.0` | No blend. Character snaps to the spline position immediately. |
| `0.1` | First 10% of the warp duration eases from raw root motion onto the spline. |
| `0.5` | Half the warp duration is spent easing in. |

Uses smooth-step interpolation for a natural transition. A value between `0.05` and `0.15` is recommended for most use cases.

**Example:** If the warp window is 1.0 second and Blend In Ratio is `0.1`, the character spends the first 0.1s easing onto the spline, then follows it for the remaining 0.9s.

---

## Debug Visualization

Enable **Draw Debug** on the notify state to see what the spline warp is doing at runtime:

| Visual | Color | Meaning |
|---|---|---|
| Spline path (behind character) | Green | Portion of the spline already traversed |
| Spline path (ahead of character) | Cyan | Remaining spline path |
| Start marker | Green sphere | Spline start point |
| End marker | Red sphere | Spline end point |
| Desired position | Yellow sphere | Where the character is being pulled toward |
| Character-to-desired line | Orange | Gap between actual and desired position |
| Progress text | White | Current warp completion percentage |

![Debug visualization showing spline path, progress markers, and desired position](images/debug-visualization.png)

> Debug visuals are available in Development and Debug builds only (stripped from Shipping).

---

## Blueprint Factory Function

For advanced or procedural setups, you can add a Spline Warp modifier directly from Blueprints without a notify state:

**Node:** `Add Root Motion Modifier Spline Warp`

| Pin | Type | Description |
|---|---|---|
| Motion Warping Comp | `MotionWarpingComponent` | The character's Motion Warping Component |
| Animation | `AnimSequenceBase` | The animation being played |
| Start Time | Float | Warp window start (seconds into the animation) |
| End Time | Float | Warp window end (seconds into the animation) |
| Warp Target Name | Name | Must match the registered warp target name |
| Rotation Mode | Enum | One of the four rotation modes |
| Blend In Ratio | Float | 0.0 to 1.0 |

---

## Example Project

A standalone example project is available that demonstrates a complete working setup using the Third Person template:

> **[Download Example Project](https://github.com/StaffanJOlsson/SplineMotionWarping-Example)**

The example includes:
- A character with a Motion Warping Component
- A spline path placed in the level
- An animation montage configured with a Spline Warp notify
- Input binding to trigger the warp on key press
- Debug visualization enabled

---

## Video

See the plugin in action:

> **[Spline Motion Warping -- Demo Video]()**

The video covers setup, different rotation modes, blend-in behavior, and debug visualization.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| **Character teleports to spline start** | Set Blend In Ratio to a small value (e.g., `0.1`) to ease onto the spline. |
| **Character follows the spline erratically** | Make sure the spline is **not** a child of the character. Use a standalone spline actor. |
| **Warp doesn't activate** | Verify the Warp Target Name in the montage matches the name in Add or Update Warp Target. Ensure the target is registered before the notify window starts. Confirm the animation has root motion enabled. |
| **Character warps in a straight line** | The Warp Target's Component pin must be connected to the Spline Component. If you pass only a transform, standard linear warping is used. |
| **Character floats or clips through ground** | Enable Ignore Z Axis on the notify state to preserve raw root motion height. |
| **Rotation looks jittery** | Enable Z Axis Only to constrain rotation to the horizontal plane. |
| **Debug lines don't appear** | Draw Debug only works in Development/Debug builds. Also check the console variable `MotionWarping.Debug`. |

---

## Support

- [Documentation](https://github.com/StaffanJOlsson/SplineMotionWarping-docs)
- [Fab Store Page](https://www.fab.com/sellers/Staffan%20Olsson)
