---
layout: project
project: Kart Wheel
permalink: /projects/kart-wheel/
title: "Making Kart Wheel: prototyping and lighting a jam game"
description: The parts I built for Kart Wheel, a 3D tower defense game made for Pirate Software Game Jam 2025, from the first prototype to the final lighting.
tags: [Godot, Game Jam, GDScript, 3D, Lighting]
image: /assets/img/games/kart-wheel.png
---

[Kart Wheel](https://caradecachapa.itch.io/kart-wheel) is a 3D tower defense game I
made with a team of five for Pirate Software Game Jam 2025. The theme was *You Are
The Weapon*: you play as a kart wheel defending a tower from waves of undead
monkeys, and you defeat them by running them over.

![Kart Wheel cover art](/assets/img/games/kart-wheel.png)

## What I worked on

On the first day I set up the project and built the first playable prototype. On the
last two days I modeled the fence that bounds the arena and did the lighting and
environment.

## The first prototype

I set up a player scene, an enemy scene, a tower scene, and a level, and connected
them so the team had something playable to build on from the first day. The player
movement was adapted from Godot's *Your first 3D game* tutorial, and the enemies
moved straight toward the tower.

One of the first things I fixed was enemies walking through the tower. They were
`RigidBody3D` nodes, and I was moving them by setting their position every frame:

```gdscript
# Before: sets the position directly, so no collisions are detected
var next_position = position.move_toward(target.position, speed)
position.x = next_position.x
position.z = next_position.z
```

Setting `position` directly bypasses the physics engine, so the enemy moves to its
new position without checking for anything in the way. The fix was to change the
enemies to `CharacterBody3D` nodes and move them with a velocity and
`move_and_slide()`, which handles collisions:

```gdscript
# After: move_and_slide() stops the enemy at the tower
var direction = target.position - position
velocity.x = direction.x * speed
velocity.z = direction.z * speed
move_and_slide()
```

I also added friction to the player so the wheel would slow down when you stopped
accelerating:

```gdscript
velocity = velocity.lerp(Vector3.ZERO, friction * delta)
```

## Level boundaries

To keep the player inside the arena, I modeled a wooden fence and placed sections
of it around the edge. Each section has a `StaticBody3D` with a box
collider in a `wall` group, so the player code can detect when the wheel hits a
wall.

## Lighting and environment

For the final version, I turned the level into a night scene:

- A dark procedural sky with noise-based cloud cover
- Light distance fog
- A dim directional light with shadows enabled
- Glow, for the particle effects
- A brown dirt ground material

I also experimented with a wind-swaying grass shader adapted from an open-source
Godot project, which animates vertices using a scrolling noise texture. It's built
for individual grass blades, so for the final build I went with a simple ground
material that fit the look of the scene.

## What I'd do differently

Next time I'd set up the sky, fog, and lighting in the first few days with
placeholder models, so the visuals could develop alongside the gameplay. That would
also leave room to bring back ideas like the grass shader with proper grass meshes.

Having a playable prototype on the first day worked well. It let the whole team
start working in parallel, and the game grew from that foundation.

**[Play Kart Wheel on itch.io →](https://caradecachapa.itch.io/kart-wheel)**
