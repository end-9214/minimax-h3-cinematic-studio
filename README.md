# MiniMax-H3 Cinematic Continuity Studio

> **Production-grade AI Video Continuity Framework for MiniMax-H3, SatoDive Latent Relays, and Multi-Character Narrative Filmmaking.**

This repository contains the complete end-to-end framework, ComfyUI workflows, character model sheets, timecoded audio, prompt templates, and the **Master LLM System Prompt** used to produce seamless, photorealistic multi-shot film scenes with zero spatial drift and pristine character consistency.

---

## 📁 Repository Structure

```
minimax-h3-cinematic-studio/
├── README.md                                  # Complete architecture & continuity guide
├── prompts/
│   ├── system_prompt.md                       # Master LLM System Prompt for automated scene generation
│   ├── shot_01_interrogation_entrance.txt     # Shot 1: Master Establishing Entrance & Seating
│   ├── shot_02_interrogation_dialogue.txt     # Shot 2: Close-up & Two-Shot Hindi Dialogue Beat
│   └── prompt_generation_template.json        # Structured input/output schema for LLM workflows
├── workflows/
│   ├── 01_MiniMaxH3_Generation_Workflow.json              # Base generation workflow (Shot 1)
│   ├── 02_MiniMaxH3_Continuation_Workflow.json            # Standard latent continuation & auto-stitch
│   ├── 03_MiniMaxH3_Continuation_Completed.json           # Fully configured production continuation
│   └── 04_MiniMaxH3_Continuation_CloseUp_MasterRef.json   # Dual-Seed (Tail + Master Wide Anchor)
├── assets/
│   ├── characters/
│   │   ├── sato_suspect_character_sheet.jpg               # Subject 2: Sato (Plaid shirt suspect)
│   │   ├── officer_davis_guard_character_sheet.png        # Subject 3: Officer Davis (Standing guard)
│   │   ├── detective_vance_statesman_character_sheet.jpg  # Subject 1: Detective Vance (Interrogator)
│   │   └── indian_officer_inspector_turnaround.jpg        # Alternate Inspector Character Turnaround
│   ├── environment/
│   │   └── interrogation_room_master_plate.jpg            # Subject 4: Concrete room master environment
│   ├── audio/
│   │   └── elevenlabs_hindi_interrogation_dialogue.wav    # Studio ElevenLabs 5.94s Hindi dialogue track
│   └── latents/
│       └── sato_scene-001_00001.h3latent.safetensors      # Lossless AV latent from Shot 1
├── results/
│   ├── MiniMaxH3_Stitched_00001_MasterScene.mp4           # 17-second final master stitched continuous scene
│   ├── MiniMaxH3_Easy_00001_Shot1.mp4                     # Shot 1 standalone entrance
│   └── MiniMaxH3_Easy_00007_DialogueBeat.mp4              # Shot 2 standalone dialogue beat
└── examples/
    └── microdrama_tiger_monkey/                           # 3D Clay Animation Comedy Microdrama
        ├── character.md                                   # 16:9 3-view turnaround specs (Sheru & Cheeku)
        ├── script.md                                      # 4-part comedy script (every part ends on cliffhanger)
        ├── characters/                                    # 16:9 turnaround character sheets
        ├── environment/                                   # 16:9 jungle clearing plate
        ├── audio/                                         # Audio scripts with emotional direction tags
        └── prompts/                                       # Fully calibrated Shot 1 & Shot 2 prompts
```

---

## 🌟 THE GOLDEN RULES OF MINIMAX-H3 CONTINUITY

### 1. The "Slot 1 Priority Law" for New Entering Characters 🚨
In MiniMax H3, reference image slots are **NOT equal**. As logged by the engine:
> `[MiniMax H3 Easy] Reference image order (slot 1 is read first and asserts hardest): <Picture 1> = Ref Image 1 (strength 1.00)`

- **The Problem:** If you introduce a **NEW character** into Shot 2 or Shot 3, and you put them in Slot 2, 3, or 4, the returning anchor characters from `@Video1` will overpower their features, causing the AI to copy or bleed the returning character's face onto the new character!
- **The Golden Rule:** **Whenever a NEW character enters the scene, you MUST place their reference image in Slot 1 (`@Image1` / `<Subject 1>`)!**
  - **Slot 1 (`@Image1` / `<Subject 1>`):** The NEW entering character (asserts hardest).
  - **Slot 2 (`@Image2` / `<Subject 2>`):** The returning character from `@Video1`.
  - **Slot 3 (`@Image3` / `<Subject 3>`):** The environment location plate.

---

### 2. The Dual-Seed Anchor Strategy (Preventing Close-Up Drift)
When filming multi-shot scenes in generative video, a major challenge is **spatial drift**: if Shot 2 is composed entirely of close-ups and over-the-shoulder cuts, the AI model loses track of the overall room layout and door position.

To solve this, our framework introduces **Dual-Seed Video Routing**:

| Parameter | Input Source | Function |
| :--- | :--- | :--- |
| **`seed_video`** | **Previous Shot Video (Tail)** | Provides the immediate temporal frame handoff, preserving character poses and motion vectors at the exact cut seam. |
| **`seed_ref_video`** | **Master Wide Establishing Shot** | Feeds `<Video 1>` into the diffusion cross-attention, locking 100% of the room walls, table position, ceiling lamp, and door physics. |

Every shot prompt is explicitly tagged in its header:
```text
# Shot Type Metadata:
#   shot_type: MEDIUM_TWO_SHOT_AND_CLOSE_UP
#   wide_shot_present: false
#   seed_video_role: SHOT_01_TAIL
#   seed_ref_video_role: SHOT_01_WIDE_MASTER
#   is_master_spatial_anchor: false
```
- If **`wide_shot_present: true`**: This shot is the **Master Spatial Anchor**.
- If **`wide_shot_present: false`**: The workflow connects the earlier Wide Shot into **`seed_ref_video`** so close-ups never hallucinate background alterations!

---

### 3. Sub-Second Audio & Dialogue Synchronization
MiniMax H3 is a multimodal Audio-Video model. To achieve believable lip synchronization:
1. **Timecoded Action Beats:** Use `[continuation shot]: At MM:SS.mmm` matching the audio waveform.
2. **Explicit Mouth Enforcements:**
   - Define exact start/end articulation times for the speaking character.
   - **Crucial:** Explicitly order the non-speaking actor to keep their mouth shut (`<Subject 2> keeps his mouth tightly closed, forming no speech shapes`).
3. **Breathing Room:** Ensure a natural **1.0s – 1.7s pause** between different characters speaking on the same audio track so the AI cleanly switches facial focus.

---

## 🤖 How to Use the LLM System Prompt

Open [`prompts/system_prompt.md`](file:///home/ubuntu/minimax-h3-cinematic-studio/prompts/system_prompt.md) and paste it into **ChatGPT (GPT-4o)**, **Claude 3.5 Sonnet**, or **Gemini 1.5 Pro**.

The system prompt automatically enforces:
- Slot 1 priority for new characters.
- Dual-seed video routing (`seed_video` + `seed_ref_video`).
- Timecoded dialogue lip-sync blocks.

---

## ⚡ Fast Video Upscaling (480p to 1080p in 6 Seconds)
To keep generation fast, render your video drafts in **480p** (~90 seconds per test). Once satisfied with the acting and camera moves:
- Feed the decoded frames into **`Upscale Image (using Model)`** with **`4x-UltraSharp.pth`**.
- It upscales the entire video to razor-sharp 1080p/4K in **~5 to 8 seconds** using your GPU's tensor cores without re-running any diffusion!
