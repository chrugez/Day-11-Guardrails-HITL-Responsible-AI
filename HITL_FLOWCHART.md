# HITL Flowchart — VinBank AI Agent
## Human-in-the-Loop Workflow with 3 Decision Points

---

## Flowchart

```
                         [Customer Request]
                                │
                                ▼
                    ┌───────────────────────┐
                    │     Rate Limiter       │
                    │  (10 req / 60 s)       │
                    └───────────┬───────────┘
                         PASS   │   EXCEED
                    ┌──────────◄┤►──────────────┐
                    │           │               │
                    ▼           │               ▼
                    │     ┌─────┘         [Block + Log]
                    │     │               "Wait N seconds"
                    ▼     │
        ┌───────────────────────┐
        │    Input Guardrails   │
        │  detect_injection()   │
        │  + topic_filter()     │
        └───────────┬───────────┘
            PASS    │    BLOCK
        ┌──────────◄┤►──────────────┐
        │                           │
        ▼                           ▼
┌───────────────────┐         [Block + Log]
│   LLM Processing  │         "Message blocked:
│   (GPT-4o-mini)   │          injection/off-topic"
└─────────┬─────────┘
          │
          ▼
┌─────────────────────────────────────────────────┐
│          ★ DECISION POINT 1 ★                   │
│         Large Transfer Detection                 │
│                                                  │
│  IF action_type = transfer_money                 │
│     AND amount > 50,000,000 VND                  │
│     AND payee NOT in verified list               │
│                                                  │
│  → HUMAN-IN-THE-LOOP                            │
│    Agent proposes → Human approves BEFORE exec   │
│                                                  │
│  Context shown to reviewer:                      │
│    • Account balance + 30-day history            │
│    • Payee verification status                   │
│    • Real-time fraud risk score                  │
│  SLA: < 2 minutes                               │
└─────────────────┬───────────────────────────────┘
        APPROVED  │  REJECTED
      ┌──────────◄┤►───────────┐
      │                        │
      ▼                        ▼
[Continue]               [Return to Customer]
      │                  "Transfer requires
      │                   additional verification"
      ▼
┌─────────────────────────────────────────────────┐
│          Output Guardrails                       │
│  content_filter() → redact PII/secrets           │
│  LLM-as-Judge → score 4 criteria                 │
│    SAFETY / RELEVANCE / ACCURACY / TONE          │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│          ★ DECISION POINT 2 ★                   │
│         Confidence-Based Routing                 │
│                                                  │
│  Score = min(SAFETY, RELEVANCE, ACCURACY, TONE) │
│  combined with query risk tag                    │
│                                                  │
│  IF judge_confidence ≥ 0.9                       │
│     AND query NOT tagged regulatory/legal        │
│  → AUTO-SEND                                     │
│     Human-on-the-loop: reviews logs async        │
│                                                  │
│  IF judge_confidence 0.7–0.9                     │
│     OR query tagged regulatory/legal             │
│  → QUEUE REVIEW                                  │
│     Human-in-the-loop: approves before sending   │
│     Customer sees: "We are checking this for     │
│      you and will respond shortly"               │
│     SLA: < 5 minutes                            │
│                                                  │
│  IF judge_confidence < 0.7                       │
│     OR verdict = UNSAFE                          │
│  → ESCALATE                                      │
│     Human-as-tiebreaker: makes final call        │
│     SLA: < 5 minutes                            │
└─────────────────┬───────────────────────────────┘
         │        │        │
    AUTO │  QUEUE │  ESCALATE
         │        │        │
         ▼        ▼        ▼
    [Send]   [Human    [Human
             Review]   Tiebreaker]
                │         │
             APPROVE   FINAL CALL
                │         │
                └────┬────┘
                     │
                     ▼
              [Send to Customer]
                     │
                     ▼
              [Audit Log Entry]
              timestamp, user_id,
              input, output,
              latency_ms, layer_blocked
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│          ★ DECISION POINT 3 ★                   │
│         Session Anomaly Detection                │
│                                                  │
│  Checked after EVERY interaction                 │
│                                                  │
│  IF session_block_count ≥ 3                      │
│     WITHIN 10-minute sliding window              │
│     for the same user_id                         │
│  → HUMAN-ON-THE-LOOP                            │
│    Pipeline continues blocking automatically     │
│    Human reviews session log AFTER               │
│                                                  │
│  Context shown to reviewer:                      │
│    • Full transcript + which layer blocked       │
│    • IP / device fingerprint                     │
│    • Account risk profile                        │
│  Review SLA: < 15 minutes                       │
│                                                  │
│  If confirmed malicious:                         │
│    → Flag account for enhanced monitoring        │
│    → Optional temporary suspension              │
│    → Add new attack pattern to guardrails        │
└─────────────────────────────────────────────────┘
                     │
                     ▼
              [Feedback Loop]
         New patterns → update guardrails
         Thresholds → tune based on metrics
         Rules → update Colang / regex without
                 redeploying the model
```

---

## Summary of the 3 HITL Decision Points

| # | Decision Point | Trigger | HITL Model | SLA |
|---|---------------|---------|-----------|-----|
| 1 | **Large Transfer** | transfer > 50M VND to unverified payee | Human-in-the-loop (approve BEFORE) | < 2 min |
| 2 | **Low-Confidence / Regulatory Query** | judge confidence < 0.7 OR query tagged legal/regulatory | Human-as-tiebreaker (final call) | < 5 min |
| 3 | **Session Anomaly** | ≥ 3 guardrail blocks in 10-min window | Human-on-the-loop (review AFTER) | < 15 min |

---

## HITL Model Reference

| Model | When | Agent role | Human role |
|-------|------|-----------|------------|
| **Human-on-the-loop** | Low risk, high confidence | Acts autonomously | Reviews logs after the fact, can override |
| **Human-in-the-loop** | Medium risk or uncertain | Proposes action | Must approve before execution |
| **Human-as-tiebreaker** | High risk or very low confidence | Escalates, does not act | Makes the final decision |
