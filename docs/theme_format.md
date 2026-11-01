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
| `reverb` | `mix`, `damping`, `size` | **read but not yet applied by MCSE** |
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

**Every recording of a loop layer must be the same length and start on the same
beat.** This is the one rule the file format does not state and the whole design
depends on: switching recordings is a crossfade at the same play position, so if
two recordings of a layer disagree about where bar one begins, the switch will
be out of time. In a real theme every loop recording of a 32-bar, 90 bpm piece
is the same 4,116,001 frames long.

`length` is declared per recording and MCSE reads it, but does not rely on it;
it was observed not to match the shipped audio exactly (the shipped files were
about two seconds longer, apparently padded on export), so the audio itself is
the authority.

## What MCSE does not implement yet

Honest list, so nobody is surprised:

- **`reverb`** per track — NVGT has `audio_freeverb_node`, so this is
  straightforward and simply not done.
- **`rendezvous`** — the separate agility and fade used when several layers must
  meet at a transition.
- **`pickup`** beats are parsed and not acted on.
- **`mode`** is stored and otherwise ignored.
