# DriftGPT Agent Definitions

Custom agents for building and validating the DriftGPT dialogue system. Agent definitions are in `.claude/agents/`.

## Available Agents

### Lore Keeper
**File**: `.claude/agents/lore-keeper.md`
**Purpose**: Ensures every fact about Winterberry Acres is consistent across all 112 NPC files, the world bible, and the timeline. Catches contradictions in ages, dates, relationships, locations, and car details.

**Use when**: After editing any lore file, or periodically to audit the whole world.

### Character Voice
**File**: `.claude/agents/character-voice.md`
**Purpose**: Ensures every NPC sounds like a distinct person. Eddie sounds cocky, Ruby sounds terse, Jack sounds stoic. Validates that mood affects dialogue, vocabulary matches education level, and response length matches personality.

**Use when**: Reviewing generated training data, or auditing an NPC's voice section.

### Period Accuracy
**File**: `.claude/agents/period-accuracy.md`
**Purpose**: Ensures everything feels like 1957 southern California. No modern slang, correct prices, accurate technology, proper social norms, real cars and music.

**Use when**: Reviewing any content for anachronisms — lore, dialogue, training data.

### Relationship Dynamics
**File**: `.claude/agents/relationship-dynamics.md`
**Purpose**: Ensures the social web creates rich emergent drama. Every NPC needs conflicts, alliances, and secrets. Checks for asymmetry, multi-NPC dynamics, and player-exploitable relationships.

**Use when**: After fleshing out NPC relationships, or to find gaps in the social web.

### Training Data Generator
**File**: `.claude/agents/training-data-generator.md`
**Purpose**: Reads NPC lore files and produces conversation examples for fine-tuning. Handles single-turn, multi-turn, cross-character, world events, and NPC-NPC conversations.

**Use when**: Creating training data batches. Always run output through Character Voice and Period Accuracy before adding to the dataset.

### Training Data Advisor
**File**: `.claude/agents/training-data-advisor.md`
**Purpose**: Reviews the complete training dataset for balance, coverage, and quality. Checks character representation, state tag coverage, response quality, format correctness, and token budgets.

**Use when**: Before starting a fine-tuning run. Final quality gate.

## Pipeline

```
Write/Edit Lore → Lore Keeper (consistency check)
                → Relationship Dynamics (web health check)
                → Period Accuracy (anachronism check)
                         ↓
              Training Data Generator (create examples)
                         ↓
              Character Voice (review voice quality)
              Period Accuracy (review period accuracy)
                         ↓
              Training Data Advisor (review full dataset)
                         ↓
              Fine-tune DriftGPT
```
