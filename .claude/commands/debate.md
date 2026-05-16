---
description: Run a bear/bull/mediator debate on a given ticker or asset
---

Run a structured investment debate on: **$ARGUMENTS**

If $ARGUMENTS is empty, ask the user which ticker or asset to debate and stop. Otherwise, you are the orchestrator. Execute the following sequence, capturing each agent's full output so you can pass it forward.

**Step 0 (Dossier — required first step):** Invoke the `analyst` subagent with the prompt: "Build the fact dossier for $ARGUMENTS per your role spec. Produce sections §1–§13 as defined. Cite inline. Target 3,000–5,000 words. Do not advocate a direction." Capture the full output as **DOSSIER**.

Pass DOSSIER into every subsequent agent prompt as a clearly labeled section, like:

```
=== DOSSIER ===
<full dossier text>
=== END DOSSIER ===
```

**Round 1 (horizon-split debate):** Invoke the `bear` and `bull` subagents **four times in parallel** in a single message (4 tool calls). Each prompt must include the full DOSSIER section.
- `bear` (near-term): "Argue the bear case for $ARGUMENTS on a **6-month horizon**. Focus on tape, recent earnings, near-term catalysts, technicals, positioning. Cite the dossier as your primary evidence. This is round 1 — no prior arguments to rebut." Capture as BEAR_NEAR_R1.
- `bear` (long-term): "Argue the bear case for $ARGUMENTS on a **2-3 year horizon**. Focus on structural/secular risks, competitive position, terminal economics, base-rate cyclicality. Cite the dossier as your primary evidence and include an explicit DCF sensitivity. This is round 1 — no prior arguments to rebut." Capture as BEAR_LONG_R1.
- `bull` (near-term): "Argue the bull case for $ARGUMENTS on a **6-month horizon**. Focus on tape, recent earnings, near-term catalysts, technicals, positioning. Cite the dossier as your primary evidence. This is round 1 — no prior arguments to rebut." Capture as BULL_NEAR_R1.
- `bull` (long-term): "Argue the bull case for $ARGUMENTS on a **2-3 year horizon**. Focus on structural/secular drivers, competitive moats, terminal economics. Cite the dossier as your primary evidence and include an explicit DCF sensitivity. This is round 1 — no prior arguments to rebut." Capture as BULL_LONG_R1.

Then invoke the `mediator` with the DOSSIER plus all four debater outputs clearly labeled by side AND horizon. Capture as MEDIATOR_R1. The mediator must emit BOTH a near-term and a long-term verdict per its role spec, plus an Overall Action.

**Red-team step:** After MEDIATOR_R1, invoke the `challenger` subagent with the DOSSIER, the full MEDIATOR_R1 verdict, and all four R1 debater outputs. Capture as CHALLENGER_R1. Then invoke `mediator` again with the DOSSIER, original verdict, and challenger critique, asking it to either revise or defend. Capture as MEDIATOR_R1_REVISED. Use the revised verdict for the final summary.

**Decision:** Look at the `Continue debate?` line of MEDIATOR_R1_REVISED. If it starts with NO, stop and go to the summary. If YES, proceed to round 2.

**Round 2 (optional):** Invoke all four debaters in parallel. Each prompt must include (a) the full DOSSIER, (b) the *opposing* side's R1 output for the same horizon in full, (c) MEDIATOR_R1_REVISED in full, and (d) instructions to rebut the strongest opposing points and address the mediator's critique. Capture as BEAR_NEAR_R2, BEAR_LONG_R2, BULL_NEAR_R2, BULL_LONG_R2. Then invoke `mediator` with DOSSIER + everything labeled. Capture as MEDIATOR_R2. Optionally re-run the red-team step.

**Decision:** Same parse on MEDIATOR_R2. If NO, stop. If YES, run round 3.

**Round 3 (optional):** Same pattern as round 2 with R2 inputs feeding the next round. Capture as BEAR_NEAR_R3, BEAR_LONG_R3, BULL_NEAR_R3, BULL_LONG_R3, then MEDIATOR_R3 with DOSSIER and all prior context.

**Hard cap:** Stop after round 3 regardless.

**Output order (strict):**

1. First, show the **DOSSIER** in full at the top of your response (clearly labeled).
2. Then show the full transcript of each round (BEAR_NEAR_Rn, BEAR_LONG_Rn, BULL_NEAR_Rn, BULL_LONG_Rn, MEDIATOR_Rn, CHALLENGER_Rn, MEDIATOR_Rn_REVISED — all clearly labeled).
3. At the very bottom, show the **Final summary** so it is the last thing the user sees:

- **Asset**: $ARGUMENTS
- **Overall Action**: BUY / SELL / HOLD (from final mediator — surface this prominently as the first line of the summary)
- **Near-term action (~6mo)**: BUY / SELL / HOLD
- **Long-term action (2-3yr)**: BUY / SELL / HOLD
- **Near-term verdict**: Bullish / Bearish / Neutral
- **Long-term verdict**: Bullish / Bearish / Neutral
- **Confidence**: (from final mediator)
- **Rounds run**: 1, 2, or 3
- **Did the challenger move the verdict?**: YES (what changed) / NO (held firm)
- **Single most important factor**: (from final mediator)
- **Key unresolved questions** (if any)
