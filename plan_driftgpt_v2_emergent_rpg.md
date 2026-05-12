---
type: plan
status: active
description: Technical roadmap for DriftGPT v2 — QLoRA fine-tune on pretrained 3B model with retrieval + state conditioning
---

# DriftGPT v2 — Emergent RPG Dialogue System

## Architecture Shift

v1 was trained from scratch (691M params). v2 starts from a pretrained 3B model that already knows English and conversation — we only teach it Winterberry Acres behavior.

| | v1 (proof of concept) | v2 (production) |
|---|---|---|
| Base | Trained from scratch (691M) | Pretrained open model (3B) |
| What it learns | English + dialogue + characters | Character voice, social rules, state reasoning |
| Context | 512 tokens | 4096-8192 |
| Training method | Full fine-tune from random init | QLoRA (4-bit quantized LoRA adapters) |
| Training time | Days (pretrain) + hours (fine-tune) | Hours (fine-tune only) |
| Training cost | ~$70 | ~$5-10 |
| Inference speed | ~8ms/token (Q4 CPU) | ~15ms/token (Q4 CPU) — still under 200ms per response |
| Quality ceiling | Limited by 691M capacity | Much higher — 3B with pretrained knowledge |

## Core Design Principle

**Do not make the model memorize everything. Make the model learn the world's behavior.**

- **Retrieval** supplies canonical facts (lore repo, event log, NPC knowledge graph)
- **State engine** supplies current relationship/social variables
- **Fine-tuning** teaches tone, character logic, social rules, emotional consequence
- **Model** generates dialogue consistent with all three

```
Corpus → SLM fine-tune teaches style/behavior
Lore repo / vector DB → supplies canonical facts
State engine → supplies current relationship/social variables
SLM → generates dialogue/action consistent with all three
```

## Base Model Selection

**Primary: Qwen2.5-3B-Instruct**
- 3B params, 32K native context (we use 4096-8192)
- Strong instruction following and conversation
- Apache 2.0 license — can ship with the game
- Quantizes well to Q4 (~1.8GB)
- Good at structured input (state tags)

**Backup: Phi-3.5-mini-instruct (3.8B)**
- Microsoft, MIT license
- Slightly larger but similar quality

**Prototype: StableLM-2-1.6B**
- Smaller, faster, for rapid iteration
- Good fallback for lower-spec hardware

## Model Size Ladder

| Size | Use Case | Inference (Q4 CPU) | Quality |
|------|----------|-------------------|---------|
| 1.5B | Prototype, testing | ~5ms/token | Good voice, basic reasoning |
| 3B | **Production target** | ~15ms/token | Strong voice, good social reasoning |
| 7B | Premium/server | ~30ms/token | Excellent but borderline for real-time |
| 13B+ | Offline generation only | Too slow | Overkill for bounded world |

**Rule: a well-trained 3B with retrieval beats a generic 7B for a bounded, socially dense world like Winterberry Acres.**

## Training Data Format

### Interaction Training (the gold)

Not stories. Interactions. Each example is a structured state-conditioned NPC response:

```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are Lindsay Smith (Red Line Lindsay), 22. Works at Valley Auto Supply. Knows every part number, every racer's build, every unpaid invoice. Sharp, witty, immovable about credit. Handles the boys' club with humor, not silence.\n\n<|character: red_line_lindsay|>\n<|mood: calm|>\n<|fortune: steady|>\n<|nerve: bold|>\n<|respect: 0.3|>\n<|trust: 0.0|>\n<|grudge: 0.0|>\n<|fear: 0.0|>\n<|player_rep: rookie|>\n<|player_standing: newcomer|>\n<|location: gas_station|>\n<|time: afternoon|>"
    },
    {
      "role": "user",
      "content": "Player walks into Valley Auto Supply and asks for credit on an expensive race part."
    },
    {
      "role": "assistant",
      "content": "Lindsay doesn't look up from the invoice she's writing. \"Cash or check.\" She flips a page. \"We don't do credit for people I haven't seen before.\" Finally she glances up, studies the player for exactly two seconds. \"What are you building?\""
    }
  ]
}
```

### Multi-Turn with World Context

```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are Henderson Cole, king of the Winterberry Acres strip. 35. Cold, measured, doesn't waste words. Drives a black '51 Comet Drifter. Carries guilt about the past but never speaks about it. Respects competence and nothing else.\n\n<|character: henderson|>\n<|mood: calm|>\n<|fortune: steady|>\n<|respect: 0.6|>\n<|trust: 0.0|>\n<|player_rep: hotshot|>\n<|player_standing: local|>\n<|location: diner|>\n<|time: evening|>\n\n<|world_context|>\nPlayer beat Santos last Friday by two car lengths. Henderson watched. Eddie owes Tony Marcone money. The Fourth of July race is next week.\n<|end_context|>"
    },
    {"role": "user", "content": "Player sits down across from Henderson at his booth."},
    {"role": "assistant", "content": "Henderson doesn't look up from his coffee. A long silence. Then: \"Saw your run Friday.\" He takes a sip. \"Sloppy launch. Good top end.\""},
    {"role": "user", "content": "Player asks about the Fourth of July race."},
    {"role": "assistant", "content": "Now he looks up. Eyes flat. Measuring. \"What about it.\" Not a question. A wall."},
    {"role": "user", "content": "Player asks if Henderson is worried about the Riverside Kid."},
    {"role": "assistant", "content": "The slightest tightening around his jaw. \"I don't worry about people.\" He sets the coffee down carefully. \"I worry about cars.\" Then he stands, drops exact change on the table, and walks out without another word."}
  ]
}
```

### Training Data Categories

| Category | Count | What It Teaches |
|----------|-------|-----------------|
| Character interactions (1 NPC + player) | 1,000 | Voice, personality, state responsiveness |
| State-conditioned variations (same NPC, different states) | 500 | How mood/trust/fear changes dialogue |
| Cross-character gossip (NPC A about NPC B) | 300 | Social web, information flow |
| Multi-turn conversations (4-8 exchanges) | 400 | Coherence, deepening trust |
| World event reactions (NPCs react to same event) | 200 | Consistent world, different perspectives |
| NPC-NPC ambient conversations | 200 | Overheard dialogue, ambient world |
| Refusals / deflections / lies | 200 | Low trust behavior, boundaries |
| High-trust reveals (burden, secrets) | 200 | The deep moments, emotional payoff |
| **Total** | **~3,000** | |

### Character Description Rules

System prompt is a behavioral summary, NOT the full lore:

- **Eddie**: "Cocky greaser, 22. Uses 'pal' and 'daddy-o.' Brags constantly. Gets louder when scared. Never admits fault. Insecure underneath the swagger."
- **Ruby**: "Quiet, terse, 19. Uses as few words as possible. Lets actions speak. Precise when she does talk."
- **Jack**: "Stoic WWII vet, 47. Slow, deliberate. Dry humor. Simple words. Calls people 'son.' Never raises his voice."

Lore details (Johnny Vale's car, Vincent Moreno's clipping) come through the world_context block from retrieval, NOT baked into the model.

## Runtime Architecture

```
Player Action
    ↓
Game Engine (Godot)
    ├── State Engine: NPC relationships, mood derivation
    ├── Event Log: recent events, race results
    ├── Knowledge Graph: what this NPC knows
    ├── Context Builder: assembles relevant facts (~200 tokens)
    ↓
Prompt Assembly
    ├── System: character description + state tags + world context
    ├── Conversation history (last 4-6 exchanges)
    ├── Current user input
    ↓
DriftGPT v2 (3B Q4, llama.cpp)
    ↓
Response
    ↓
State Engine Update
    ├── Relationship changes
    ├── Event logging
    ├── Gossip propagation
```

## Training Pipeline

### Step 1: Generate Training Data (~4-6 hours)
Use the Training Data Generator agent to create ~3,000 examples from:
- 31 canon NPC profiles (character voice + state variations)
- 27 stories (event reactions, cross-character gossip)
- State variable spec (mood/trust/fear response patterns)

### Step 2: QLoRA Fine-Tune (~2-4 hours on A100)
```bash
python finetune_qlora.py \
  --base-model Qwen/Qwen2.5-3B-Instruct \
  --dataset data/v2_training.json \
  --lora-rank 64 \
  --lora-alpha 128 \
  --epochs 3 \
  --batch-size 4 \
  --lr 2e-4 \
  --context-length 4096 \
  --output driftgpt_v2_lora
```

### Step 3: Merge + Quantize
```bash
python merge_lora.py --base Qwen2.5-3B-Instruct --lora driftgpt_v2_lora --output driftgpt_v2_merged
llama-quantize driftgpt_v2_merged.gguf driftgpt_v2_Q4_K_M.gguf Q4_K_M
```

### Step 4: Test in Winterberry Nights

## Cost

| Step | Time | Cost |
|------|------|------|
| Generate training data | 4-6 hours | Free (local agents) |
| Download base model | 30 min | Free |
| QLoRA fine-tune | 2-4 hours | ~$3-5 (Vast.ai A100) |
| Quantize + test | 1 hour | Free (local) |
| **Total** | **~8-12 hours** | **~$3-5** |

Vast.ai balance: ~$97. Enough for dozens of iterations.

## Game Engine Memory System (unchanged from v1 plan)

See previous sections on Event Log, NPC Knowledge Graph, Context Builder, and Conversation Manager. These are game engine components (Godot/GDScript) that feed the model's context window at runtime. The model doesn't memorize facts — the game engine retrieves them.

## What This Enables

- Sub-200ms NPC responses on consumer hardware
- Every NPC sounds distinct based on fine-tuned character voice
- State tags change behavior (cocky Eddie vs desperate Eddie)
- World context makes NPCs reference specific events
- Multi-turn conversations that build and deepen
- No internet required — model ships with the game
- Cheap to iterate — regenerate data, retrain in hours, test immediately
