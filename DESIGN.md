# Lantern Keep: design document

An original game built to feel like an early-1980s Intellivision cartridge.
It runs in a browser (`index.html`, no build step), but every design decision
is checked against what the real console could do.

## Pitch

You are a lone archer in a walled keep. Each room has four cold lanterns.
Light all four and a stair appears; take it down to the next room. A bar at the
top of the screen counts down the Keeper's sleep. When it runs out, an
unkillable guardian walks out of the wall and hunts you. Meanwhile a heartbeat
in the speaker gets faster.

It belongs to the same family as *Night Stalker*, *Advanced Dungeons & Dragons*
and *Venture*: single-screen mazes, a small arsenal and steadily rising tension.

## What makes it feel like an Intellivision game

| Trait | How Lantern Keep uses it |
|---|---|
| 160 × 96 screen built from a 20 × 12 grid of 8 × 8 "cards" | The top card row is the HUD. The other 11 rows are the maze. Walls, lanterns, stairs, quivers and the bell are all background cards. |
| Fixed 16-colour palette | Only those 16 colours appear on screen. Each room recolours its walls with a different bg/fg pair from that palette. |
| 8 moving objects (MOBs) | Budget: player, arrow, Keeper, up to 5 creatures. The cap is why only one arrow can be in flight, and why killed creatures wait for a free slot before respawning. |
| Single-colour MOBs | Every sprite is one colour. The wraith flickers on alternate frames, the usual hardware trick for ghosts. |
| Keypad with a printed overlay | 1 Flare, 3 Bell, 5 Pause, 7 Sound, CLR Quit, ENT Start. Blank overlay keys do nothing, as on the real overlays. |
| 16-direction disc | Movement uses 4 directions because the corridors are exactly one card wide. Off-axis disc presses are snapped. |
| Side action buttons | Both side buttons fire. |
| AY-3-891x sound chip (3 tone voices + noise) | All sound is square, triangle or noise. Signature sounds: a heartbeat that speeds up, a two-tone siren when the Keeper is awake, and short arpeggios for lanterns and stairs. |
| Pixels taller than wide on a 4:3 TV | The 160 × 96 image is stretched to 4:3, so pixels come out at 0.8:1. |

## Core loop

1. **Room intro.** The banner shows the room number. A fresh maze is generated.
2. **Explore and light.** Walk into each of the 4 lanterns (+100 each).
3. **Survive.** Shoot creatures, use flares and the bell when cornered.
4. **Escape.** When all 4 are lit, the stair appears at the cell farthest from
   the start. Reach it for the room bonus.
5. Repeat with faster creatures, a shorter Keeper timer and new wall colours.

Lantern progress survives a death. Creatures and the Keeper reset.

## Maze generation

* A 9 × 5 grid of cells is carved with a recursive backtracker, giving a
  19 × 11 card maze with pillars between corridors.
* About 80% of dead ends are knocked through ("braiding") and 5 extra walls are
  removed. Loops matter: a maze with no loops is a death trap against chasers.
* Placement uses a breadth-first distance map from the start cell:
  stair = farthest cell, lanterns = spread-out cells at least 4 steps away,
  creature nests = cells at least 10 steps away.

## Creatures

| Creature | Movement | HP | Points | Appears |
|---|---|---|---|---|
| Bat | Random at each junction, 50% chance to keep going straight | 1 | 50 | Room 1 |
| Spider | Follows the distance map toward you (25% random turns) | 1 | 100 | Room 1 |
| Wraith | Drifts directly at you through walls, flickers | 2 | 200 | Room 3 |
| Keeper | Follows the distance map almost perfectly; arrows stun it for 2 s | — | — | When the timer empties |

Creatures never reverse in a corridor unless they reach a dead end. That gives
them the readable, Pac-Man-like flow players can learn and exploit.

## Player tools

* **Arrows**: start with 9, max 9. When you're down to 3 or fewer, a quiver
  (+5) appears somewhere at least 6 cards away.
* **Flare (key 1)**: freezes everything for 4 s, including the Keeper's timer.
  Frozen creatures are harmless and can be shot. Start with 3, +1 per room, max 5.
* **Bell (key 3)**: one per room. Spiders and the Keeper path to the bell for
  6 s instead of to you. Bats and wraiths ignore it.

## Difficulty curve

| Room | Bats | Spiders | Wraiths | Keeper timer |
|---|---|---|---|---|
| 1 | 1 | 1 | 0 | 45 s |
| 2 | 2 | 1 | 0 | 43 s |
| 3 | 2 | 2 | 1 | 41 s |
| 4+ | capped at 5 creatures total | | | −2 s per room, minimum 25 s |

Creature speeds rise by a few percent per room. The Keeper is slower than the
player in early rooms and faster than the player from room 6.

## Scoring

* Lantern 100 · Bat 50 · Spider 100 · Wraith 200
* Room bonus: 100 × room number, plus 10 per second left on the Keeper bar
* Extra life every 5,000 points (max 6)
* High score is kept in the browser's local storage

## Ideas for the next cartridge revision

* **Skill select on the title screen** with keypad 1–4, as many Intellivision
  games did.
* **Two-player alternating** play, with the controller swap prompt.
* **Speech lines** in the style of the Intellivoice module ("The Keeper wakes!").
* **Treasure rooms** every 5th room: no lanterns, just a timed loot grab.
* **A real ROM.** The design fits the hardware budget, so it could be ported to
  actual hardware with IntyBASIC and tested in the jzIntv emulator.
