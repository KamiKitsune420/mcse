# MCSE — the Music Crossfade Switching Engine

Layered adaptive music for [NVGT](https://nvgt.dev). Your game pushes one
number at it — how much trouble the player is in — and the music answers.

```nvgt
#include "mcse_theme_xml.nvgt"

mcse_theme@ theme = mcse_load_theme_file("music/boss/boss.epro");
mcse_player@ music = mcse_player(theme, "music/boss/");
music.start(1, 500);          // intensity 1, fading in over half a second

// ...later, in your game loop
music.loop();
music.set_intensity(enemies_near_me * 2 + 1);
if (boss_appears) music.play_stinger();
```

## What it does

A piece of music is a set of **layers** — bass, strings, brass, percussion —
and each layer holds several alternative **recordings of the same loop**, one
per intensity and musical key. All of them are the same length and bar-aligned,
so moving a layer from one recording to another is a crossfade at the same play
position and is always in time. Nothing has to be composed as a transition.

That is the whole trick, and it is why this works:

| | |
|---|---|
| layers play | simultaneously, in lockstep |
| intensity changes | crossfade each layer to its recording for that intensity |
| key changes | the same, to the recordings in the new key |
| stingers | fired once, over the top |
| transitions land | on a beat, so the music never lurches |

## Why it is accurate

Crossfades are scheduled, not ramped frame by frame. Both sides of a switch are
handed to `sound.set_timed_fade_in_frames` at the **same absolute frame** of the
audio engine's own clock, so the two halves are welded together no matter what
your game loop is doing. Positions and lengths are in frames throughout, never
milliseconds.

## Installing

Copy `include/mcse.nvgt` and `include/mcse_theme_xml.nvgt` next to your script,
or pass `-I` to the compiler. No plugin, no DLL, no build step.

## Themes

MCSE reads the XML theme format that Elias Studio exports, so themes authored
with that tool work, and a theme can equally be written by hand in a text
editor — see [docs/theme_format.md](docs/theme_format.md), which documents the
format including which attributes are required and which pairs of elements may
not both appear.

Audio is loaded through `sound.stream` by default. A game whose music lives in
a pack supplies its own loader, exactly as NVGT's own `music.nvgt` does:

```nvgt
sound@ from_my_pack(sound@ s, string filename) {
	return s.stream(filename, my_pack) ? s : null;
}
mcse_player@ music = mcse_player(theme, "music/boss/", from_my_pack);
```

## Choosing a recording

Given an intensity, a layer plays **the highest recording at or below it**,
falling back to its lowest if it has nothing that low. That rule is MCSE's own
and it is deliberately simple, deterministic and monotone: raising the
intensity never moves a layer to a quieter recording. Compose against it and
you always know what you will hear.

`tests/choices.nvgt` prints the whole table for a theme and fails if it is not
monotone:

```
Bass:    1 1 1 1 1 1 7 7 7 7 11 11 11 11 11 16 16 18 18
Strings: 1 2 3 4 5 5 5 8 9 10 10 10 13 14 14 16 16 16 19
Brass:   1 2 3 4 4 6 6 6 9 9 9 9 13 13 13 13 17 17 17
Perc:    2 2 2 2 5 5 5 8 8 8 8 12 12 12 15 15 15 15 15
```

## Examples and tests

```bash
nvgt -I include tests/choices.nvgt -- path/to/theme.epro   # no audio, no window
nvgt -I include examples/play_theme.nvgt -- path/to/theme/folder
```

`play_theme` plays a theme and lets you steer it: up and down change intensity,
left and right change key, space fires a stinger, escape stops.

## Independence

MCSE shares no code with any commercial product. It reads a documented file
format and implements layered playback over NVGT's own audio API. See
[docs/provenance.md](docs/provenance.md) for exactly what was and was not
examined, and why that matters.

## Licence

zlib, the same licence NVGT's own includes use. See [LICENSE](LICENSE).
