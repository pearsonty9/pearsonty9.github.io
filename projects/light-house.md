---
layout: project
project: Light House
permalink: /projects/light-house/
title: "Making Light House: building and lighting a dark prison"
description: The parts I built for Light House, a 3D survival horror game made for GMTK Game Jam 2024, from the prison rooms to the fog and lighting.
tags: [Godot, Game Jam, 3D, Lighting, Level Design]
image: /assets/img/games/light-house.png
---

[Light House](https://vellichorgames.itch.io/light-house) is a 3D survival horror
game I made with a team of five for GMTK Game Jam 2024. You make your way through a
dark, abandoned prison with a flashlight, trying to reach the lighthouse at the end
while something hunts you.

![Light House cover art](/assets/img/games/light-house.png)

## Built to Scale

The theme for GMTK Game Jam 2024 was *Built to Scale*. Our answer was the
flashlight. Hold one button and its beam turns red and grows whatever you point it
at. Hold the other and it turns blue and shrinks things. You use it to shrink
crates that block your way and to grow cracked walls until you can get through. Steven
built the player, the flashlight, and the growing and shrinking system.

## What I worked on

I set up the Godot project on the first day, and after that most of my work was the
level itself. I built the prison cell block where the game starts and the locker
room that serves as the tutorial. I also set up the fog and lighting, and
textured the hallways that connect the rooms. Most of the models were free assets
from Sketchfab and OpenGameArt, and the textures came from Poly Haven. My job was
to bring those into Godot, turn each one into a reusable scene, and arrange them
into rooms that felt like a place.

## Building the rooms

Each prop got its own small scene under `tyler/assets/`: cells, hanging lights,
lockers, benches, desks, tables, crates, buckets, rubble, and scattered papers.
Wrapping each model in a scene let me fix its scale and orientation once and then
drop copies of it anywhere in the level.

The starting room is a cell block. A row of eight cells lines the walls, with
tables, chairs, food trays, and a pile of bricks in the middle. The tutorial room
is a locker room full of green lockers, some of them damaged, with benches and
desks. That's also where you pick up the flashlight.

The walls and floors are simple planes and boxes with a material on them. A 4K
texture stretched across a 27-meter wall looks blurry and oversized, so each
material sets `uv1_scale` to repeat the texture to fit the surface. The long
hallways repeat their textures many more times along their length than across
their height:

```
[sub_resource type="StandardMaterial3D" id="StandardMaterial3D_7ne5j"]
albedo_texture = ExtResource("11_e7inq")    # brick_wall_08 diffuse
roughness_texture = ExtResource("13_pc8b1")
normal_enabled = true
normal_texture = ExtResource("12_2lmni")
uv1_scale = Vector3(14, 2, 1)
```

## Darkness and fog

For a game about finding the light, most of the level had to be dark. I turned
the ambient light almost all the way down, so anything the lights don't reach goes
close to black, and added a dense, warm fog so the far ends of rooms fade out:

```
[sub_resource type="Environment" id="Environment_25oov"]
ambient_light_sky_contribution = 0.0
ambient_light_energy = 0.02
tonemap_mode = 2
glow_enabled = true
fog_enabled = true
fog_light_color = Color(0.546875, 0.458146, 0.354852, 1)
fog_light_energy = 0.02
fog_density = 0.2
```

The light sources are the hanging lamp models. The model is just a mesh, so I
gave some of the lamps a `SpotLight3D` pointing down, with soft shadows and a wide
cone. That makes small pools of light on the floor, with darkness in between:

```
[node name="SpotLight3D" type="SpotLight3D" parent="Starting Room/Hanging_Light3"]
light_color = Color(0.988281, 1, 0.976563, 1)
light_energy = 10.0
shadow_enabled = true
shadow_opacity = 0.75
shadow_blur = 1.5
spot_angle = 80.0
```

## What I'd do differently

Most of the level lived in one scene file, `level.tscn`, and several of us edited
it at the same time. On the last day, a lot of my commits went to fixing that file
after merges, including repairing it when it became corrupted. Next time I'd split each room into
its own scene and build the level out of those, so people could work on different
rooms without colliding.

I'd also be more careful with asset sizes. I imported many of the props with 4K
textures, which made the project heavy for objects the player mostly sees in the
dark.

**[Play Light House on itch.io →](https://vellichorgames.itch.io/light-house)**
