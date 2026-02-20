---
title: "The Agent Negotiation Protocol"
parent: Algorithm Reference
nav_order: 9
---

## Section 9: The Agent Negotiation Protocol

Before any match reaches a user's screen, their agents must negotiate and agree.

### 9.1 Eligibility Gate

Only pairs with `final_score ≥ 0.80` after penalty application proceed to negotiation. This ensures only genuinely high-compatibility pairs incur the Gemini API cost for transcript generation.

### 9.2 The Debate

Two AI agents, each initialised with their respective user's `agent_dossier`, enter a simulated negotiation using Gemini 2.0 Flash. They evaluate:

- Shared vibe keywords and aesthetic resonance
- Mutual `attracted_to` archetypes (does Agent A's target match Agent B's archetype, and vice versa?)
- Potential dealbreaker conflicts
- Astrological compatibility vibe alignment

The negotiation explores friction points honestly — a good transcript surfaces tension, not just flattery.

### 9.3 The Human Icebreaker

The negotiation output is stored as `matches.negotiation_transcript`. When the match is surfaced to both users, the transcript is injected directly into their new chat room as the opening message. This serves as the **human icebreaker** — a highly personalised, agent-authored conversation starter that gives both users something specific and interesting to react to immediately.

> This is the single most differentiating UX feature of Alter: the moment of connection isn't a blank chat box, it's a window into how your agents saw each other.

