# Provenance — what was examined, and what was not

MCSE is an independent implementation of layered adaptive music. It is
**based on an understanding of how such a system behaves**, not on anybody
else's source code or compiled code. This document records exactly how that
understanding was obtained, because the distinction is the whole basis on which
this project can be open source.

## What was examined

A commercial adaptive-music library, `elias.dll` by
[Elias Software](https://eliassoftware.com), ships inside a 2015 game. Four
things about it were looked at, all of them from the outside:

1. **Its export table.** The names of the 92 functions a program may call, read
   out of the PE export directory. A list of names, not their code.
2. **A game's calls into it.** How one program arranges its arguments before
   calling, read from that *game's* own instructions — the game's code, not the
   library's.
3. **Its error strings.** The 31 messages it emits when it rejects a theme file,
   which between them describe which attributes a theme must carry.
4. **Theme files.** The XML a composer's authoring tool exports, which ships
   unencrypted inside that game's assets.

## What was never examined

**The implementation.** `elias.dll` was never disassembled, never decompiled,
and no algorithm inside it was read, transcribed, inferred from its code, or
reimplemented from its code. Not one instruction of it was looked at.

That was a deliberate choice made before any of this began, and it was kept.

## What MCSE therefore has, and does not have

| | |
|---|---|
| reads the same theme file format | **yes** — reading a documented data format is not copying an implementation |
| shares any code | **no** |
| shares any algorithm taken from the library | **no** |
| mixes, crossfades or schedules the way it does | **unknown, and not copied.** MCSE uses NVGT's audio engine and its own rules |
| chooses recordings the way it does | **no, and deliberately not.** Theirs is nondeterministic; MCSE's rule is its own, deterministic and documented |

The last row is worth expanding, because it is the clearest evidence that these
are different engines. Rendering one theme at one intensity through the
commercial library gave different recordings on different runs — consistent
with the `progression mode="random"` its own themes declare. MCSE instead picks
the highest recording at or below the intensity asked for, always, and a test
in this repository fails if that is ever not monotone. A composer can therefore
know what MCSE will play. That is a design decision, not an imitation.

## Naming

MCSE is not called ELIAS, does not use that name in its code, documentation or
output, and claims no connection to, endorsement by, or compatibility
certification from Elias Software. Where this project refers to that product it
does so to say what file format MCSE reads.

If you want ELIAS itself, license it from Elias Software. It is a good product
and this is not a replacement for it.

## A note for contributors

Please keep it this way. Do not paste in anything derived from a disassembly or
decompilation of a commercial audio library, and do not implement something
"the way ELIAS does it" on the basis of having read its code. Implement what
sounds right, measure it, and write down why.
