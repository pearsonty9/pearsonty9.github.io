---
layout: project
project: Corruption
permalink: /projects/corruption/
title: "Making Corruption: building the wall that chases you"
description: The part I built for Corruption, a 2D speedrunning platformer made for VimJam 5, where a wall of corruption sweeps across the level behind you.
tags: [Godot, Game Jam, GDScript, Shaders, 2D]
image: /assets/img/games/corruption.png
---

[Corruption](https://vellichorgames.itch.io/corruption) is a challenging 2D pixel-art
platformer I made with Zaher and Steven for VimJam 5. It's designed around
speedrunning: a wall of corruption moves across the level behind you, and if it
catches you, you die.

![Corruption cover art](/assets/img/games/corruption.png)

## The jam

VimJam 5 ran for ten days in November 2024. Instead of a required theme, it had a
required focus, *Racing*, which could mean any kind of race against something or
against the clock. There was also an optional theme, *Escape*. Our answer was a
platformer where you're always racing the corruption to the end of the level.

We didn't get the game submitted before the deadline, so it isn't in the jam
results, but we still put it up on itch.io.

## What I worked on

Steven and Zaher built the player movement, enemies, and platforms, and Zaher built
most of the level. I built the
corruption itself: the shader that draws it, the barrier that kills the player,
and the code that moves it through the level. I also hooked it up to the player's
death, the death screen, and the checkpoints.

## Drawing the corruption

The corruption is a `Sprite2D` with a `canvas_item` shader. A `pos` uniform goes
from 0 to 1 and marks how far the corruption has spread across the sprite.
Everything to the left of `pos` is black. To give it a pixelated, ragged edge, the
shader snaps the UVs to a grid of square blocks and gives each block a random
threshold, so blocks just ahead of the edge turn black at slightly different times:

```glsl
float random(vec2 uv) {
    return fract(sin(dot(uv.xy, vec2(12.9898, 78.233))) * 43758.5453123);
}

void fragment() {
    float aspect_ratio = TEXTURE_PIXEL_SIZE.y / TEXTURE_PIXEL_SIZE.x;
    vec2 grid_uv = round(UV * float(amount)) / float(amount);
    grid_uv.x = round(UV.x * float(amount) * aspect_ratio) / float(amount) / aspect_ratio;

    if (grid_uv.x < pos)
        COLOR = vec4(0, 0, 0, 1);
    else if (grid_uv.x - pos < random(grid_uv) / 10.0)
        COLOR = vec4(0, 0, 0, 1);
    else
        COLOR = vec4(1, 1, 1, 0);
}
```

The aspect ratio correction keeps the blocks square even though the sprite is much
wider than it is tall.

In my first version, the uncorrupted part was drawn white, so the corruption was a
background behind the level. Later I changed it to be transparent, which turned it
into an overlay that eats the level as it moves.

## Moving it and killing the player

The shader only draws the corruption, so I added an `Area2D` barrier as a child of
the sprite. Every frame, the script advances `progress`, passes it to the shader,
and moves the barrier to match the visible edge. The player dies when it touches
the barrier. Simplified:

```gdscript
func _process(delta: float) -> void:
    background.material.set_shader_parameter("pos", progress)
    collision.position.x = progress * background.texture.get_width() + collision_offset
    progress += delta * speed / 100
```

When the player dies, the corruption speeds up and sweeps across the screen. Once
the barrier has passed the player and gone off screen, the death screen fades to
black. The death screen has a button that reloads the scene to try again.

The level is much wider than one screen, so the corruption also has to work with
checkpoints. When the player reaches a checkpoint, the script resets `progress` and moves the corruption to just off the
left edge of the screen:

```gdscript
func _on_checkpoint_body_entered(body: Node2D) -> void:
    if body.name == "Player":
        progress = 0
        if !visibility.is_on_screen():
            var screen_width = player.get_node("Camera2D").get_viewport_rect().size.x
            position.x = player.position.x - screen_width - collision_offset
```

The speeds after a death and after a checkpoint were tuned by the team while
building the level.

## What I'd do differently

The corruption works by tying a shader value and a collider to the same number,
and then teleporting the whole thing around the level. Once there were checkpoints
and a death sequence, each one needed its own rules for where the corruption should
be and how fast it should move. Next time I'd design
how it behaves across a whole level at the start, instead of adding those cases one
at a time.

I'd also aim to have a complete, submittable build well before the deadline.
Missing the submission meant the game was never played or rated as part of the jam.

**[Play Corruption on itch.io →](https://vellichorgames.itch.io/corruption)**
