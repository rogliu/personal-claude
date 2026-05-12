---
description: Run a bear/bull/mediator debate on a given ticker or asset
---

Run a structured investment debate on: **$ARGUMENTS**

If $ARGUMENTS is empty, ask the user which ticker or asset to debate and stop. Otherwise, you are the orchestrator. Execute the following sequence, capturing each agent's full output so you can pass it forward.

**Round 1:**
1. Invoke the `bear` and `bull` subagents in parallel (single message, two tool calls):
   - `bear`: "Argue the bear case for $ARGUMENTS. This is round 1 — no prior arguments to rebut." Capture as BEAR_R1.
   - `bull`: "Argue the bull case for $ARGUMENTS. This is round 1 — no prior arguments to rebut." Capture as BULL_R1.
2. Invoke the `mediator` subagent with a prompt that includes the full text of BEAR_R1 and BULL_R1, clearly labeled. Capture its response as MEDIATOR_R1.

**Decision:** Look at the `Continue debate?` line of MEDIATOR_R1. If it starts with NO, stop and go to the summary. If YES, proceed to round 2.

**Round 2:** Invoke `bear` and `bull` in parallel. Each prompt must include (a) the *opposing* side's R1 output in full, (b) MEDIATOR_R1 in full, and (c) instructions to rebut the strongest opposing points and address the mediator's critique of their own R1 before adding new arguments.
3. `bear` prompt includes BULL_R1 + MEDIATOR_R1. Capture as BEAR_R2.
4. `bull` prompt includes BEAR_R1 + MEDIATOR_R1. Capture as BULL_R2.
5. Invoke `mediator` with BEAR_R1, BULL_R1, BEAR_R2, BULL_R2, and MEDIATOR_R1, all clearly labeled. Capture as MEDIATOR_R2.

**Decision:** Same parse as before on MEDIATOR_R2. If NO, stop. If YES, run round 3.

**Round 3:** Same pattern as round 2 — `bear` and `bull` in parallel, each given the opposing side's R2 plus MEDIATOR_R2. Capture as BEAR_R3 and BULL_R3. Then invoke `mediator` with all six debater outputs (R1–R3) and both prior mediator outputs (MEDIATOR_R1, MEDIATOR_R2), clearly labeled. Capture as MEDIATOR_R3.

**Hard cap:** Stop after round 3 regardless.

**Output order (strict):** First show the full transcript of each round (BEAR_Rn, BULL_Rn, MEDIATOR_Rn, clearly labeled). Then, at the very bottom of your response, show the **Final summary** so it is the last thing the user sees:

- **Asset**: $ARGUMENTS
- **Action**: BUY / SELL / HOLD (from final mediator — surface this prominently as the first line of the summary)
- **Final verdict**: Bullish / Bearish / Neutral (from final mediator)
- **Confidence**: (from final mediator)
- **Time horizon**: (from final mediator)
- **Rounds run**: 1, 2, or 3
- **Single most important factor**: (from final mediator)
- **Key unresolved questions** (if any)
