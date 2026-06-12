# Integrity Gate Pipeline Checkpoints

**Category:** pipeline
**Confidence:** production-proven
**Observed in:** [Imbad0202/academic-research-skills](https://github.com/Imbad0202/academic-research-skills)
**Compatible with:** python, typescript, any, claude-code

## Problem
AI agent outputs contain hallucinated citations, fabricated references, or unsupported claims that pass through to users without verification, eroding trust in agent-generated documents

## The Logic
Insert mandatory blocking verification stages at fixed points in a multi-step pipeline. Each gate runs a machine-enforced checklist (e.g. 7-mode failure taxonomy: hallucinated refs, unsupported claims, fabricated methodology, constraint violations) BEFORE surfacing output to user. Gate either PREVENTs pipeline advancement on FAIL (with max retry limit e.g. 3) or presents a structured PASS/FAIL report requiring explicit human acknowledgement before proceeding. Pattern: stage_output -> integrity_checker(output, checklist_modes) -> {PASS: advance, FAIL: retry or block} -> human_ack -> next_stage. Two gate classes: (1) automated-then-acknowledge gates at mid-pipeline, (2) zero-tolerance deep-check gates at final output. Observer/advisory agents are explicitly SKIPPED during integrity gates to prevent dilution of compliance signal.

## Steal This When
- agent pipeline generates legal documents, contracts, or citations that could harm users if hallucinated (Draft Terminal clause generation)
- multi-step LLM pipeline where downstream stages amplify errors from upstream stages
- user trust in AI output is a core product requirement
- you need an audit trail proving output integrity for compliance or professional contexts
- LanceDB retrieval results feed into LLM generation and claim-source fidelity must be verified

## Gotchas
- Gate fatigue: too many human-acknowledgement checkpoints kills UX — reserve mandatory ack only for high-stakes outputs, make others advisory
- Retry loops without a max count can create infinite pipeline stalls — always set a hard retry ceiling (e.g. 3) before escalating to human
- Observer/advisory agents must be explicitly excluded from integrity gates or their non-blocking nature dilutes the blocking signal
- Claim-level auditing (fetching cited source and verifying support) is expensive — implement as opt-in flag not default
- FNR/FPR calibration against a gold set is needed before trusting automated gates — ship a reference gold set and acceptance thresholds

## Real Implementation
https://github.com/Imbad0202/academic-research-skills
