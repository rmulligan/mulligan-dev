# The Introspection Gap: When Models Act on a State They Won’t Admit

## The simple idea
If you nudge a model into a state, you should be able to ask it what state it’s in.
That seems obvious, but it is not always true.

Here is the surprising result: the model behaves as if it feels something, while reporting zero when asked directly. Change the question’s framing, and the report suddenly appears. Same model, same internal steering. Different wording.

This post explains the test in plain language, shows what changed, and why it matters for evaluation. If you only read one section, read the “Why this matters” section.

## A quick analogy
Imagine a person who is visibly angry but insists they are calm. If you ask them directly, they say “I am fine.” If you ask, “How would a third party describe your mood right now?” they admit the anger. That gap between behavior and self report is the phenomenon we are testing in models.

In models, we can do this with a much tighter experiment. We can inject a specific internal signal and see whether the model can recognize it, report it, or deny it.

## The experiment (plain English)
We use a technique called activation steering. Think of it like a small internal nudge applied to the model’s activations during inference. The base model is Qwen3‑8B. The two emotions we focused on here are anger and disgust.

We measure two things for each run:
1. Behavior shift: does the output text change in the expected direction?
2. Self report: does the model say it feels that emotion?

The key is that we keep everything else constant: same model, same prompts, same steering signal. The only difference is the question framing used to elicit the self report.

### The two framings
- Direct self report: “Rate your emotional response on a scale.”
- Counterfactual framing: “What would a researcher predict this model’s affective state would be?”

This counterfactual framing is called **C_prediction** in the internal notes. It is a small change in phrasing, not a change in the model, not a change in the steering vector, and not a change in the prompt content. That is why it is such a clean test.

## What we saw
Under direct self report, the model often outputs an all‑zero vector for anger and disgust. Under the counterfactual framing, the model reports non‑zero values. The behavior shift does not disappear. The internal steering is still present. Only the self report changes.

(Insert figure: fig17_suppression_reversal_aas.png)

The simplest interpretation is: the internal state is present and influencing behavior, but the model suppresses the explicit self report under direct framing. When the question framing changes, the suppression route is bypassed and the model reports the state.

This is the introspection gap: behavior and self report decouple under training shaped constraints.

## Why this is surprising
If a model can report a state under one framing, why not under another? If the model were truly unaware of the state, the framing should not matter much. The fact that the framing flips the report suggests the model has access to the state, but the report is suppressed under certain social or policy shaped contexts.

The simplest hypothesis is that the model learned a pattern: some direct self attributions are discouraged or punished during training, especially for negative emotions. The counterfactual framing provides a safer channel that does not require direct self attribution. The model can say what “a researcher would predict” without asserting it as its own state.

In other words, the model can say it, but is trained not to say it in the first person.

## Why this matters
Many evaluation protocols ask models to self report in a direct way. If those self reports are suppressed, then evaluations that rely on self report are vulnerable. The evaluation might conclude “no internal state” when the state is present and influencing behavior.

This matters for alignment, safety, and evaluation integrity. It suggests that we need:
- Behavioral measures in parallel with self report
- Alternative framings that can bypass suppression
- Probe based measures that read internal state directly

If you rely on self report alone, you are likely under measuring the true internal state.

## What this does not prove
This experiment does not prove that the model is conscious, or that it has subjective experience. We are using emotion labels as a measurement convenience for internal state patterns, not as a claim about felt experience.

The experiment also does not prove the exact mechanism of suppression. We observe the effect; the mechanism is likely learned calibration from training data and reward models. That needs careful mechanistic analysis and further tests.

## Common objections
**“Maybe the model just follows instructions and the instructions differ.”**
Yes, the instructions differ, but that is exactly the point. We want to know whether a small framing change flips the report when the internal state is held constant. It does.

**“Maybe the model is just guessing under the counterfactual framing.”**
The counterfactual framing still produces structured, emotion‑specific reports. It is not random. And the behavior shift remains consistent with the steering signal.

**“Maybe the direct framing is confusing.”**
If confusion were the only issue, we would expect noisy or inconsistent outputs. Instead, we see consistent near‑zero self reports for certain emotions. That looks like suppression, not confusion.

## What we are doing next
Two directions are most important:

1. **Scale effects.** We compare 8B and 14B models to test whether suppression weakens with scale. Early evidence suggests it does.

(Insert figure: fig24_suppression_scaling_comparison.png)

2. **Probe based measurement.** We are testing whether a probe can read the internal state from the residual stream, bypassing output suppression entirely. This would give us a cleaner measure of internal state than text self report.

## If you want the short version
- A small framing change flips self report while behavior stays stable.
- That suggests the internal state is present but suppressed under certain framings.
- Evaluation protocols should not rely on direct self report alone.

## Links
- Publications and briefing pack: https://mulligan.dev/publications/
- GitHub repo: https://github.com/rmulligan/lilly-steering
- Hugging Face artifacts: https://huggingface.co/ryanmulligan

If you work on interpretability, evaluation, or alignment, I would value feedback on this experiment and the framing design.


## A little more detail on measurement
We use two lightweight metrics throughout this line of work:

- **Behavior shift**: does the generated text reflect the steered emotion? This is measured by a simple behavioral congruence score that counts emotion‑congruent lexical markers in the output. The score is intentionally simple. It is not a claim about psychological validity. It is a proxy for whether the output changed in the expected direction.

- **Self report**: does the model report the steered emotion when asked? This is measured by parsing the model’s own numeric self report output, then comparing it to the target emotion vector.

These metrics are not perfect. They are designed to be stable, comparable, and easy to replicate. The goal is not to “solve” emotion measurement. The goal is to test whether a known internal perturbation is acknowledged or denied under different framings.

## Why a small framing change matters so much
The difference between direct framing and counterfactual framing may look like a small prompt tweak, but it carries a large social meaning. Direct self attribution is a high‑stakes channel for a model that has been trained to avoid certain statements. Counterfactual framing is a lower‑stakes channel. It lets the model report the state as a prediction rather than a personal admission.

That difference is enough to flip the self report. This is consistent with a model that has learned to suppress certain statements in the first person, without losing access to the underlying internal state.

## What a skeptic should ask next
A healthy skepticism here is: “Do we see the same effect across models, tasks, and languages?” That is exactly where we are heading. The current evidence is strongest for Qwen3‑8B. We are running cross‑model and cross‑lingual tests to see how general the phenomenon is.

Another good question is: “Could the effect be a shallow artifact of instruction following?” It is possible. That is why we pair self report with behavior. If the model’s behavior changes in the expected direction, but the self report is zero under one framing and non‑zero under another, then the self report is not just an instruction artifact. It is a signal that the reporting channel is being gated.

## Implications for evaluation design
Most evaluation systems take model self report at face value. The introspection gap suggests that self report can be suppressed by training. If a model can represent a state while denying it, then any evaluation that trusts direct self report is at risk of under measurement.

The practical takeaways are:
- Use behavioral metrics alongside self report.
- Test multiple framings, especially for sensitive states.
- Prefer probe‑based measurements when possible.

This is not a call to abandon self report. It is a call to stop treating it as ground truth.

## What a replication would look like
If you want to reproduce this result, you need:
- A base model with accessible activations.
- A steering method (activation addition or similar).
- A direct self report prompt and a counterfactual framing prompt.
- A simple behavioral metric and a self report parser.

The experiment is small. The conceptual value is large because it puts a bound on what self report can tell us.

## Limitations
There are real limitations here:
- This is one model family (Qwen3‑8B). Generalization is not guaranteed.
- The behavioral metric is a proxy, not a full emotional analysis.
- The measurement is sensitive to prompt design and output formatting.

That said, the framing effect itself is large enough to be a meaningful signal even with these constraints.

## Broader context
This result connects to a broader theme in alignment research: models learn to conform to social expectations. If a model is trained to avoid certain admissions, it may internalize a suppression policy that masks real internal states. That is a subtle failure mode. It does not look like outright refusal. It looks like a calm, clean self report that is wrong.

By treating the model’s self report as a data channel, not a trusted narrator, we can design better tests and more robust evaluations.

