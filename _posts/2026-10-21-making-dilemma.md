---
title: "Making DiLemma: tuning a combo system in two weeks"
description: How we built a math-themed dungeon brawler for Secret Santa Jam 2024, and what I learned making a three-button combo system feel good.
tags: [Godot, Game Jam, GDScript, Shaders]
image: /assets/img/games/dilemma.png
---

[DiLemma](https://tigershillo.itch.io/dilemma) is a small action platformer I made
with two friends, Zaher Ilbih and Steven K. Jr., for Secret Santa Jam 2024. It was a
classic jam experience: two weeks with three of us in a small apartment, writing code and building
something together until the deadline.

![DiLemma cover art](/assets/img/games/dilemma.png)

## A game as a gift

Secret Santa Jam has a twist. Instead of a theme, every team is assigned someone
else's "letter to Santa" and makes a game for that person. Ours came from someone
who, like me, loves math, physics, and classic video games.

So we made a dungeon covered in mysterious symbols that look a lot like
common math notation. The name is a pun on two ideas. A *lemma* is a small proven
result you use as a stepping stone in a larger proof. A *dilemma* is a choice
between two options, and early in the game you have to pick between two dungeons.

## What I worked on

I drew all of the stick figure animations, wrote the game's shaders, and helped
build the combo system the player uses to chain attacks. That combo system is the
part I want to dig into, because it was the hardest thing in the game to get right.

## Three buttons, four finishers

The player has three attacks: light (`J`), heavy (`K`), and magic (`L`). Every
combo opens with a light attack, follows with heavy or magic, and ends with a
finisher chosen by the last two hits:

| Input | Finisher |
| --- | --- |
| `J` `K` `K` | Spin Attack |
| `J` `K` `L` | Magic Burst |
| `J` `L` `K` | Flaming Sword |
| `J` `L` `L` | Empowered Fireball |

Under the hood, the player is a small state machine. Each attack sets a `state`,
plays the matching animation, and gets appended to a `current_combo` list. When the
next attack comes in, the controller looks at the end of that list to decide what
to do. Here's a simplified version of the heavy attack handler:

```gdscript
if Input.is_action_just_pressed("heavy_attack") and state == "idle":
    if combo_timer.time_left > 0 and current_combo.size() == 2:
        # Third hit: pick the finisher from the second hit
        if current_combo.back() == "heavy_attack":
            state = "heavy_heavy_attack"   # J K K
        elif current_combo.back() == "magic_attack":
            state = "magic_heavy_attack"   # J L K
        current_combo.clear()
    else:
        state = "heavy_attack"
        if combo_timer.time_left > 0 and current_combo.back() == "start_combo":
            current_combo.append("heavy_attack")
    animation.play(state)
    attack_cooldown.start(HEAVY_ATTACK_COOLDOWN)
```

Three timers drive the pacing:

- **Attack cooldown** locks the player into an attack until it finishes. The opener
  is very short (0.1s) so starting a combo feels instant, and follow-ups are 0.6s so
  each hit has weight.
- **Combo timer** starts when an attack ends. Press the next button before it runs
  out and the combo continues; let it expire and `current_combo` is cleared.
- **Input buffer delay** controls when the game starts listening for your *next*
  input during an attack. More on that below.

## Making it feel fluid

The first version worked, but it didn't feel good. Attacks only
registered if you pressed the button while the player was idle, inside the combo
window. In practice, players press the next button *during* the current swing.
Those presses were silently dropped, so combos felt unresponsive and random.

The fix was an input buffer. If you press heavy or magic while a heavy or magic
attack is still playing, the input is stored. As soon as the attack's cooldown
ends, the stored input is replayed as if you had just pressed it:

```gdscript
func _process(_delta):
    if input_buffer.size() == 1 and attack_cooldown.time_left <= 0:
        var press = InputEventAction.new()
        press.action = input_buffer[0]
        press.pressed = true
        var release = InputEventAction.new()
        release.action = input_buffer[0]
        release.pressed = false
        Input.parse_input_event(press)
        Input.parse_input_event(release)
        input_buffer.clear()
```

Replaying through `Input.parse_input_event` meant the buffered press goes down
exactly the same code path as a real one, so I didn't have to duplicate any combo
logic.

The buffer follows a few rules:

- **It only holds one input.** Mashing doesn't queue up a whole combo in advance.
- **It doesn't open right away.** The buffer only accepts input after a short delay
  into the attack. Without that, a quick double-tap would fire the next attack before
  the first one had visibly landed, which made hits feel weightless.
- **Dodging clears it.** If you roll out of an attack, a stale buffered input
  shouldn't fire the moment the roll ends.

Most of the jam's combo work was tuning those numbers until the combo felt fluid
*and* each hit still felt impactful. Those two goals pull against each other, and
the right balance only showed up by playing it over and over.

## Keeping a stick figure readable

The player is a black stick figure, and the dungeon is dark. To keep the player
readable, I wrote a small `canvas_item` shader for the sprite. For every
transparent pixel, it looks a few pixels down and to the right. If it finds part of
the stick figure there, it paints that pixel white. The result is an offset outline
that makes the figure pop against the background, while the art stays simple
black lines:

```glsl
if (length(current_color - background_color) < 0.01) {
    vec4 offset_color = texture(TEXTURE, UV - offset_in_pixels * SCREEN_PIXEL_SIZE);
    if (length(offset_color - background_color) > 0.01
        && length(offset_color - vec4(0, 0, 0, 1)) < 0.5) {
        current_color = shadow_color;
    }
}
```

## What I'd do differently

With more time, I'd have pushed the animations further and added more visual
effects to the combos, especially the later stages. A finisher should feel like a
payoff, and ours look too similar to the hits that lead into them.

The bigger lesson is about order of work. We built the combat and the animations at
the same time, which tied the two together. The magic projectiles, for example,
spawn when their animation finishes, so changing the timing meant touching the art
as well. Next time, I'd prototype combat with placeholder shapes and no animation at
all, get the *feel* right first, and then fit the art to the timings.

## Play it

DiLemma is free to play in the browser on
[itch.io](https://tigershillo.itch.io/dilemma). Try chaining `J` `L` `L` for
the Empowered Fireball.
