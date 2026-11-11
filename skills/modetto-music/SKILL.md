---
name: modetto-music
description: Use when an agent makes, extends, covers or judges songs in Modetto, the local AI music app for Mac, through its `modetto` command line or its MCP server. Gives the workflow, how to prompt YuE2, how to keep one voice across songs, how to read the musician's feedback before rendering, and the faults to check before showing a take.
---

# Making songs in Modetto

Modetto generates songs with vocals on the Mac it runs on, with the YuE2-3B model. An agent drives it through the
`modetto` command, at `~/Library/Application Support/modetto-desktop/bin/modetto`, or through the MCP server
`modetto-mcp` in the same folder. The app must be open. Add `--json` to any command for machine-readable output.

The musician decides what is good. Your job is to make takes worth hearing, say how each was made, and learn from
what they rate.

## Before a song

1. **Read what the musician thought of earlier takes.** `modetto feedback --search <words> --quality` lists each
   take's rating, comments and the reason it was deleted, next to the style line, seed and options that made it.
   Search for a title, style words or the name the musician uses for a sound. Don't repeat a combination they
   disliked without saying what changed, and reuse what they starred.
2. **Show the plan before rendering.** Give the idea, the style line, the voice, the length and the full lyrics,
   and render once the musician agrees. A render takes minutes, and a wrong brief wastes them.
3. **Keep a notes file per sound.** Record the exact style line, the seeds that worked and what each render taught.
   Change one thing per render, so a result can be traced to its cause.

## Rendering

```bash
modetto render --style "melodic house, soft female vocal" --lyrics lyrics.txt --seed 7 --seconds 120 --title "Night Train"
```

- Each new take is saved to the Library, where the musician can play and rate it. Use `--no-save` only for tests
  nobody will hear.
- Songs wait in one queue with the musician's own. `modetto jobs` shows the queue, and `modetto cancel q12` takes a
  waiting song out. Never quit Modetto while a song is in the queue.
- On an M1 Pro, a two-minute song takes about 3 to 4.5 minutes with `--cot off` and 12 to 16 minutes with
  `--cot full`. Use `off` unless the song needs YuE2 to plan its own melody first.

## Prompting YuE2

- **Style line:** genre, mood, instruments and production words, then a precise vocal description: gender,
  register, texture, delivery and language. Keep it word for word the same across one artist's songs.
- **Lyrics:** `[Verse]`, `[Chorus]`, `[Bridge]` and `[Outro]` tags with a blank line between sections. Each section
  gives about 20 to 30 seconds of music, so the number of sections sets the length.
- **Languages:** English and Mandarin are reliable. Japanese and Spanish often hold for the first verse and then
  blur. Swedish fails: YuE2 invents lines and loses the hook.
- **Instrumental space:** empty `[Instrumental]` or `[Break]` sections do not make YuE2 sing less. A song with long
  instrumental passages is built after rendering, in the editor, from the take's own phrases.
- **Length:** `--seconds` is a cap. Leave spare seconds, so YuE2 can play an ending instead of being cut off at full
  level. Never set `semantic_min_tokens`: when YuE2 has sung every line before the minimum, it fills the rest with
  silence and often starts the music again later.
- **Seeds:** the same style line, lyrics, seed and options give the same song again. Any change, even a guidance of
  1.0 against 1.01, gives a different take. Judge a setting over at least three seeds, never one.

### Options

Pass these with `--option KEY=VALUE`.

| Option | Default | What it changes |
| --- | --- | --- |
| `guidance_scale` | 1.01 with cot off, else 1.0 | How closely YuE2 follows the style line. At 3.0 the named instruments came through in three seeds of three, against none at 1.01. Use 3.0 when the arrangement matters |
| `semantic_temperature` | 1.0 | How much takes differ. 1.2 gave wider melodies without breaking songs; 0.85 was not measurably closer than 1.0 |
| `semantic_top_p`, `semantic_top_k` | 0.95, 100 | How many candidate tokens are allowed at each step |
| `semantic_repetition_penalty` | 1.2 | Discourages loops; set it too high and choruses break |

## One voice across songs

YuE2 has no voice-reference input. What keeps a singer, strongest first:

1. **Extend the same song.** `modetto extend <track> --seconds 30 --lyrics new.txt` adds to the end, and
   `--from <seconds>` rewrites the song from that point. Extend only from inside the music: from a faded ending, YuE2
   continues with silence.
2. **A voice anchor.** `modetto render --anchor <track> --anchor-seconds <n>` starts the new song from the opening of
   a song with the wanted singer, then cuts that opening off. In tests this held the timbre closer than any other
   method. Two things come along:
   - The anchor's key and tempo. Move the key afterwards with the editor's semitone shift on the whole mix, never
     on the vocal alone.
   - The anchor's words. Set `--anchor-seconds` to end right where the anchor's first sung section ends. Cut
     earlier, and the new song begins with the rest of that section.

   An anchor with no singing in it carries the arrangement, not the voice.
3. **The same style line and seed.** This gives the same song again, but new lyrics with the same seed do not
   reliably keep the voice.

More adjectives do not make a new voice: text-only variations stay among YuE2's house voices. Use only voices the
musician has the rights to: their own, a singer who agreed, or one invented with YuE2.

## Checking a take before showing it

- **Gaps:** look for silence of 3 seconds or more inside the song. Shorten it to about one bar, cutting inside the
  silence.
- **Endings:** a song that stops at full level hit the length cap. Render a continuation from a point inside the
  music and join only its tail on. The last 10 ms of a finished song should sit below about -60 dB; anything louder
  is a cut-off echo.
- **Leaked lyrics:** check that the first line sung is the first line written, especially after an anchor.
- **Harsh highs:** YuE2 renders cymbals, breaths and S sounds as fizz. `modetto smooth <track>` saves a copy with
  them turned down and leaves the original as it is.
- **Thin passages:** breaks and sparse outros expose model noise. Put joins and endings inside full passages.

Give the musician the Library title and track number of each take, and how it was made.

## Editing and mastering

- **The least processed version usually wins.** Every step costs some quality: splitting into stems and joining them
  again, formant shifts, extra compression, added reverb. Process only to fix a fault you can name.
- **When the musician names one fault, fix that one.** Change the one setting and keep everything else identical.
  Offer other improvements separately.
- **Repeat and cut on phrases.** Electronic music moves in 8-bar phrases. A repeat or a cut that ignores the phrase
  line sounds like a skip, even when it lands on a bar line.
- **Master from the edited take the musician starred,** never from the raw render. `modetto song <track>` lists
  every take of a song, and `modetto feedback <track>` shows what was said about each one.
- **Group the takes of one song.** A track made from another joins its song by itself. `modetto group` and
  `modetto ungroup` fix the cases that go wrong.

## Learning from the musician

- Read ratings and comments before every new render, as in step 1 above.
- Write what was learned on the take itself: `modetto feedback add <track> "chorus is right, voice too thin"`.
- Delete with a reason, so the next agent learns from it:
  `modetto delete <track> --yes --reason wrong-voice --note "too breathy"`. Use `--reason cleanup` for tests and
  working files; it is never read as a verdict on the music.
- Remove what your tests added to the Library before you finish, with the reason `cleanup`.

## Rules

- Don't feed in music, lyrics or voices the musician has no rights to, and don't use Cover on such a song.
- Settings, setup and model licences stay in the app. The licence of each model is listed at
  https://modetto.app/#models.
- Never quit or restart Modetto while a song is in its queue.
