# Writing music for MCSE

You do not need an authoring tool. A theme is one XML file you can type by
hand plus a folder of audio, and this is what the audio has to be.

## The shape of a piece

Pick a tempo, a time signature and a length in bars. Everything follows from
those three.

Then decide your **layers** — four is a good number, and a real theme uses
bass, strings, brass and percussion. A layer is one instrument group that can
come and go.

For each layer, record several **takes of the same bars** at different
intensities. A take at intensity 1 might be the bass playing roots on the
downbeat; at intensity 11 the same bars with the bass playing eighths. They
are not different music. They are the same music played with more or less
urgency.

## The four rules that actually matter

**1. Every take of every layer is the same bars at the same tempo.** This is
the whole design. MCSE moves a layer from one take to another by crossfading
at the same position in the music, so bar 9 of one take has to be bar 9 of
every other take. Get this wrong and switches land in the wrong place.

**2. Record longer than the loop, and let the tail ring.** The loop is `bars`
bars. The *file* should be longer — keep recording while the reverb and the
instruments die away. MCSE loops at the bar count you declared, not at the end
of the file, so the extra is there to be crossfaded out of rather than played.
A real theme declares 32 bars, and its files carry **2 to 4 extra bars** of
tail, each as long as that instrument needs.

**3. Every file must be at least as long as the loop.** A file shorter than
`bars` bars cannot complete a pass. `tests/validate.nvgt` checks this and will
tell you which file is short and by how much.

**4. Layers do not have to agree about file length.** Only about the music.
Your percussion can stop ringing in two bars while the strings take four —
that is normal, and it is why MCSE loops at the declared bar count instead of
at end of file.

## Naming

Anything works; the theme names each file explicitly. The convention a real
theme uses is readable and worth copying:

```
audio/<layer id>_<key>_<intensity>.wav
audio/1_0_7.wav      layer 1, key 0, intensity 7
```

## Intensities

Number them however you like — they are just thresholds. MCSE plays **the
highest take at or below** the intensity your game asks for, so:

```
<trigger value="1"  .../>    plays at intensity 1 to 6
<trigger value="7"  .../>    plays at 7 to 10
<trigger value="11" .../>    plays at 11 and up
```

You do not need every layer to have takes at the same intensities. Give the
percussion three steps and the strings twelve if that is what the music wants.
A layer with nothing at or below the asked-for intensity plays its lowest take.

Keep the numbers meaningful to your game. If your intensity is "enemies
within earshot", let the takes sit at the counts where the music should
change.

## Keys

If your piece can move between keys, record each take once per key and give
them `key` values. Your game calls `set_key`. If it cannot, use one key and
never call it.

## Stingers

A stinger is a one-shot over the top — a hit when the boss appears. Put it in
an `audio_stinger_track` and fire it with `play_stinger()`. It is not looped
and it has no length requirement.

## How often the music may change

`agility` is in beats and says how often a layer is allowed to switch. With
`agility="8"` a layer only ever changes on an eight-beat boundary, so a frantic
game cannot make the music twitch. Longer for slow-moving layers like bass,
shorter for percussion.

`crossfade` gives a minimum and a maximum in milliseconds. MCSE scales between
them by how far the intensity moved: a one-step nudge gets the short fade, a
leap across the range gets the long one. Set both to the same number for a
fixed length.

## Before you ship

```bash
nvgt -I include tests/validate.nvgt -- music/boss/boss.epro   # files and lengths
nvgt -I include tests/choices.nvgt  -- music/boss/boss.epro   # what plays when
nvgt -I include tests/drift.nvgt    -- music/boss 95          # loops stay together
```

`choices.nvgt` prints the take each layer picks at every intensity, which is
the fastest way to see whether your thresholds say what you meant. `drift.nvgt`
plays for longer than one loop and checks the layers are still together
afterwards — the one failure that sounds perfect for the first ninety seconds.

## What MCSE does not do yet

- `rendezvous`, the separate timing used when several layers must meet at a
  transition, is read and ignored.
- `pickup` beats are read and ignored.
- `progression`, which in the format means "pick a take at random", is ignored:
  MCSE is always deterministic, so you can know what you will hear.
