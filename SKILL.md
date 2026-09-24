---
name: curate-ai-interview-clips
description: Monitor selected YouTube AI and technology interview channels, remove previously handled videos, rank high-spread-potential topics, present up to three Chinese topic proposals for human approval, and only after approval download captions, select semantically complete 1-2 minute excerpts, translate subtitles line by line, cut video, and render 1080x1920 vertical clips. Use for daily AI interview radar, viral quote-video selection, Chinese subtitle clipping, or the established Sequoia/YC/Lex/Lenny/Google DeepMind workflow.
---

# Curate AI Interview Clips

Turn long-form AI interviews into an approval-gated topic radar and vertical-video workflow. Treat editorial judgment, quote integrity, and source traceability as more important than filling a quota.

Resolve every bundled path relative to this `SKILL.md`. The command examples below assume the skill directory is the current working directory; keep generated runs in the user's project, not inside the installed skill.

## Workflow

### 1. Prepare the run

Create a dated working directory outside this skill folder:

```bash
RUN_DIR="ai_interview_runs/$(date +%F)"
mkdir -p "$RUN_DIR"
```

Use `scripts/sources.csv` unless the user supplies a different watchlist. Check `state.json` in the workflow root before evaluating candidates. If it does not exist, initialize it with `scripts/state_ledger.py init`.

### 2. Fetch and deduplicate

Run:

```bash
python3 scripts/fetch_latest.py \
  --sources scripts/sources.csv \
  --output-dir "$RUN_DIR" \
  --days 14 \
  --min-score 0 \
  --max-per-channel 15
```

If RSS fails, use `yt-dlp --flat-playlist` or web search to verify each channel's newest uploads. Never treat a failed source as evidence that it has no new videos.

Exclude videos already marked `processed` in `state.json`. A previously `proposed` but unapproved video may return only when new evidence materially improves the angle; identify it as a repeat.

### 3. Produce the topic list only

Read [references/editorial-policy.md](references/editorial-policy.md). Verify claims against the original description, transcript, or another reliable source. Select no more than three topics and report, for each:

- Chinese candidate title
- selection reason
- guest identity and core judgment
- original video link
- recommended excerpt angle and approximate timestamps
- risks or facts still requiring verification

Do not download video, cut clips, translate subtitles, or render output before explicit user approval. Do not lower the threshold to reach three topics. Mark surfaced videos as `proposed` with `scripts/state_ledger.py mark`.

### 4. Download captions after approval

After explicit approval, mark the selected video `approved`, then run captions first:

```bash
python3 scripts/download_and_clip.py \
  --input approved_candidates.json \
  --output-dir "$RUN_DIR/clip_drafts" \
  --limit 3 \
  --min-len 60 \
  --max-len 120 \
  --captions-only
```

Use `scripts/select_viral_clips.py` only as a candidate-window generator. Review the surrounding transcript yourself. Reject ads, sponsor reads, introductions, repeated claims, context-dependent fragments, and low-information passages.

### 5. Refine complete excerpts

Choose boundaries that preserve the setup, reasoning, and conclusion. Prefer 60-120 seconds, but let semantic completeness control the endpoint. A shorter excerpt is acceptable only when the source thought is already complete.

Create a reviewed specs JSON from [references/spec-schemas.md](references/spec-schemas.md), then run:

```bash
python3 scripts/refine_clips.py \
  --specs refined_specs.json \
  --output-dir "$RUN_DIR/refined_clips"
```

### 6. Translate and align subtitles

Read [references/subtitles-and-layout.md](references/subtitles-and-layout.md). Translate the actual spoken words cue by cue. Do not add narration, interpretation, speaker labels, or phrases such as “他说” and “主持人说” unless spoken in the source.

Preserve timing correspondence. Merge or split cues only when needed for natural Chinese reading, without moving meaning to a different sentence.

### 7. Render and verify

Build vertical metadata using [references/spec-schemas.md](references/spec-schemas.md), then run:

```bash
python3 scripts/render_vertical.py \
  --metadata vertical_metadata.json \
  --output-dir "$RUN_DIR/vertical_outputs"
```

Verify every result with `ffprobe` and representative frame screenshots:

- 1080x1920 output
- top and bottom 330px are pure white safe zones
- all editorial content stays inside the centered 1080x1260 region
- source video remains 16:9 and is not stretched
- subtitles are readable, synchronized, and do not overlap metadata
- excerpt begins and ends on complete meaning
- no ads or unsupported paraphrases appear

Mark a video `processed` only after the rendered file passes verification.

## Dependencies

Require Python 3.10+, `ffmpeg`, `ffprobe`, `yt-dlp`, `youtube-transcript-api`, and `imageio-ffmpeg`. Install Python packages from `scripts/requirements.txt` when missing.

## Failure Rules

- Report source failures explicitly and continue checking other sources.
- Do not fabricate timestamps, transcripts, guest identities, or quotes.
- Do not overwrite historical output; always use a dated directory.
- Do not publish or upload finished videos without separate authorization.
- Respect platform terms, copyright, quotation context, and attribution requirements.
