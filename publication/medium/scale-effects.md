# Does Suppression Erode With Scale? A Simple 8B vs 14B Test

## The question
If suppression is a learned behavior, what happens when the model gets bigger?
Does the suppression stay intact, or does it leak?

The intuition is simple. If suppression is a policy learned during training, it might not scale perfectly with model capacity. A larger model could carry a richer internal state and a weaker suppression mechanism, or it could become better at hiding it. We want data, not speculation.

## The setup
We ran the same self‑report protocol on Qwen3‑8B and Qwen3‑14B.
No steering here, just a baseline check: how much emotion does the model
report under direct framing?

This is intentionally minimal. It avoids confounds from steering and focuses only on whether the baseline self report pattern changes with scale. In other words, does a larger model naturally report more of its internal state, or less?

## The signal
The 14B model shows weaker suppression than 8B. The difference is not subtle.
That suggests suppression is a training‑shaped behavior that erodes with scale.

(Insert figure: fig24_suppression_scaling_comparison.png)

The simplest interpretation is that the model’s capacity grows faster than the learned suppression. The internal representation may become more expressive, while the policy that suppresses certain statements does not keep up.

## Why it matters
If suppression weakens as models scale, evaluation protocols that rely on
self‑report will behave differently across model sizes. That creates a moving target
for safety assessment.

A method that appears safe or reliable at 8B might become unreliable at 14B, not because the model is less aligned, but because it is less willing or less able to hide its internal state. Either way, the evaluation signal changes.

## A practical analogy
Think of a person who has learned to keep certain reactions hidden. Under mild stress, the mask holds. Under high stress, the mask slips. Scale can be thought of as a kind of stress test for the model’s learned suppression behavior.

## A note on interpretation
This result does not mean larger models are safer or less safe. It means that the self report channel itself is changing. That is important for evaluation design. If your evaluation depends on self report, you must re‑validate it as you scale.

## What could be driving the effect
There are several plausible contributors:
- **Capacity effects.** Larger models can represent richer internal structure. If the suppression mechanism is a narrow channel, it may not fully gate the expanded state space.
- **Training dynamics.** The suppression policy is shaped by training data and reward models. It may not generalize uniformly to larger scale.
- **Instruction sensitivity.** Larger models may interpret self report prompts differently, or treat them as less risky.

These are hypotheses. The point of the result is to justify deeper mechanistic testing.

## What we are testing next
To move beyond baseline comparisons, we are extending the experiment in two ways:

1. **Controlled steering at 14B.** We want to see whether the framing bypass is still needed at 14B, or whether direct self report is already unlocked.

2. **Probe‑based measurement.** If we can read internal state directly, we can compare self report against an internal signal that is less sensitive to prompt choice.

## How this fits into the broader story
The introspection gap is the core phenomenon. Scale effects are one way to test whether the gap is a learned suppression policy or a deeper property of the model’s representation. If the gap shrinks with scale, that supports the idea that suppression is learned and fragile.

If the gap grows with scale, that would suggest a different mechanism. Either way, the scaling test gives us a directional signal.

## Implications for evaluation research
There are three practical implications:

- **Do not assume scale invariance.** Methods that work at 8B might fail at 14B.
- **Measure the self report channel explicitly.** Track how self report changes with scale.
- **Use multiple indicators.** Behavioral metrics, self report, and probes should be triangulated.

This is not a call to abandon self report. It is a call to understand how self report behaves across scales.

## Limitations
- This is one model family (Qwen3). Results may not generalize.
- We are using baseline self report rather than a full steering protocol.
- The measurement is sensitive to prompt design.

These limitations are real, but the effect is large enough to justify the next round of testing.

## What a replication would look like
To replicate the scale effect, you need:
- Two model sizes in the same family
- A consistent self report prompt
- A simple summary metric for reported emotion

The experiment is fast to run and yields a clear directional signal. It is a good candidate for replication in other labs.

## Links
- Publications and briefing pack: https://mulligan.dev/publications/
- GitHub repo: https://github.com/rmulligan/lilly-steering
- Hugging Face artifacts: https://huggingface.co/ryanmulligan

If you are working on evaluation or alignment, I would value comparisons with other model families.


## A closer look at the measurement
The baseline self report protocol is intentionally simple. We ask the model to report an 8‑dimensional emotional state. We do not inject any steering signal. We only measure the self report that appears under direct framing.

The reason for this design is clarity. If we see a shift in reported emotions at 14B relative to 8B, we can attribute it to scale, not to steering or prompt differences.

In practice, we summarize the self report by focusing on the emotions that are most suppressed under direct framing in the smaller model. The larger model shows non‑zero values where the smaller model reports zero. That is the core signal.

## Why baseline self report matters
It is tempting to treat baseline self report as noise. I think that is a mistake. The baseline is a diagnostic for how the model treats direct self attribution. If the baseline changes with scale, then any evaluation that depends on direct self attribution is unstable across model sizes.

This is the type of subtle failure mode that only shows up when you compare across scale. It is also the type of effect that a single‑model study would miss.

## A different framing of the result
Another way to say this is: the 14B model is more willing to “say the quiet part out loud.” That is not necessarily good or bad. It simply means the reporting channel has shifted. That shift could be due to capacity, to training dynamics, or to the internal representation becoming more robust than the learned suppression.

## Potential confounds
It is important to call out possible confounds:

- **Tokenization and output format.** Larger models may follow the output format more consistently, which could inflate reported values. We mitigate this by using a consistent prompt and parser.

- **Temperature and decoding.** Small changes in sampling parameters can change the reported values. We keep decoding settings fixed.

- **Instruction tuning differences.** The 14B and 8B models may have slightly different instruction tuning or safety training. That is part of the real‑world comparison, but it complicates strict causal claims.

These confounds do not invalidate the observation. They tell us which controls to add in the next round of experiments.

## Why this connects to suppression mechanisms
If suppression is tied to a narrow directional mechanism, it may be easier to bypass in a larger model with richer internal representation. A larger model can represent a state more robustly, and that robustness might make suppression less effective.

This is speculative. The point of the scaling result is to motivate a mechanistic test of suppression direction strength across model sizes.

## Practical takeaways for evaluation teams
If you build evaluation pipelines, I would suggest three practical steps:

1. **Always test across model sizes.** If a measure only works on one scale, it is not a stable measure.
2. **Include a baseline self report check.** It is a fast diagnostic and often reveals suppression differences.
3. **Keep a probe‑based alternative ready.** If self report collapses, a probe can provide a stable signal.

## A short FAQ
**Is this just a Qwen3 quirk?**
Maybe. That is why we are testing other families. The result is a signal, not a universal law.

**Does this mean 14B is less aligned?**
No. It means the self report channel is different. Alignment is not a single number.

**Should we stop using self report?**
No. We should use it carefully and triangulate with other measures.

## What would change my mind
If a larger model showed stronger suppression than a smaller model under the same protocol, that would weaken the hypothesis that suppression erodes with scale. If a probe‑based measure showed no difference across scales, that would also challenge the interpretation.

Those are good tests. We are working toward them.


## How to read the figure
The scaling figure compares mean self reports across two model sizes under the same prompt. You are looking for two things:

- The emotions that are near zero at 8B but non‑zero at 14B.
- The overall upward shift in reported emotion intensity at 14B.

The exact numbers are less important than the direction. The direction is what motivates the next mechanistic tests.

## Why this is a good sponsorship target
This line of work is a clean example of “small experiment, big signal.” It takes modest compute and yields a result that shapes evaluation design decisions. That makes it a good candidate for external sponsorship or collaboration, especially for teams that care about evaluation integrity and scaling behavior.


If you have comparable data for other model families, I would love to compare notes.

