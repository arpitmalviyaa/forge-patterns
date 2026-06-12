# Channel-Agnostic Skill Router

**Category:** agent
**Confidence:** production-proven
**Observed in:** [openclaw/openclaw](https://github.com/openclaw/openclaw)
**Compatible with:** python, typescript, any

## Problem
Agent logic gets tangled with delivery channel specifics — you write separate handlers for Slack, Telegram, WhatsApp and the business logic duplicates or diverges across each, making skills hard to test and reuse

## The Logic
Decouple skill execution from channel delivery via a gateway control plane. Incoming messages from any channel are normalized into a canonical envelope {channel, user, text, attachments, context}. A router maps intent/command to a skill handler that knows nothing about the originating channel. The skill returns a canonical response object. A channel adapter layer then serializes the response for the target channel. Skills register themselves with metadata {name, triggers, permissions} at startup. The gateway owns routing policy; skills own workflow logic. Pattern: normalize_input(raw_msg) -> canonical_envelope -> route_to_skill(envelope) -> skill.execute(envelope) -> canonical_response -> channel_adapter.send(response, channel_id)

## Steal This When
- you are building Baby agent and want to surface it via Slack AND a web UI without duplicating logic
- Draft Terminal needs to push clause suggestions to multiple surfaces (web, email, Slack notification) from one skill
- Greenlit needs the same analysis skill callable from a Next.js UI and a Telegram bot
- any project where one AI capability must be reachable from more than one channel or frontend

## Gotchas
- Channel-specific features (reactions, threads, rich cards) leak back into skills if the canonical response schema is too thin — define a rich enough envelope upfront with optional channel_hints field
- Session and auth context must travel with the canonical envelope or skills silently operate without user permissions
- Skill registration at startup creates hidden ordering dependencies — skills that depend on other skills must declare it explicitly or routing races occur
- Normalizing wildly different input types (voice, image, text) into one envelope shape requires versioning the schema early or you break older skills silently

## Real Implementation
https://github.com/openclaw/openclaw
