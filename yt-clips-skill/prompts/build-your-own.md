# Build this around YOUR workflow

Rather than installing this skill verbatim, you can paste the prompt below
into Claude Code and let it build a version wired to your own tools.

---

I want a "yt-clips" skill for Claude Code that turns one long-form video into
short-form clips. Build it for me using this outcome spec:

1. INPUT: a YouTube URL or local mp4. Download with yt-dlp if needed, then
   transcribe locally with Whisper (word timestamps on).
2. SELECTION: read the whole transcript and flag clippable segments —
   contrarian takes, specific numbers, complete micro-stories, tactical
   how-tos, lists, emotional spikes, confessions, quotable claims,
   before/afters, myth-busts.
3. ASSEMBLY: for each candidate use reverse storytelling — find a hook (open
   loop) and a climax (payoff), move the hook to the front, put the
   supporting material between, land the climax ~60% in, and END the clip the
   moment the climax lands. Output an EDL of word-timestamped spans.
4. GATES + SCORE: discard anything a stranger wouldn't fully understand or
   wouldn't send to a friend. Score the rest /10 (hook 35%, self-containment
   20%, payoff 20%, emotion 15%, value density 10%) — but I pick the winners.
5. RENDER: cut each span with its own -ss/-to input into one ffmpeg concat
   command (re-encode, never stream copy), blur-pad to 9:16, and put
   watchable review files + a clip sheet in a folder for me. Also give me a
   review spreadsheet: one row per span (Clip / Order / Text / Start / End /
   Keep / Notes). When I flip Keep to N, renumber Order or leave a note and
   say "re-cut", read it back and re-render.
6. FINISH (my picks only): on-screen text hook from frame 1 (in the top blur
   zone, never over a face), small word-synced captions in the lower third,
   and normalise audio to −14 LUFS integrated with peak ≤ −1 dB.

Ask me: where I want review files saved, where the review sheet should live
(Google Sheets, a CSV, Notion), which platforms I publish to, and
whether I want it to connect to my scheduling tool.
