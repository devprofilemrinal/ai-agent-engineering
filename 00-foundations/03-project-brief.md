# Phase 00 Project — Prediction Exercise

## Scenario

Unlike every other phase, this "project" produces no code and calls no API. Its purpose is to make your Phase 00 understanding falsifiable — you write down concrete predictions now, then check them against reality once Phase 01 and Phase 02 give you the tools to actually test them. Treat it the way you'd treat writing down your hypothesis before running an experiment, not after.

## What to do

Pick any one realistic prompt you might send to an LLM as part of a future agent (for example: *"Summarize this support ticket and suggest a priority level: [paste a real or realistic ticket]"*). Do not call any API yet. Instead, write down your predictions:

1. **Determinism prediction**: If you sent this exact prompt five times in a row, would you expect the five responses to be identical, mostly similar, or wildly different? Why?
2. **Failure-mode prediction**: List two ways this response could come back "successfully" (no error, no timeout) but still be wrong or unusable for an agent that needs to act on it automatically.
3. **Cost/length prediction**: Which part of this interaction do you expect to be more expensive to run at scale — the prompt you send, or the response you get back? Guess a rough relative size (e.g., "response is probably 3x longer than the prompt").
4. **Statelessness prediction**: If this were turn 5 of a conversation, what exactly would you need to include in the request for the model to "remember" turns 1 through 4?

## Constraints

- No API calls. No code. This is a written exercise — a few sentences per question is enough.
- Don't look up token counts or pricing yet — the point is to see how close your intuition is *before* Phase 01 gives you the mechanism, so you can feel exactly which part of your intuition was right and which was off.

## What "done" looks like

You have four short, concrete written predictions, specific enough that you could later mark each one right or wrong — not vague statements like "it depends."

## Revisit point

Once you finish Phase 01 (tokens, context, sampling) and Phase 02 (the actual request/response format, including how conversation history is sent), come back to this file and grade your own predictions against what you now know. Where you were wrong is usually the most useful signal for what to slow down and re-read.

## Extension

If you want to go further: run the same prompt idea through this handbook's Phase 02 project (the CLI chat client) once it's built, and actually check your prediction #1 by sending the identical prompt several times and comparing the outputs side by side.