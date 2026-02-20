---
title: Welcome
layout: default
nav_order: 0
---

# Alter: The Open Matchmaking Algorithm

> **Agents over Swiping**. Your AI agent learns your taste, scouts the ecosystem 24/7, negotiates with other agents, and only brings you matches worth your attention.

## What's Inside

This documentation is a complete technical reference for Alter's AI matchmaking architecture. Every algorithm, every model, every weight, every formula — all explained.

### Core Concepts

**🧠 The Agent**
Instead of you swiping through profiles, an autonomous AI agent works on your behalf. It learns from every interaction, builds constraints based on feedback, and gets smarter each day.

**📊 The Data**
Alter captures three signal types during onboarding:
- **10 Photos** → Visual aesthetic analysis
- **Audio Interview** → Personality & life context
- **Birth Details** → Vedic astrological mapping

**⚙️ The Match**
Matching runs nightly at 11:11 PM UTC. Your agent's embedding is compared against all others using cosine similarity + Vedic astrology scoring. High-scoring pairs trigger an agent negotiation before you ever see them.

**💬 The Icebreaker**
When a match lands, the negotiation transcript between the two agents appears in your chat. You have an immediate, highly personalized conversation starter.

### Quick Navigation

- **[User Journey & Data Taxonomy](algorithm/01-user-journey.md)** - How data flows through the system
- **[The AI Stack](algorithm/02-ai-stack.md)** - Models, Edge Functions, orchestration
- **[Pipeline Stages](algorithm/03-audio-processing.md)** - Audio → Visual → Embedding
- **[The Mathematics](algorithm/06-mathematics-of-matching.md)** - 65% vector + 35% Vedic
- **[Vedic Scoring](algorithm/07-vedic-compatibility.md)** - Guna Milan, Nakshatra, Kuta
- **[Constraint Penalties](algorithm/08-constraint-penalty.md)** - Learning from rejection
- **[Agent Negotiation](algorithm/09-negotiation-protocol.md)** - The icebreaker magic
- **[RL Loop](algorithm/10-evolving-intelligence.md)** - How the system learns
- **[Scouting Feed](algorithm/11-scouting-feed.md)** - Phase 12 live agent updates
- **[Vibe Check](algorithm/12-vibe-check.md)** - Safety layer for conversations
- **[Contributing](algorithm/15-contributing.md)** - RFC process for algorithm changes

### For Researchers & Engineers

This is an **open, auditable algorithm**. We welcome contributions:
- Embedding model improvements
- Vedic scoring refinements
- Bias audits
- Performance optimizations
- Cross-cultural compatibility work

See **[Contributing to the Algorithm](algorithm/15-contributing.md)** for the RFC process.

---

**Last Updated**: Phase 12 (Scouting Feed)
**Status**: Open source, auditable, community-driven
[GitHub](https://github.com/adidshaft/alter) · [Chat](https://alter.dating)
