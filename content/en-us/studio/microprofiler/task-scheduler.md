---
title: Task scheduler
description: Task Scheduler coordinates tasks done in each frame as the experience runs.
---

The **task scheduler** coordinates tasks done each frame as the game runs, even when the game is paused. These tasks include detecting player input, animating characters, updating the physics simulation, and resuming scripts in a `Library.task.wait()` state.

While there may be multiple tasks running, the task scheduler can potentially be overloaded, especially in the following situations:

- Using a custom character rig or input scheme.
- Animating parts yourself (instead of using an `Class.Animator`).
- Depending heavily on precise physics.
- Replicating objects regularly.

## Frames

A **frame** is a unit of game logic where work is done. Each frame should perform tasks efficiently, leading to more **frames per second** and a smoother player experience.

## RunService

The most direct way to add frame-by-frame game tasks is through the following members of `Class.RunService`:

- `Class.RunService:BindToRenderStep()`
- `Class.RunService.PreAnimation`
- `Class.RunService.PreRender`
- `Class.RunService.PreSimulation`
- `Class.RunService.PostSimulation`
- `Class.RunService.Heartbeat`

## Scheduler priority

The task scheduler categorizes and completes tasks in the following order. Some tasks may not perform work in a frame, while others may run multiple times.

<img src="../../assets/optimization/task-scheduler/scheduler-priority.png" />

## Best practices

To build performant games with efficiency in mind, note the following:

- **Don't connect/bind functions to the render step unless absolutely necessary.**
  Only tasks that must be done after input but before rendering should be done in such a way, like camera movement. For strict control over order, use `Class.RunService:BindToRenderStep()|BindToRenderStep()` instead of `Class.RunService.PreRender|PreRender`.

- **Manage physical states carefully.**
  `Class.RunService.PreSimulation|PreSimulation` happens **before** physics, while `Class.RunService.PostSimulation|PostSimulation` happens **after** physics. Therefore, gameplay logic that affects the physics state should be done in `Class.RunService.PreSimulation|PreSimulation`, such as setting the `Class.BasePart.Velocity|Velocity` of parts. In contrast, gameplay logic that relies on or reacts to the physics state should be handled in `Class.RunService.PostSimulation|PostSimulation`, such as reading the `Class.BasePart.Position|Position` of parts to detect when they enter defined zones.

- **Motor6D transform changes should be done on the PreSimulation event.**
  If you don't, `Class.Animator|Animators` will overwrite changes on the next frame. Even without an `Class.Animator`, `Class.RunService.PreSimulation|PreSimulation` is the last Luau event fired before `Class.Motor6D.Transform` is applied to part positions.
