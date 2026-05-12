---
description: Run a bear/bull/mediator debate on a given ticker or asset
---

Run a structured investment debate on: **$ARGUMENTS**

You are the orchestrator. Execute the following sequence, capturing each agent's full output so you can pass it forward.

**Round 1:**
1. Invoke the `bear` subagent with the prompt: "Argue the bear case for $ARGUMENTS. This is round 1 — no prior arguments to rebut." Capture its full response as BEAR_R1.
2. Invoke the `bull` subagent with the prompt: "Argue the bull case for $ARGUMENTS. This is round 1 — no prior arguments to rebut." Capture its full response as BULL_R1.
3. Invoke the `mediator` subagent with a prompt that includes the full text of BEAR_R1 and BULL_R1, clearly labeled. Capture its response as MEDIATOR_R1.

**Decision:** If MEDIATOR_R1 says "Continue debate? NO", stop and go to the summary. If YES, proceed to round 2.

**Round 2:**
4. Invoke `bear` with a prompt that includes BULL_R1 in full and instructs it to rebut the strongest points before adding new arguments. Capture as BEAR_R2.
5. Invoke `bull` with a prompt that includes BEAR_R1 in full and instructs it to rebut the strongest points before adding new arguments. Capture as BULL_R2.
6. Invoke `mediator` with both round 1 and round 2 outputs from bear and bull, plus MEDIATOR_R1, all clearly labeled. Capture as MEDIATOR_R2.

**Decision:** If MEDIATOR_R2 says "Continue debate? NO", stop. If YES, run round 3 using the same pattern (each side rebuts the most recent opposing round, mediator sees all prior rounds).

**Hard cap:** Stop after round 3 regardless.

**Final summary** (present to the user):
- **Asset**: $ARGUMENTS
- **Final verdict**: (from final mediator)
- **Confidence**: (from final mediator)
- **Time horizon**: (from final mediator)
- **Rounds run**: 1, 2, or 3
- **Single most important factor**: (from final mediator)
- **Key unresolved questions** (if any)

Show the full transcript of each round below the summary so the user can read the underlying arguments.
