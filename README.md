# MiniMax-H3 Cinematic Continuity Studio

> **Production-grade AI Video Continuity Framework for MiniMax-H3, SatoDive Latent Relays, and Multi-Character Narrative Filmmaking.**

This repository contains the complete end-to-end framework, ComfyUI workflows, character model sheets, timecoded audio, prompt templates, and the **Master LLM System Prompt** used to produce seamless, photorealistic multi-shot film scenes with zero spatial drift and pristine character consistency.

---

## 📁 Repository Structure

```
minimax-h3-cinematic-studio/
├── README.md                                  # This documentation
├── prompts/
│   ├── system_prompt.md                       # Master LLM System Prompt for automated scene generation
│   ├── shot_01_interrogation_entrance.txt     # Shot 1: Master Establishing Entrance & Seating
│   ├── shot_02_interrogation_dialogue.txt     # Shot 2: Close-up & Two-Shot Hindi Dialogue Beat
│   └── prompt_generation_template.json        # Structured input/output schema for LLM workflows
├── workflows/
│   ├── 01_MiniMaxH3_Generation_Workflow.json              # Base generation workflow (Shot 1)
│   ├── 02_MiniMaxH3_Continuation_Workflow.json            # Standard latent continuation & auto-stitch
│   ├── 03_MiniMaxH3_Continuation_Completed.json           # Fully configured production continuation
│   └── 04_MiniMaxH3_Continuation_CloseUp_MasterRef.json   # Dual-seed (Tail + Master Wide Anchor)
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
└── results/
    ├── MiniMaxH3_Stitched_00001_MasterScene.mp4           # 17-second final master stitched continuous scene
    ├── MiniMaxH3_Easy_00001_Shot1.mp4                     # Shot 1 standalone entrance
    └── MiniMaxH3_Easy_00007_DialogueBeat.mp4              # Shot 2 standalone dialogue & reaction beat
```

---

## 🎬 Core Continuity Methodology

### 1. The Dual-Seed Anchor Strategy (Preventing Close-Up Drift)
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

### 2. The Anti-Bleed Rule for New Entering Characters
When a new character enters a scene (e.g., Detective Vance entering an interrogation room where the Guard is already standing):
- **Problem:** If the prompt simply says `preserve character features from @Video1`, the AI will copy the Guard's face onto the Detective because only the Guard existed in Video 1!
- **Solution:** Explicitly split your `Character Continuity` prompt section:
```text
Character Continuity:
- For returning characters (Sato & Davis): strictly preserve facial structure and wardrobe from @Video1.
- For new characters (Detective Vance): strictly take facial identity exclusively from @Image1. 
  He is a NEW character and must NEVER blend or share features with Officer Davis (<Subject 3>).
```

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

Then, simply provide your scene input in this format:

```text
Current Scene Objective:
Detective Vance slams his hands on the table and presents a photograph as evidence. Sato recoils and averts his gaze.

Previous Scene Summary:
Vance questioned Sato about who he was with, and Sato claimed he didn't do anything. Both are seated across the metal table.

Dialogue Script (if any):
[00:00.20 - 00:02.10] Vance: "Then explain what you were doing at the docks!"
[00:02.10 - 00:03.50] (Tense pause)
[00:03.50 - 00:05.20] Sato: "That's not me. You're setting me up."

Shot Framing:
Medium Close-Up on table and hands, cutting to Sato's face (no wide shot present).
```

The LLM will automatically generate the 100% compliant, timecoded, dual-anchor prompt ready to paste directly into ComfyUI!

---

## ⚡ Hardware & Performance Recommendations (NVIDIA L40S / 48GB)

To prevent CUDA OOMs and maintain fast iteration speeds:
1. **PyTorch Memory Config:**
   ```bash
   export PYTORCH_CUDA_ALLOC_CONF="expandable_segments:True"
   ```
2. **VRAM Offload Headroom:** Launch ComfyUI with `--reserve-vram 16.0` to automatically park the 14.6 GB Text Encoder into system RAM during video sampling.
3. **Draft Testing:** Use `steps: 4` on `BasicScheduler` with the **MiniMax H3 Turbo LoRA** and test with `Seconds: 3.5` for sub-90-second feedback loops.
