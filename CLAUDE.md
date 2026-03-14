# lex-cognitive-inertia

**Level 3 Leaf Documentation**
- **Parent**: `/Users/miverso2/rubymine/legion/extensions-agentic/CLAUDE.md`
- **Gem**: `lex-cognitive-inertia`

## Purpose

Models cognitive inertia — the resistance of established beliefs to change. Beliefs are formed with a starting inertia level and can be reinforced (increasing inertia) or challenged (potentially changing them if the challenge force exceeds inertia). Challenge outcomes depend on the belief's current inertia: high-inertia beliefs resist change even against strong challenges. Entrenched beliefs (inertia >= CONVICTION_THRESHOLD) rarely yield. Flexible beliefs (inertia <= FLEXIBILITY_THRESHOLD) change readily. Overall flexibility = 1.0 - average_inertia across all beliefs.

## Gem Info

| Field | Value |
|---|---|
| Gem name | `lex-cognitive-inertia` |
| Version | `0.1.0` |
| Namespace | `Legion::Extensions::CognitiveInertia` |
| Ruby | `>= 3.4` |
| License | MIT |
| GitHub | https://github.com/LegionIO/lex-cognitive-inertia |

## File Structure

```
lib/legion/extensions/cognitive_inertia/
  cognitive_inertia.rb              # Top-level require
  version.rb                        # VERSION = '0.1.0'
  client.rb                         # Client class
  helpers/
    constants.rb                    # Max beliefs, inertia rates, conviction/flexibility thresholds, labels, domains
    belief.rb                       # Belief value object
    inertia_engine.rb               # Engine: beliefs, challenge, reinforcement, domain grouping
  runners/
    cognitive_inertia.rb            # Runner module
```

## Key Constants

| Constant | Value | Meaning |
|---|---|---|
| `MAX_BELIEFS` | 300 | Belief store cap |
| `DEFAULT_INERTIA` | 0.5 | Starting inertia for new beliefs |
| `INERTIA_GROWTH_RATE` | 0.03 | Inertia increase per reinforcement call |
| `CONVICTION_THRESHOLD` | 0.8 | Inertia above this = entrenched/convicted |
| `FLEXIBILITY_THRESHOLD` | 0.3 | Inertia below this = flexible/open |
| `INERTIA_LABELS` | hash | `entrenched` (0.8+) through `fluid` |
| `CONVICTION_LABELS` | hash | Labels for conviction level |
| `BELIEF_DOMAINS` | array | `[:factual, :normative, :identity, :relational, :causal, :aesthetic, :existential]` |

## Helpers

### `Belief`

A held cognitive position with resistance to change.

- `initialize(domain:, content:, inertia: DEFAULT_INERTIA, belief_id: nil)`
- `reinforce!(rate)` — increases inertia by `INERTIA_GROWTH_RATE`, cap 1.0
- `challenge!(force)` — applies challenge of given force; returns outcome (`:changed`, `:resisted`, `:weakened`)
  - `:changed` — force > inertia; belief content could be updated, inertia resets lower
  - `:weakened` — force within range of inertia; inertia decreases slightly
  - `:resisted` — force << inertia; belief unchanged, inertia unchanged
- `entrenched?` — inertia >= `CONVICTION_THRESHOLD`
- `flexible?` — inertia <= `FLEXIBILITY_THRESHOLD`
- `inertia_label`
- `to_h`

### `InertiaEngine`

- `form_belief(domain:, content:, inertia: DEFAULT_INERTIA)` — returns `{ formed:, belief_id:, belief: }` or capacity error
- `challenge_belief(belief_id:, force:)` — returns `{ outcome:, inertia_before:, inertia_after:, belief: }`
- `reinforce_belief(belief_id:)` — increases inertia; returns before/after
- `entrenched_beliefs(limit: 20)` — sorted by inertia descending, filtered to `entrenched?`
- `flexible_beliefs(limit: 20)` — filtered to `flexible?`
- `beliefs_by_domain` — grouped hash
- `most_resistant(limit: 10)` — highest inertia regardless of threshold
- `average_inertia` — mean across all beliefs
- `overall_flexibility` — `1.0 - average_inertia`
- `inertia_report` — full stats

## Runners

**Module**: `Legion::Extensions::CognitiveInertia::Runners::CognitiveInertia`

| Method | Key Args | Returns |
|---|---|---|
| `form_belief` | `domain:`, `content:`, `inertia: DEFAULT_INERTIA` | `{ success:, belief_id:, belief: }` |
| `challenge_belief` | `belief_id:`, `force:` | `{ success:, outcome:, inertia_before:, inertia_after: }` |
| `reinforce_belief` | `belief_id:` | `{ success:, before:, after: }` |
| `entrenched_beliefs` | `limit: 20` | `{ success:, beliefs: }` |
| `flexible_beliefs` | `limit: 20` | `{ success:, beliefs: }` |
| `average_inertia` | — | `{ success:, inertia:, label: }` |
| `overall_flexibility` | — | `{ success:, flexibility: }` |
| `inertia_report` | — | Full report hash |

Private: `inertia_engine` — memoized `InertiaEngine`. Logs via `log_debug` helper.

## Integration Points

- **`lex-cognitive-flexibility`**: `lex-cognitive-flexibility` tracks task-set switching cost and flexibility score. `lex-cognitive-inertia` tracks per-belief resistance. Both model resistance to change — flexibility at the behavioral level, inertia at the belief level. High average inertia correlates with low flexibility.
- **`lex-cognitive-gravity`**: Entrenched beliefs in inertia correspond to high-mass attractors in gravity. Both model how dominant cognitive patterns resist displacement.
- **`lex-memory`**: Highly reinforced beliefs (high inertia) correlate with high-strength memory traces. Challenging a belief and producing a `:changed` outcome could trigger memory trace modification in lex-memory.

## Development Notes

- `challenge_belief` outcomes are probabilistic relative to force vs. inertia. A force exactly equal to inertia produces `:weakened`. Force significantly greater than inertia produces `:changed`. Force significantly less produces `:resisted`. The exact thresholds are implemented in the Belief class.
- Inertia growth via `reinforce_belief` is incremental (`INERTIA_GROWTH_RATE = 0.03`). It takes many reinforcements to drive a belief from `DEFAULT_INERTIA` (0.5) to `CONVICTION_THRESHOLD` (0.8): approximately 10 reinforcement calls.
- In-memory only.

---

**Maintained By**: Matthew Iverson (@Esity)
