# lex-cognitive-inertia

Belief resistance and cognitive inertia model for brain-modeled agentic AI in the LegionIO ecosystem.

## What It Does

Models the resistance of established beliefs to change. Beliefs are formed across seven domains (factual, normative, identity, relational, causal, aesthetic, existential) with a starting inertia level. Reinforcing a belief increases its inertia; challenging it with a force value produces one of three outcomes: changed (force exceeds inertia), weakened (force partially overcomes inertia), or resisted (belief holds). Entrenched beliefs (inertia >= 0.8) rarely yield. Overall flexibility is computed as `1.0 - average_inertia` across all beliefs.

## Usage

```ruby
require 'legion/extensions/cognitive_inertia'

client = Legion::Extensions::CognitiveInertia::Client.new

# Form a new belief
result = client.form_belief(domain: :causal, content: 'strict typing prevents most bugs', inertia: 0.4)
belief_id = result[:belief_id]

# Reinforce it — increases inertia
client.reinforce_belief(belief_id: belief_id)
# => { success: true, before: 0.4, after: 0.43 }

# Challenge with moderate force
client.challenge_belief(belief_id: belief_id, force: 0.3)
# => { success: true, outcome: :weakened, inertia_before: 0.43, inertia_after: 0.38 }

# Find entrenched beliefs
client.entrenched_beliefs(limit: 5)
# => { success: true, beliefs: [...] }

# Check overall cognitive flexibility
client.overall_flexibility
# => { success: true, flexibility: 0.62 }
```

## Development

```bash
bundle install
bundle exec rspec
bundle exec rubocop
```

## License

MIT
