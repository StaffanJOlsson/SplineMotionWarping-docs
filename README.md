# Spline Motion Warping

A code plugin for Unreal Engine that adds spline-based root motion warping. Characters follow a `USplineComponent` path during a motion warp window, using the same Motion Warping notify you already know.

## Requirements

- Unreal Engine 5.6+
- **Motion Warping** plugin (built-in, must be enabled)

## Installation

1. Copy the `SplineMotionWarping` folder into your project's `Plugins/` directory.
2. Enable the **Motion Warping** plugin in your project if it isn't already.
3. Rebuild your project.

## Quick Start

### 1. Montage Setup

Add an **Anim Notify State — Motion Warping** to your montage (the same notify you use for regular warps). In its details panel:

- Set **Root Motion Modifier** to **Spline Warp**
- Set **Warp Target Name** to match your target name (e.g. `Roll`)

![Montage setup](images/montage-setup.png)

### 2. Blueprint Setup

Wire a spline component into a warp target using the standard **Add or Update Warp Target** node:

1. Get a reference to your **Spline Actor** and its **Spline Component**
2. Create a **Make Motion Warping Target** node
3. Plug the Spline Component into the **Component** pin
4. Set the **Name** to match your montage's Warp Target Name
5. Pass the target to **Add or Update Warp Target** on the Motion Warping Component

![Blueprint setup](images/blueprint-setup.png)

That's it. Play the montage and the character follows the spline.

## Configuration

All settings are exposed on the Motion Warping notify in the montage editor.

### Spline Rotation Mode

| Mode | Description |
|------|-------------|
| **Face Spline Tangent** | Character faces the movement direction along the spline (default) |
| **Face Target** | Character faces the spline endpoint throughout traversal |
| **Locked** | Rotation locks to the character's orientation when the warp began |
| **Use Warp Rotation** | Falls back to the inherited warp rotation logic |

All modes support a **Z-Axis Only** toggle that constrains rotation to yaw (no pitch/roll).

### Blend In Ratio

Controls how much of the warp duration is spent easing from raw root motion onto the spline path. Prevents teleporting to the spline start when the character isn't already standing on it.

- `0.0` — Snap to spline immediately (default)
- `0.1` — First 10% of the warp duration smoothly blends onto the spline

Uses smooth-step interpolation for natural acceleration.

### Warp Rotation Time Multiplier

Controls how quickly rotation reaches its target. Works with Face Spline Tangent, Face Target, and Use Warp Rotation modes.

- `1.0` — Rotation spreads evenly across the remaining window (default)
- `0.5` — Rotation completes in half the time, snapping earlier

### Inherited Properties

The modifier inherits from `URootMotionModifier_Warp`. These inherited properties are actively used:

- **Warp Translation** — Enable/disable position warping along the spline
- **Ignore Z Axis** — Prevents vertical warping (preserves raw root motion Z)
- **Warp Rotation** — Enable/disable rotation warping
- **Warp Rotation Time Multiplier** — See above

Other inherited properties (Rotation Type, Rotation Method, Warp Point Anim Provider, etc.) are unused during spline traversal and can be ignored.

## Debug Visualization

Enable **Draw Debug** on the modifier to see:

- Spline path colored by progress (green = traversed, cyan = ahead)
- Start/end markers (green/red spheres)
- Current desired position on the spline (yellow sphere)
- Line from character to desired position (orange)
- Coordinate system at the spline endpoint
- Progress percentage text above the character

## Blueprint Factory (C++)

For programmatic setup without montage notifies:

```cpp
URootMotionModifier_SplineWarp::AddRootMotionModifierSplineWarp(
    MotionWarpingComp,
    Animation,
    StartTime,
    EndTime,
    WarpTargetName,
    ESplineWarpRotationMode::FaceSplineTangent, // optional
    0.f);                                        // BlendInRatio, optional
```

## Support

[Fab Store Page](https://www.fab.com/sellers/Staffan%20Olsson)
