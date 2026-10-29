---
name: yt-clips
description: >-
  Turn ONE long-form video (YouTube link or local mp4) into 3-8 short-form
  clips ready for Reels/TikTok/Shorts. Selection (what's clippable), assembly
  (reverse-storytelling: hook to the front, end on the climax), hard gates,
  scoring, and ffmpeg render mechanics — with a human approval gate before
  anything is published. Use when you say "clip this video", "cut shorts from
  this", or drop a YouTube URL and want clips.
---

# /yt-clips

Turn one long-form video into a small set of short clips a stranger would
actually watch. The craft is in WHAT you select and HOW you reorder it — the
editing polish comes after, in your own overlay/caption/scheduling workflow.

## Read first, every run
1. `references/reverse-storytelling.md` — the assembly method (the spine).
2. `references/selection-and-scoring.md` — selection markers, scoring rubric,
   platform lengths, hook/edit rules.

## Pipeline

### Step 1 — Source + word-level transcript
- YouTube URL → `yt-dlp -f "bv*[height<=1080]+ba" --merge-output-format mp4`
  (keep yt-dlp updated — a stale version throws HTTP 403).
- Local mp4 → use as-is.
- Transcribe LOCALLY with word timestamps (free, private):
  `whisper <file>.mp4 --model large-v3-turbo --language en --output_format json --word_timestamps True`
  (or faster-whisper). Transcribe ONCE — every later step reuses this JSON.

### Step 2 — Selection
Read the FULL transcript (no premature summarising). Mark every segment that
hits a clippability marker (reference §2): contrarian take, specific numbers,
complete micro-story, tactical how-to, list moment, emotional spike,
confession, quotable claim, before/after, myth-bust.
Expect 8-15 candidates per hour of source. Fewer than 3? Say so — never pad
with weak segments.

### Step 3 — Reverse-storytelling assembly (per candidate)
1. Isolate the segment where the concept lives.
2. Find TWO sentences (verbatim, word-timestamped): the **climax** (payoff)
   and the **hook** (an open loop that points AT the climax). They may be far
   apart, in either order.
3. Reorder: hook to position 0; climax lands at ~60% depth of the FINAL clip.
4. The context bridge (the story/detour that supports the point) goes between.
5. Ruthless cuts — exactly two kinds: repeated explanations (keep the better
   one) and anything off the hook→climax through-line. Strip fillers and
   warm-up ramble; first spoken word within 0.5s.
6. **End the clip the moment the climax lands.** Nothing after.

Output per clip = an EDL: an ordered list of [start, end] spans from the
source. A good clip is usually 2-5 NON-contiguous spans.

### Step 4 — Hard gates, then score
Gates (fail either → discard): **self-containment** (a stranger gets the full
meaning; no "like I said earlier") and the **send test** (would someone send
this to a friend?).
Score survivors /10: hook 35%, self-containment 20%, payoff 20%, emotional
charge 15%, value density 10%. **The score RANKS candidates — a human picks.**
Every AI clipper's auto-score is documented as unreliable.

### Step 5 — Render ALL survivors for review
Render every gated candidate into a review folder, filenames sorted by score
(`01-<slug>.mp4` = highest). Add a `clip-sheet.md`: per clip — hook line +
climax line (verbatim), EDL spans, runtime, score with one-line reasoning per
dimension, which platform the length favours, and a suggested on-screen hook.
**Then build the review SPREADSHEET** — this is how the human edits clips by
text instead of in an editor. One sheet per video, titled
`yt-clips — <video-slug>`, one row per span, in clip order:

| Clip | Order | Text (verbatim) | Start | End | Keep | Notes |
|---|---|---|---|---|---|---|
| 01 | 1 | "hook line…" | 04:12.3 | 04:15.1 | Y | |

- If a Google Sheets connection is available (a Google Workspace MCP or
  connector), create it as a Google Sheet, freeze the header row, widen the
  Text column, and put the link in `clip-sheet.md` AND in chat. Write
  timestamps as plain text (RAW), or Sheets turns them into times.
- No Sheets connection? Write `review.csv` in the review folder instead —
  it opens in Google Sheets, Excel or Numbers. Same columns.
- Want it somewhere else (Notion, Airtable)? Same columns, their tool.
- How the human edits: flip Keep to N (drops that line), renumber Order
  (moves it), or write a note ("end here instead", "tighten this"). It's
  filmed footage: spans can be CUT or REORDERED, never rewritten. If a note
  asks for words that were never said, say so rather than faking it.

Tell the human: the folder, the sheet link, the ranked list (one line each),
and "pick the ones you want, or edit the sheet and say re-cut."
**APPROVAL GATE: nothing publishes until the human names their picks.**

### Step 5b — Re-cut from the sheet
When they say "re-cut" or "I edited the sheet": read the whole sheet back
(explicit range, e.g. `A1:G200`), rebuild each clip's EDL from the rows
(Keep = Y only, sorted by Order, honouring Notes), re-render over the old
file in the same folder, and say what changed per clip. Loop until they pick.
Once the clips are consistently right, it's fine to skip the sheet review
and go straight to picking.

### Step 6 — Render mechanics
- Cut every span with its own seek-decoded input in ONE command — never a
  single linear decode with many trims (long-source linear decode can smear
  faces with stale blocks), never stream-copy (cuts land mid-GOP):
  `-ss <start> -to <end> -i source.mp4` per span → concat FILTER → one encode.
- Pad span edges ~0.08-0.18s so words don't clip.
- **9:16 framing — default to full-frame blur-pad** (keeps the whole 16:9
  frame, including any baked graphics, over a blurred fill):
  `[v]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,boxblur=luma_radius=40:luma_power=2[bg];[v]scale=1080:-2[fg];[bg][fg]overlay=(W-w)/2:(H-h)/2`
  Centre-crop only for clean talking-head with nothing baked in — check
  frames first.
- Verify every render: duration matches the EDL, first word audible ~0.5s,
  ends on the climax, video/audio durations within one frame.

### Step 7 — Finishing for publish (the picks only)
- **On-screen hook from frame 1** — 3-6 punchy words that foreshadow the
  climax harder than the spoken hook. On blur-pad clips, place it in the TOP
  blur zone (never over a face); fade it out ~3s in.
- **Captions**: small one-word karaoke captions in the lower third, BELOW the
  video band on blur-pad clips. Build them from the SAME word JSON mapped
  through the EDL (span offset = sum of previous span durations) — no
  re-transcription needed.
- **Loudness**: platforms normalise DOWN to target but never boost — land ON
  the target. Instagram/TikTok −14 LUFS integrated (YouTube −13..−15, so −14
  works everywhere). `loudnorm=I=-14:TP=-1.5:LRA=11`; verify the FINISHED mp4
  with `ffmpeg -hide_banner -i out.mp4 -af ebur128=framelog=quiet -f null -`
  (Summary `I:` = −14 ±0.5). If single-pass undershoots, run measured two-pass
  (`measured_I=...` + `linear=true`), and keep peak ≤ −1 dB (alimiter
  `limit=0.85:level=false` — measure the finished file, not the chain).

## Traps learned the hard way
- Whisper stores contractions as single tokens ("I'm", "we'll") — phrase
  anchors must use natural apostrophes.
- **Collapsed-timestamp blobs**: around stumbles/repeats whisper can stamp a
  whole run of words with one timestamp. Cutting inside a blob grabs unknown
  audio — pick a different span boundary or drop the span; the fact you
  wanted often exists cleanly elsewhere in the transcript.
- Only merge adjacent spans when the next span moves FORWARD in source time —
  naive overlap-merging corrupts reordered clips.
- Search-window edges: when phrase-matching by time window, pad the window a
  couple of seconds each side — the first word often starts just before your
  estimate.
- 720p source → 1080x1920 is soft but usable; flag it.

## Don'ts
- Don't auto-pick clips — the score ranks, the human picks.
- Don't keep source order out of loyalty — moving the hook to the front IS
  the method.
- Don't let a clip run past the climax.
- Don't use cloud transcription.
- Don't pad weak videos to hit a clip count — 2 great clips beat 6 mediocre.
