# openenv-ai-symptoms-triage

## About This Project
An RL environment where an AI agent acts as a medical triage nurse.
The agent reads patient symptoms, age group, duration, and existing conditions, then decides:
- **Severity** — how serious is this? (mild / moderate / severe)
- **Pathway** — where should this patient go? (home care / see a doctor / emergency room / call ambulance)
- **Timeframe** — how urgently? (within 24h / within 4h / immediately)

## Why This Matters
Triage errors in real life cost lives. Undertriaging a severe patient (calling them mild)
is penalized heavily — a 0.3 penalty (-0.3 reward) on top of the scoring loss. The environment is
designed to reward not just accuracy but safety-critical caution.

## Environment Design
- **12 patients** across 3 difficulty tiers
- **Easy** — textbook cases with obvious symptoms
- **Medium** — requires multi-symptom reasoning (e.g. chest tightness alone = severe)
- **Hard** — ambiguous red flag combinations where age and context change everything
  (e.g. elderly + mild confusion + chest pain = cardiac emergency)

## Observation Space
At each step, the agent receives:
- Patient ID and age group
- Presenting symptoms (free text)
- Duration of symptoms
- Existing conditions
- Current step and max steps
- Cumulative reward so far

## Action Space
- **Severity** — mild / moderate / severe
- **Pathway** — home_care / see_doctor / emergency_room / call_ambulance
- **Timeframe** — within_24h / within_4h / immediate

## Scoring
Each decision is scored across three fields:

| Field | Weight |
|-------|--------|
| Severity | 50% |
| Pathway | 30% |
| Timeframe | 20% |

> ⚠️ **Safety penalty:** Undertriaging a severe patient as mild incurs -0.3 on top of the scoring loss — because in real triage, this mistake costs lives.

## Tasks
| Task | Patients | Difficulty | Threshold |
|------|----------|------------|-----------|
| task_easy | p001–p004 | Easy | 0.8 |
| task_medium | p005–p008 | Medium | 0.6 |
| task_hard | p009–p012 | Hard | 0.4 |

## Baseline Agent
Uses `meta-llama/Llama-3.3-70B-Instruct` via HuggingFace's inference router — no local GPU required, runs via API key.

The baseline uses a rule-augmented system prompt encoding clinical triage logic to establish a reproducible performance benchmark.

## Baseline Results
| Task | Score |
|------|-------|
| Easy | 1.0 |
| Medium | 1.0 |
| Hard | 0.93 |
| **Overall** | **0.98** |

## Setup
```bash
uv sync
```

## Run Server
```bash
uvicorn server.app:app --reload --host 0.0.0.0 --port 8000
```

## Run Baseline
```bash
# Windows
$env:HF_TOKEN="your_token_here"
python baseline_inference.py

# Linux/Mac
export HF_TOKEN="your_token_here"
python baseline_inference.py
```

## Validate
```bash
openenv validate
```
