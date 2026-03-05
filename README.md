# Spline Motion Warping: Quick Setup Guide

Spline Motion Warping is a UE 5.6+ plugin that allows characters to follow a `USplineComponent` path during a motion warp window. This guide covers the installation and setup required to get your characters moving along custom paths.

## Requirements & Installation

- **Engine Version:** Unreal Engine 5.6 or higher.
- **Dependencies:** The built-in **MotionWarping** engine plugin must be enabled.
- **Installation:** Copy the `SplineMotionWarping` folder into your project's `Plugins/` directory and rebuild.

## Quick Setup

Follow these steps to implement spline-based movement:

### 1. Prepare the Character and Spline

- Add a `UMotionWarpingComponent` to your character blueprint.
- Place a `USplineComponent` in your level.

> **Pro-Tip:** Ensure the Spline Component is not a child of the actor performing the motion warp. If the spline is attached to the character, the path will move along with the warping, leading to unpredictable results. It is best to use a detached spline or a standalone Spline Actor.

### 2. Register the Warp Target

Use the **Add or Update Warp Target** node in Blueprints:

- **Name:** Set a unique name (e.g., `Roll`).
- **Component:** Plug your Spline Component reference directly into the Component parameter.

### 3. Configure the Animation Montage

In your Animation Montage, add an **Anim Notify State — Motion Warping**:

- **Warp Target Name:** Match the name used in Step 2.
- **Root Motion Modifier:** Select **Spline Warp** from the dropdown menu.
- **Rotation Mode:** Choose how the character faces, such as `FaceSplineTangent`.

## Key Features

| Feature | Description |
|---------|-------------|
| **Rotation Modes** | Supports `FaceSplineTangent`, `FaceTarget`, `Locked`, and `UseWarpRotation`. |
| **Blend In Ratio** | (0.0 -- 1.0) Controls the time spent easing from raw root motion onto the spline path. |
| **Debug Visuals** | Enable `bDrawDebug` to see the path progress, start/end markers, and current desired position. |
| **Z-Axis Only** | A toggle to constrain all warping rotations strictly to the Yaw axis. |

## Support

[Fab Store Page](https://www.fab.com/sellers/Staffan%20Olsson)
