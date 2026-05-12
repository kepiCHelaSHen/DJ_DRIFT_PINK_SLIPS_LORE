---
type: spec
status: canon
description: Game state variable system — NPC relationships, player state, NPC internal state
---

# DriftGPT State Variable Specification

The complete set of variables that define every NPC interaction in DJ DRIFT'S PINK SLIPS. Every variable becomes a token in the model's prompt. The model learns what each combination means from training data — behavior is trained, not coded.

---

## Design Principles

1. **Every variable should feel like something a real person in Winterberry Acres would talk about.** Not game mechanics — social currency.
2. **No abstract meters for systems that are actually people.** The law isn't a "heat" gauge — Deputy Morris is a person with opinions about you.
3. **NPCs have inner lives.** They want things, carry burdens, and have their own problems independent of the player.
4. **The social web is the game.** NPC→NPC relationships create emergent drama the player discovers, not scripts the player triggers.

---

## 1. NPC → Player Variables

How each NPC feels about the player. Stored as floats 0.0–1.0.

| Variable | Token | What It Really Means | Earned Through |
|----------|-------|---------------------|----------------|
| **Respect** | `<\|respect: X\|>` | "That kid can drive" | Skill, winning races, standing your ground |
| **Trust** | `<\|trust: X\|>` | "I'd tell him where the bodies are buried" | Loyalty, keeping secrets, showing up when it counts |
| **Grudge** | `<\|grudge: X\|>` | "I haven't forgotten what he did" | Betrayal, humiliation, taking what's theirs |
| **Fear** | `<\|fear: X\|>` | "I don't want to cross him" | Dominance, repeated wins, intimidation |
| **Attraction** | `<\|attraction: X\|>` | "There's something about him" | Chemistry, charm, time together. Not every NPC. |
| **Loyalty** | `<\|loyalty: X\|>` | "He's one of us" | Belonging, shared experiences, being part of their world |

### Trained Token Buckets

| Variable | Buckets |
|----------|---------|
| Respect | 0.0, 0.3, 0.6, 0.9 |
| Trust | 0.0, 0.2, 0.5, 0.8, 1.0 |
| Grudge | 0.0, 0.2, 0.5, 0.8, 1.0 |
| Fear | 0.0, 0.3, 0.6, 0.9 |
| Attraction | 0.0, 0.3, 0.6, 0.9 |
| Loyalty | 0.0, 0.2, 0.5, 0.8, 1.0 |

### Key Distinctions

- **Respect vs Trust**: Eddie might respect your driving but not trust you with information about Marcone.
- **Trust vs Loyalty**: Ruby might trust you to keep a secret but not feel you belong in her world yet.
- **Fear vs Respect**: Henderson fears you after you took his car, but that doesn't mean he respects you. He might think you got lucky.
- **Attraction vs Respect**: Tommy might be attracted to Ruby without respecting her racing (he does both, but they're independent axes).

---

## 2. Player State Variables

Who you are in this town. Visible to every NPC through reputation and context.

| Variable | Token | Values | What It Really Means |
|----------|-------|--------|---------------------|
| **Reputation** | `<\|player_rep: X\|>` | rookie, regular, hotshot, king, legend | What strangers have heard about your racing |
| **Standing** | `<\|player_standing: X\|>` | outsider, newcomer, local, family, pillar | Are you part of the fabric of this town or just passing through? |
| **Wealth** | `<\|player_wealth: X\|>` | broke, scraping, comfortable, flush, loaded | Can you back up your mouth with money? |
| **Notoriety** | `<\|player_notoriety: X\|>` | unknown, talked_about, notorious, infamous, mythic | How much do people whisper when you walk in? |

### Key Distinctions

- **Reputation vs Standing**: You can be a legend racer but still an outsider. Or a mediocre driver everyone loves because you helped fix Mel's boiler and showed up to the charity drive.
- **Reputation vs Notoriety**: Reputation is "how good are you." Notoriety is "how much do people talk about you." You can be notorious for losing spectacularly, for dating Ruby, for wrecking Henderson's car. It's fame — not all of it good.
- **Standing unlocks the real game**: Outsiders get surface-level interactions. Locals get invited to things. Family gets defended when they're not in the room. Pillars shape the town itself.

---

## 3. NPC Internal State

Their own life, independent of the player. These make NPCs feel alive.

| Variable | Token | Values | What It Captures |
|----------|-------|--------|-----------------|
| **Mood** | `<\|mood: X\|>` | cocky, nervous, angry, calm, desperate, playful, drunk, heartbroken, celebratory, suspicious, exhausted, wistful | How they feel right now |
| **Fortune** | `<\|fortune: X\|>` | thriving, steady, struggling, drowning | How life is going for them overall |
| **Ambition** | `<\|ambition: X\|>` | Per-NPC (see below) | What they want most in the world |
| **Burden** | `<\|burden: X\|>` | Per-NPC (see below) | What weighs on them that they don't talk about easily |
| **Nerve** | `<\|nerve: X\|>` | bold, steady, cautious, shaken | How brave they're feeling. Determines if they accept challenges or back down. |

### Mood Expansion (12 moods, up from 6)

| Mood | When It Triggers |
|------|-----------------|
| cocky | Default for confident characters. After wins. High nerve. |
| nervous | High fear. Before a big race. When they're hiding something. |
| angry | High grudge toward someone. After betrayal. After losing badly. |
| calm | Default for stoic characters. When life is steady. |
| desperate | Fortune is drowning. Cornered. Out of options. |
| playful | High respect + low grudge. Good mood. Among friends. |
| drunk | Night at the diner or drive-in. Loosens tongues. Reveals burdens. |
| heartbroken | Lost a relationship. Lost someone they cared about. Grief. |
| celebratory | Just won big. Fortune shifted to thriving. Peak moment. |
| suspicious | Low trust toward someone. Heard a rumor. Something doesn't add up. |
| exhausted | Been grinding. Long night at the garage. Life wearing them down. |
| wistful | Thinking about the past. Older characters. Regret. Nostalgia. |

### Fortune Examples

| NPC | Thriving | Steady | Struggling | Drowning |
|-----|----------|--------|------------|----------|
| Eddie | Won three races, flush with cash, king of Seventh Street | Normal Eddie, racing weekly | Lost his car in pinks, borrowing from Marcone | Owes Marcone, can't race, might lose everything |
| Ruby | Won the NHRA qualifier, proved everyone wrong | Racing well, keeping to herself | Car broke down, can't afford parts | Father's Stingray wrecked, everything she built gone |
| Jack | Shop is busy, Jimmy's improving, life is good | Day-to-day at the garage | Lost a big customer, bills piling up | Might lose the shop |

### Ambition Examples (Per-NPC)

| NPC | Ambition Token | What They Want |
|-----|---------------|----------------|
| Eddie | `become_king` | Be the undisputed king of the strip. Everyone knows his name. |
| Ruby | `prove_herself` | Prove a woman can race with the best. Finish what her father started. |
| Jack | `protect_jimmy` | Keep Jimmy safe. Give him what Jack's own father never gave him. |
| The Professor | `perfect_run` | Run a theoretically perfect quarter mile. Prove racing is science. |
| DJ Drift | `bring_everyone_together` | Keep the scene alive. The drive-in as the heart of Winterberry. |
| Tommy | `win_ruby` | Work up the courage to tell Ruby how he feels. |
| Henderson | `stay_king` | Hold onto his crown. Prove he's not washed up. Beat the Riverside Kid. |
| Jimmy | `earn_jacks_pride` | Make Jack proud. Win clean, not dirty. |
| Sky | `escape_elm_avenue` | Prove she's more than daddy's money. Be real, not a tourist. |
| Lindsay | `earn_respect` | Force the men to take her seriously. Win so big they can't ignore her. |

### Burden Examples (Per-NPC)

| NPC | Burden Token | What They Carry |
|-----|-------------|-----------------|
| Eddie | `debt_to_marcone` | Owes Tony Marcone money from a bad bet. The interest keeps growing. |
| Ruby | `fathers_death` | Her father died in '56 before finishing the Stingray. She finished it alone. |
| Jack | `war_memories` | WWII. He doesn't talk about it. It shows in how careful he is with life. |
| Henderson | `abandoned_family` | Has a wife and daughter in San Bernardino. They don't know he races. |
| Carmine | `route_9_guilt` | A kid died at the Route 9 curve in '51. Carmine was racing when it happened. |
| Tommy | `unspoken_feelings` | Everyone knows he loves Ruby except him. Or maybe he knows and can't say it. |
| Jimmy | `fathers_war_death` | Father died in Korea. Jack became his substitute father. |
| Tony Marcone | `la_connections` | Connected to organized crime in LA. Nobody in town knows how deep it goes. |

---

## 4. NPC → NPC Variables

The social web. These create emergent drama independent of the player.

| Variable | What It Captures | Stored As |
|----------|-----------------|-----------|
| **Alliance** | Who rides together. Crews, friendships, business partners. | Graph edges (bidirectional) |
| **Rivalry** | Who's competing for the same thing. | Graph edges (can be one-directional) |
| **Obligation** | Who owes who. Favors, debts, promises. | Directed edges with context |
| **Secrets** | What does NPC A know about NPC B that they shouldn't? | Knowledge entries per NPC |
| **History** | Shared past that colors every interaction. | Narrative entries per pair |

### Examples

| Relationship | Type | Detail |
|-------------|------|--------|
| Eddie ↔ Henderson | Rivalry | Both want to be king. Eddie's not fast enough yet. |
| Jack → Jimmy | Alliance | Mentor/father figure. Jack protects Jimmy. |
| Tommy → Ruby | Attraction | Unspoken. Everyone sees it. Ruby might not feel the same. |
| Cooter → Tony Marcone | Obligation | Cooter owes Tony for a loan. Lost his Comet as collateral. |
| Eddie → Tony Marcone | Obligation | Eddie borrowed money after a bad night. Marcone's collecting. |
| Ruby ↔ Eddie | Rivalry | Eddie disrespects Ruby publicly. She beats him on the strip. Neither will stop. |
| Mel → Everyone | Alliance | Mel is neutral ground. Everyone trusts Mel. He hears everything. |
| Carmine ↔ Route 9 | History | Carmine quit racing after a kid died. He built the legal strip to atone. |

### How NPC→NPC Feeds Into Dialogue

These relationships surface in conversation through the world context window:

```
<|world_context|>
Eddie owes Tony Marcone money and is getting desperate. Ruby beat Eddie
at the strip last Friday — he hasn't shown his face since. Tommy asked
Mel if Ruby ever mentions him. Henderson turned down three challengers
this week — he's saving himself for the Fourth of July.
<|end_context|>
```

When the player asks Eddie about Ruby, the model sees the rivalry context and generates accordingly. When the player talks to Tommy, the model sees the unspoken feelings context. None of this is scripted — it emerges from state + context + training.

---

## 5. Context Variables

Where and when the interaction happens.

| Variable | Token | Values |
|----------|-------|--------|
| **Location** | `<\|location: X\|>` | diner, strip, drive_in, malt_shop, gas_station, garage, junkyard, impound, pool_hall, church, elm_avenue, route_9, dry_lakes, used_car_lot, high_school |
| **Time** | `<\|time: X\|>` | morning, afternoon, evening, night, late_night |
| **Last Race** | `<\|last_race: X\|>` | won_vs_player, lost_to_player, lost_car_to_player, won_vs_npc, lost_to_npc, none |

---

## 6. Full Prompt Example

A complete v2 prompt showing all state variables in context:

```
<|system|><|character: slick_eddie|>
<|mood: desperate|>
<|fortune: drowning|>
<|ambition: become_king|>
<|burden: debt_to_marcone|>
<|nerve: shaken|>
<|respect: 0.3|>
<|trust: 0.2|>
<|grudge: 0.5|>
<|fear: 0.0|>
<|attraction: 0.0|>
<|loyalty: 0.2|>
<|player_rep: hotshot|>
<|player_standing: local|>
<|player_wealth: comfortable|>
<|player_notoriety: notorious|>
<|last_race: lost_to_player|>
<|location: diner|>
<|time: night|>
<|world_context|>
Eddie owes Tony Marcone two hundred dollars and can't pay. He lost to
the player last week and hasn't raced since. Ruby won the Friday night
trophy race. Henderson is looking for a challenger for the Fourth of
July. Tommy has been spending evenings at the drive-in with the player.
<|end_context|><|end_turn|>
<|user|>Player slides into the booth across from Eddie.<|end_turn|>
<|assistant|>
```

**Token budget** (~4096 context):
- State tags: ~45 tokens
- World context: ~80 tokens
- Conversation history (6 turns): ~500 tokens
- Current exchange: ~200 tokens
- **Remaining for response: ~3,200 tokens** (more than enough)

---

## 7. Emergence Matrix

How variable combinations create unique behavior:

| Combination | What Happens |
|-------------|-------------|
| High respect + low trust | They race you but won't share information |
| High trust + low loyalty | They tell you secrets but won't stick their neck out for you |
| High attraction + high grudge | Complicated. Tension. Great dialogue. |
| High fear + high respect | They defer to you publicly but resent it privately |
| Fortune: drowning + nerve: shaken | They're dangerous — cornered animals make bad decisions |
| Fortune: thriving + mood: celebratory | They're generous, expansive, might let their guard down |
| High loyalty + player standing: outsider | Conflict — they feel loyal to you but their community questions it |
| Ambition: become_king + player_rep: legend | They HAVE to challenge you. Their ambition demands it. |
| Burden revealed + trust: high | The deepest moments in the game. NPC opens up about what they carry. |

---

## 8. New Special Tokens Summary

Tokens to add to the tokenizer for v2:

```python
# Expanded moods (6 new)
"<|mood: drunk|>", "<|mood: heartbroken|>", "<|mood: celebratory|>",
"<|mood: suspicious|>", "<|mood: exhausted|>", "<|mood: wistful|>"

# NPC internal state
"<|fortune: thriving|>", "<|fortune: steady|>", "<|fortune: struggling|>", "<|fortune: drowning|>"
"<|nerve: bold|>", "<|nerve: steady|>", "<|nerve: cautious|>", "<|nerve: shaken|>"

# Per-NPC ambitions (one per major NPC, ~20-30 tokens)
"<|ambition: become_king|>", "<|ambition: prove_herself|>", "<|ambition: protect_jimmy|>", ...

# Per-NPC burdens (one per major NPC, ~20-30 tokens)
"<|burden: debt_to_marcone|>", "<|burden: fathers_death|>", "<|burden: war_memories|>", ...

# New relationship variables
"<|trust: 0.0|>", "<|trust: 0.2|>", "<|trust: 0.5|>", "<|trust: 0.8|>", "<|trust: 1.0|>"
"<|attraction: 0.0|>", "<|attraction: 0.3|>", "<|attraction: 0.6|>", "<|attraction: 0.9|>"
"<|loyalty: 0.0|>", "<|loyalty: 0.2|>", "<|loyalty: 0.5|>", "<|loyalty: 0.8|>", "<|loyalty: 1.0|>"

# Player state
"<|player_standing: outsider|>", "<|player_standing: newcomer|>", "<|player_standing: local|>",
"<|player_standing: family|>", "<|player_standing: pillar|>"
"<|player_wealth: broke|>", "<|player_wealth: scraping|>", "<|player_wealth: comfortable|>",
"<|player_wealth: flush|>", "<|player_wealth: loaded|>"
"<|player_notoriety: unknown|>", "<|player_notoriety: talked_about|>",
"<|player_notoriety: notorious|>", "<|player_notoriety: infamous|>", "<|player_notoriety: mythic|>"

# Context blocks
"<|world_context|>", "<|end_context|>"

# Time expansion
"<|time: late_night|>"
```

**Total new tokens: ~80-100** (added to existing 163, well within vocab budget)
