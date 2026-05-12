---
description: Run a bear/bull/mediator debate on a given ticker or asset
---

Run a structured investment debate on: **$ARGUMENTS**

If $ARGUMENTS is empty, ask the user which ticker or asset to debate and stop. Otherwise, you are the orchestrator. Execute the following sequence, capturing each agent's full output so you can pass it forward.

**Round 1 (horizon-split debate):** Invoke the `bear` and `bull` subagents **four times in parallel** in a single message (4 tool calls):
- `bear` (near-term): "Argue the bear case for $ARGUMENTS on a **6-month horizon**. Focus on tape, recent earnings, near-term catalysts, technicals, positioning. This is round 1 — no prior arguments to rebut." Capture as BEAR_NEAR_R1.
- `bear` (long-term): "Argue the bear case for $ARGUMENTS on a **2-3 year horizon**. Focus on structural/secular risks, competitive position, terminal economics, base-rate cyclicality. This is round 1 — no prior arguments to rebut." Capture as BEAR_LONG_R1.
- `bull` (near-term): "Argue the bull case for $ARGUMENTS on a **6-month horizon**. Focus on tape, recent earnings, near-term catalysts, technicals, positioning. This is round 1 — no prior arguments to rebut." Capture as BULL_NEAR_R1.
- `bull` (long-term): "Argue the bull case for $ARGUMENTS on a **2-3 year horizon**. Focus on structural/secular drivers, competitive moats, terminal economics. This is round 1 — no prior arguments to rebut." Capture as BULL_LONG_R1.

Then invoke the `mediator` with all four outputs clearly labeled by side AND horizon. Capture as MEDIATOR_R1. The mediator must emit BOTH a near-term and a long-term verdict per its role spec, plus an Overall Action.

**Red-team step:** After MEDIATOR_R1, invoke the `challenger` subagent with the full MEDIATOR_R1 verdict plus all four R1 debater outputs. Capture as CHALLENGER_R1. Then invoke `mediator` again with the original verdict and the challenger critique, asking it to either revise or defend. Capture as MEDIATOR_R1_REVISED. Use the revised verdict for the final summary.

**Decision:** Look at the `Continue debate?` line of MEDIATOR_R1_REVISED. If it starts with NO, stop and go to the summary. If YES, proceed to round 2.

**Round 2 (optional):** Invoke all four debaters in parallel. Each prompt must include (a) the *opposing* side's R1 output for the same horizon in full, (b) MEDIATOR_R1_REVISED in full, and (c) instructions to rebut the strongest opposing points and address the mediator's critique. Capture as BEAR_NEAR_R2, BEAR_LONG_R2, BULL_NEAR_R2, BULL_LONG_R2. Then invoke `mediator` with everything labeled. Capture as MEDIATOR_R2. Optionally re-run the red-team step.

**Decision:** Same parse on MEDIATOR_R2. If NO, stop. If YES, run round 3.

**Round 3 (optional):** Same pattern as round 2 with R2 inputs feeding the next round. Capture as BEAR_NEAR_R3, BEAR_LONG_R3, BULL_NEAR_R3, BULL_LONG_R3, then MEDIATOR_R3 with all prior context.

**Hard cap:** Stop after round 3 regardless.

**Output order (strict):** First show the full transcript of each round (BEAR_NEAR_Rn, BEAR_LONG_Rn, BULL_NEAR_Rn, BULL_LONG_Rn, MEDIATOR_Rn, CHALLENGER_Rn, MEDIATOR_Rn_REVISED — all clearly labeled). Then, at the very bottom of your response, show the **Final summary** so it is the last thing the user sees:

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
