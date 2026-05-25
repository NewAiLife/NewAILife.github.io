---
title: "Backyard Birding with AI: Video Cropping, Keyframes, and the Audio That Got Away"
date: 2026-05-24 22:00:00 +0800
categories: [AI for Nature]
tags: [ffmpeg, BirdNET, Birding, Video Processing, Keyframes, Google Photos, Coolfly]
pin: true
---

## The Idea

I set up a bird feeder camera using the Coolfly/Birding iPhone app, capturing visitors in 2304×1296 video. Nine clips later, I had a simple goal: crop them to a tight 1680×945, extract key frames, and identify every bird that stopped by.

Rather than doing it manually, I handed the project to an AI coding agent — and discovered surprises at every step.

---

## The Setup

| Component | Choice |
|---|---|
| **Camera App** | Coolfly / Birding (iPhone) |
| **Source Resolution** | 2304 × 1296 |
| **Target Resolution** | 1680 × 944 |
| **Video Processing** | ffmpeg 7.1.1 |
| **Bird Sound ID** | BirdNET Analyzer 2.4 |
| **AI Agent** | DeepCode (DeepSeek V4) |
| **Project Docs** | agents.md + skill.md |

Simple enough. But nothing went quite as planned.

---

## Challenge 1: The Vanishing Pixel

The crop math was trivial: 2304 → 1680 centered (x=312), and 1296 → 945 from the bottom (y=351):

```bash
ffmpeg -i original/INPUT.mp4 -vf "crop=1680:945:312:351" -c:a copy cropped/INPUT.mp4
```

The agent ran the first video. Output: **1680×944**, not 945.

Turns out H.264 with yuv420p requires **even-numbered dimensions**. ffmpeg silently rounds 945 → 944. One pixel lost to the encoder gods — visually imperceptible, but a lesson in codec constraints the agent caught and documented immediately.

Eight videos, 20 seconds each, cropped in under a minute.

<video controls muted preload="metadata" style="width:100%; max-width:720px; display:block; margin:1.5em auto;">
  <source src="/assets/img/backyardbirding/C0158E54-FEC7-4D83-9207-9A69BCECCF75.mp4" type="video/mp4">
</video>
*Cropped video (1680×944) — bird feeder action, bottom-center framed.*

---

## Challenge 2: The Audio That Wasn't There

With BirdNET Analyzer installed (Cornell Lab's deep learning bird-call classifier), the agent probed the files:

```
ffprobe → 1 stream: video only. No audio.
```

All nine files. Silent.

The culprit: **Google Photos**. The Coolfly app records audio with a Lavc60 AAC encoder, but uploading to Google Photos strips the audio track entirely during server-side processing. The same thing happened through Pairdrop, the share sheet, and every other export path.

The fix — transfer directly from iPhone via USB cable to bypass iOS remuxing — worked. But by then, the keyframe approach proved more practical anyway.

---

## Challenge 3: BirdNET Without a Location

One file *did* have audio (a direct USB transfer). The agent ran BirdNET:

```
BirdNET-Analyzer → 6,522 species, 22 seconds
```

Without `--lat` and `--lon`, BirdNET scanned every species on Earth. The results were... creative:

| Bird | Confidence | Actually Possible Here? |
|---|---|---|
| Ashy Prinia | 92.9% | Asia |
| Tickell's Leaf Warbler | 70.6% | Asia |
| Grace's Warbler | 85.7% | North America |
| Slaty Bristlefront | 86.7% | Brazil |
| Virginia Rail | 81.2% | Global |

Lesson: BirdNET *needs* location filtering. Without it, you get a global bird lottery.

---

## The Keyframe Approach

The agent pivoted to visual ID. Using ffmpeg's `fps` filter, it extracted one frame every two seconds from each video:

```bash
ffmpeg -i original/VIDEO.mp4 -vf "fps=1/2" -q:v 2 keyframes/VIDEO/frame_%03d.jpg
```

Result: **87 high-quality JPEGs** across 9 videos, organized in subfolders. Each frame ready for upload to Merlin Bird ID or iNaturalist. The agent also produced cropped versions of every frame at 1680×944, matching the video output:

![Keyframe sample](/assets/img/backyardbirding/frame_002_cropped.jpg)
*Cropped keyframe (1680×944) — extracted every 2 seconds, ready for visual bird ID.*

---

## The Final Stack

```text
BackyardBirding/
├── original/         # 8 source videos (2304×1296, 15fps, ~6MB each)
├── cropped/          # 8 cropped videos (1680×944, bottom-center)
├── keyframes/        # 87 frames (1 per 2s, named frame_001–010)
├── audio/            # Extracted WAV for BirdNET analysis
├── agents.md         # Project guide + processing table
└── skill.md          # ffmpeg commands + crop math reference
```

The `agents.md` tracks everything in a table:

| Video | Cropped | Date | Key Frames | Bird |
|---|---|---|---|---|
| 265E33B1... | ✅ | 2026-05-24 | 10 | |
| 37E3DAFE... | ✅ | 2026-05-24 | 7 | |
| ... | | | | |

---

## Key Takeaways

1. **H.264/yuv420p requires even dimensions.** Your crop math may say 945; the encoder says 944. Plan around it.

2. **Google Photos strips non-standard audio.** If your camera app uses Lavc encoders, transfer via USB — not the cloud.

3. **BirdNET needs location context.** A `--lat` / `--lon` flag changes it from a random bird generator to a precision tool.

4. **Keyframes are the universal fallback.** No audio? No problem. One frame every two seconds gives you plenty to work with for visual ID via Merlin or iNaturalist.

5. **AI agents handle the boring parts.** The agent wrote the ffmpeg pipeline, calculated crop offsets, documented the pixel rounding issue, built the tracking table, and generated all 87 keyframes — while I watched birds.

---

## What's Next

The `Bird Identified` column in the tracking table is still empty. The frames are ready at `keyframes/` — each one waiting for a trip through Merlin Bird ID's Photo ID.

If the audio situation improves (direct USB transfer from Coolfly), BirdNET with proper location parameters could add sound-based confirmation.

---

*Tools used: ffmpeg 7.1.1, BirdNET Analyzer 2.4, DeepSeek V4 via DeepCode. Project at [github.com/baifang/BackyardBirding](https://github.com/baifang/BackyardBirding).*
