# Professor: a model that speaks and draws

Senior thesis research at Princeton (in progress). This is the public overview; the working repository stays private because the main training corpus is derived from YouTube videos and has no license.

## The question

A good teacher at a whiteboard talks and draws at the same time. The strokes on the board follow what they're saying, and they revise both when a student interrupts. Current systems mostly fake this: an LLM writes a say/draw script, and a player animates it.

This project asks whether **one model can produce speech and pen strokes together**, in real time, with a board it can keep referring to and revise. The strongest version of the claim is about *coordination with persistent, editable graphics*, not "first model to draw while talking". Related work on concurrent speech and actions and on voice–canvas systems (ELLSA, DyaPlex, VoCa, Sketchpad Live, SketchAgent, RUBATO) is mapped against this goal in the private repo's prior-work notes.

## Starting point

A prior flat word+pen transformer trained on a chalk-talk corpus gave a useful negative result. Board context helps predict the next word, but carrying pen tokens in the same stream costs more than it gains, and the model can't form characters. That motivates the approach here, though it doesn't prove that serialization is the only cause.

The corpus: 1,191 math lecture videos, about 1.78M spoken words (~209 h) and 1.66M pen strokes, with strokes attached to words by time overlap.

## Hypotheses

Each is tested on its own, with a stated failure condition.

1. **Representation.** Parallel speech and pen streams (Moshi-style, one frame every 80 ms) reduce the interference of a single interleaved sequence: language quality holds while drawings become recognizable and correct.
2. **Learning.** Verified synthetic *construction and revision* trajectories teach a drawing policy more than supervision on finished boards alone.
3. **Interaction.** A learned policy that conditions on incoming speech and the actual canvas state beats strong asynchronous tool-calling baselines on the quality-vs-responsiveness tradeoff, especially for spoken corrections and revisions.

## Approach

- **Base model:** start from an existing multi-stream speech model (NVIDIA NemotronLabs VoiceChat 11B), whose per-frame function-call channel is the closest existing precedent for a pen channel. Then add a dedicated pen stream with its own embedding and head.
- **Benchmark (v0, built):** 130 prompts (100 "teach" items from held-out lectures, 30 "draw" items from Quick, Draw!), a renderer, rule-based timing metrics (speech/pen overlap, sync lag), a VLM judge, and baselines: a script pipeline and VoiceChat zero-shot. Human rating is the primary evaluation, because the VLM judge saturates on any clean board.
- **Dot-canvas testbed:** a minimal version of the problem. A Qwen3-VL policy talks while moving one dot on a 32×32 canvas, seeing the canvas after each move. It's used to compare action interfaces (tool calls, action tokens, a separate action expert, a small MLP head) and to probe what the policy actually reads from the image.

## Early findings

- VoiceChat zero-shot never draws. It talks about drawing, and its function channel stays empty on all 40 tested items.
- On the dot canvas, fine-tuning lifts success on random tasks from ~15% to 67–74%. The choice of action interface barely changes accuracy but changes cost a lot (about 31 tokens per move for tool calls vs about 8 for action tokens).
- Held-out shapes fail for every interface (0–5%). Given the reference speech, the action expert reaches ~23%, so most of the gap is in *planning* the unseen shape.
- Image probes show the fine-tuned policy uses the canvas for its position (a blank canvas costs 15–21 points) but draws shapes from its own move history. Zero-shot, small VLMs locate a single dot's cell ≤13% of the time; larger models reach 70–89% only with a labelled grid.

These are development results, not final numbers.

## Status

| Stage | What | Status |
|---|---|---|
| 0 | data, models, environments, smoke tests | done |
| 1 | benchmark v0 and baselines | done (human review next) |
| 2 | fine-tune VoiceChat with drawing commands in its function channel | not started |
| 3 | dedicated pen stream | not started |
| side | dot-canvas testbed | v1 done |
