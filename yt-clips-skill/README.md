# yt-clips — clip long-form video into shorts that actually hold

A Claude Code skill that turns one long-form video (YouTube link or local
file) into 3-8 short-form clips using reverse storytelling: find the hook and
the climax, move the hook to the front, cut everything off the through-line,
and end the clip the moment the payoff lands. You always get watchable review
files plus a review spreadsheet (one row per line: flip Keep to N, reorder,
leave a note, say "re-cut") and YOU pick what publishes.

## The easy way
1. Clone this repo (or download it as a zip) into `~/.claude/skills/yt-clips/`
2. Open Claude Code and say: *"read the yt-clips skill and set it up for me —
   check I have yt-dlp, ffmpeg and local Whisper, and install what's missing"*
3. Then: *"clip this video: <YouTube URL>"*

## The even-more-yours way
Open `prompts/build-your-own.md` and paste the prompt into Claude Code — it
builds the same workflow wired to your own folders and publishing tools.

## What's inside
- `SKILL.md` — the full pipeline (selection → assembly → gates → render →
  review sheet → approval → finishing)
- `references/reverse-storytelling.md` — the assembly method (credit: Briar
  Cochran)
- `references/selection-and-scoring.md` — what makes a moment clippable, the
  scoring rubric, platform lengths
- `prompts/build-your-own.md` — outcome prompt to rebuild it around your stack

## You'll need
- [Claude Code](https://claude.com/claude-code)
- `ffmpeg`, `yt-dlp` (Homebrew: `brew install ffmpeg yt-dlp`)
- Local Whisper (`pip install faster-whisper` or `openai-whisper`) — cloud
  transcription is deliberately not used

Optional: a Google Sheets connection (Google Workspace MCP or connector) so
the review sheet lands in Google Sheets. Without one you get `review.csv`,
which opens in Sheets, Excel or Numbers.

No accounts, no API keys, nothing leaves your machine until you publish.
