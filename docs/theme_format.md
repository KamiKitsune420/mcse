# The theme format

A theme is one XML file — conventionally `.epro` — plus a folder of audio. It
is plain text, so you can write one by hand with no authoring tool.

The format below is documented from two independent directions that agree: five
themes a composer actually shipped, and the 31 validation messages the original
library emits when it rejects one. Where a message says an attribute is
required, it is listed as required here.

```xml
<theme volume="0" channels="2" mode="objective" tempo="90"
       time_signature="4/4" sample_rate="44100" bars="32">
    <defaults>
        <fade_out minimum="500" maximum="10000"/>
        <crossfade minimum="50" maximum="250"/>
        <fade_in minimum="50" maximum="1000"/>
        <agility value="4" strict="no"/>
        <pickup beats="0"/>
        <progression mode="random"/>
    </defaults>
    <audio_loop_track volume="0" id="1" name="Bass">
        <crossfade minimum="6000" maximum="12000"/>
        <agility value="8" strict="no"/>
        <key value="0" name="C">
            <trigger value="1"  length="8035414" filename="audio/1_0_1.wav"/>
            <trigger value="7"  length="7978302" filename="audio/1_0_7.wav"/>
            <trigger value="11" length="7647418" filename="audio/1_0_11.wav"/>
        </key>
        <key value="1" name="D">
            <trigger value="1"  length="7618126" filename="audio/1_1_1.wav"/>
        </key>
    </audio_loop_track>
    <audio_stinger_track volume="0" id="5" name="Timpani">
        <key value="0" name="C">
            <stinger value="1" length="300256" filename="audio/5_0_1.wav"/>
        </key>
    </audio_stinger_track>
</theme>
```

`<theme>` is the root. There is no XML declaration and no wrapper element.

## Elements

| element | attributes | notes |
|---|---|---|
| `theme` | `tempo`, `time_signature`, `bars`, `sample_rate`, `channels`, `mode` **all required**; `volume` optional | the root |
| `defaults` | — | settings every track inherits unless it overrides them |
| `audio_loop_track` | `id` **required**; `name`, `volume` | a layer that plays continuously |
| `audio_stinger_track` | `id` **required**; `name`, `volume` | a layer fired once, over the top |
| `key` | `value`, `name` | a musical key; `value` is what `set_key` takes |
| `trigger` / `stinger` | `value`, `length`, `filename` | one recording. `value` is the intensity |
| `crossfade` | `minimum`, `maximum` | milliseconds |
| `fade_in`, `fade_out` | `minimum`, `maximum` | milliseconds |
| `reverb` | `mix`, `damping`, `size` | 0 to 100 each; applied through `audio_freeverb_node` |
| `agility` | `value`, `strict` | `value` is in beats: how often a switch may happen |
| `agility_points` | holds `<beat point="N"/>` children | specific beats instead of a regular interval |
| `pickup` | `beats` | a lead-in before the switch point |
| `progression` | `mode` | the original randomises on `random`; **MCSE ignores this and is always deterministic** |
| `rendezvous` | holds `agility`/`agility_points` and `fade` | **read but not yet applied by MCSE** |

## Rules the validator enforces

- All six `<theme>` attributes above must be present.
- A theme must have at least one track.
- Every track must have an `id`.
- **`agility` and `agility_points` may not both appear** in the same scope —
  they are two ways of saying the same thing. MCSE rejects a theme that has
  both, with the same message.
- `<beat>` requires `point`; `<pickup>` requires `beats`; `<progression>`
  requires `mode`.

## The audio

Filenames are relative to the theme file's folder. The convention a real theme
uses is `audio/<track id>_<key>_<intensity>.wav`, but nothing requires it —
MCSE uses whatever `filename` says.

**Every recording of a loop layer must be the same bars at the same tempo**, so
that bar 9 of one take is bar 9 of every other. Switching takes is a crossfade
at the same play position, so if two takes disagree about where bar one begins,
the switch lands in the wrong place.

**They do NOT have to be the same length, and this file previously said they
did.** That was wrong, written from the first few rows of a table rather than
all of it. Measured properly, a real theme's 54 loop recordings come in four
different lengths:

| length | seconds | bars at 90 bpm, 4/4 | files |
|---|---|---|---|
| 3,998,40x | 90.667 | 34 | 14 |
| 4,057,200 | 92.000 | 34.5 | 1 |
| 4,116,00x | 93.333 | 35 | 33 |
| 4,233,60x | 96.000 | 36 | 6 |

while the theme declares **32 bars = 85.333 s** -- shorter than every single
file. So `bars` is the **loop point**, and the files run on past it carrying a
release tail of 2 to 4 whole bars, each as long as that instrument needs to
ring out.

**Which means a layer must loop at the declared bar count, not at the end of
its file.** Loop at end of file and every layer comes round at a different
moment and the piece pulls apart after one pass -- while sounding perfect for
the first minute and a half, which is long enough to be called working.
`tests/drift.nvgt` is the check for exactly that.

What must hold is only that **every file is at least as long as the loop**,
which `tests/validate.nvgt` enforces.

`length` is declared per recording and MCSE reads it but does not rely on it:
it does not match the shipped audio, so the audio itself is the authority.

### Two frame units, which is an 8.8% error waiting to happen

Worth stating because it passes silently. A `sound`'s frames -- `position_in_frames`,
`length_in_frames`, `seek_in_frames`, the timed fades and the engine clock --
are all in the **engine's output rate**, which is the audio device's. An
`audio_data_source`'s frames are in the **file's** rate. Where the device runs
at 48000 and a theme is 44100, a 3,998,400-frame file reports 4,352,000 as a
sound: exactly 48000/44100 of it.

So the loop point, which lives on the data source, is set in file frames, and
every fade and position is handled in engine frames. Mixing them makes every
crossfade 8.8% short, which sounds almost right.

And a **streamed** sound's `length_in_frames` is not reliable at all before it
has loaded -- it reported figures several seconds out. `validate.nvgt` uses
`load` rather than `stream` for that reason.

## What MCSE does not implement yet

Honest list, so nobody is surprised:

- **`reverb`** per track is implemented, through `audio_freeverb_node`: the
  layer is routed through it and the theme's `mix` becomes freeverb's own wet
  and dry balance, so there is no second summing stage. Turn it off with
  `player.use_reverb = false`.
- **`rendezvous`** — the separate agility and fade used when several layers must
  meet at a transition.
- **`pickup`** beats are parsed and not acted on.
- **`mode`** is stored and otherwise ignored.
