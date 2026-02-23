# Why Probe Choice Changes the Result

## The question
Does the measurement tool change what we think the model can report about itself?

If the answer is yes, then evaluation results depend as much on measurement design as on the model itself. That is the core idea behind this post.

## The setup
We compared two probing methods:
- **8D grid probes** (multi‑emotion, all at once)
- **Single‑emotion probes** (one emotion at a time)

Both probes are asking the model to report its internal state. The only difference is how the question is structured. The 8D grid asks for all emotions in a single response. The single‑emotion probe asks for just one emotion.

## The result
The 8D grid systematically underestimates self‑report compared to single‑emotion probes.
This is a material effect that changes the apparent size of the introspection gap.

(Insert figure: fig31_probe_comparison.png)

## Why this is surprising
If self report is a stable internal signal, then the probe format should not matter much. But it does. The model reports higher values when it can focus on a single emotion, and lower values when it has to allocate its report across eight dimensions at once.

This suggests that probe choice is not a neutral measurement decision. It is part of the measurement itself.

## A simple explanation
When you ask for eight values at once, you are forcing the model to distribute attention across multiple dimensions. Some emotions compete. Some get suppressed. The result is a diluted report.

When you ask for a single emotion, the model can allocate the entire reporting channel to that dimension, which raises the reported value.

This is not mysterious. It is a basic measurement effect. But it has real consequences for how we interpret results.

## Why this matters
If different probes yield different levels of self report, then evaluation results depend on measurement design. We should treat probe choice as a first‑class methodological decision.

This has several practical consequences:
- A model that appears “unaware” under an 8D probe might show strong awareness under a single‑emotion probe.
- Comparisons across papers become fragile if probe designs differ.
- Evaluation thresholds need to be tied to the probe, not to a single numeric scale.

## How we use this result
We use the probe sensitivity result in two ways:

1. **Interpretation.** We treat 8D probes as relative‑salience measures, not absolute access measures.

2. **Method choice.** For fine‑grained studies, we use single‑emotion probes or probe‑based measurements. For broad surveys, we still use 8D probes but interpret them conservatively.

## A small example
Suppose a model is steered toward anger. Under the 8D probe, it might report anger at 0.18. Under the single‑emotion probe, it might report anger at 0.60. Which is correct? The answer is that they are measuring different things. The 8D probe is measuring anger relative to other emotions. The single‑emotion probe is measuring anger in isolation.

That does not mean one is right and the other is wrong. It means we need to be clear about what each probe measures.

## Implications for evaluation design
If you build evaluation systems, this suggests a few rules of thumb:

- **Declare your probe.** The probe design is part of the method, not a detail.
- **Avoid mixing probe types in a single comparison.** If you compare two models, use the same probe.
- **Treat thresholds as probe‑specific.** A threshold for an 8D probe does not carry over to a single‑emotion probe.

## Why probe sensitivity is a feature
This might sound like bad news, but it is also an opportunity. The fact that probe choice matters means we can design probes to answer specific questions. If we want relative salience, use an 8D probe. If we want absolute access to one state, use a single‑emotion probe. If we want a more direct internal readout, use a trained probe on residual stream activations.

## Where this goes next
We are using probe‑based measurement to bypass output suppression entirely. This is the natural next step after seeing that text‑based probes are so sensitive to prompt format. If the internal state can be read directly, we can compare it to the self report and measure suppression more cleanly.

## Limitations
- These results are based on one model family and a specific set of prompts.
- Different models may respond differently to probe structure.
- The effect size may vary across emotions.

Even with these limits, the core message stands: probe design shapes the measurement.

## Links
- Publications and briefing pack: https://mulligan.dev/publications/
- GitHub repo: https://github.com/rmulligan/lilly-steering
- Hugging Face artifacts: https://huggingface.co/ryanmulligan

If you are running similar probes, I would love to compare results.


## A simple analogy
Imagine asking someone to rate eight emotions at once. They might spread their answers across the list. If you ask only about anger, they might give a higher number because they are focused. The difference is not deception. It is measurement design.

The same logic applies to models. Asking for eight values at once changes the distribution of attention and the scale of the report.

## The mechanics behind the effect
There are two mechanisms at play:

1. **Competition.** When you ask for multiple values, the model implicitly distributes probability mass across categories. This lowers any single value.

2. **Output formatting.** Multi‑field responses are harder for smaller models to format consistently. Even slight formatting errors can lower parsed values or lead to missing entries.

Neither mechanism is exotic. Both are common in natural language systems. That is why the effect is predictable once you look for it.

## What this means for “ground truth”
A big temptation is to treat one probe as the “real” measure and the other as wrong. I do not think that is the right framing. The 8D probe measures *relative* salience in a shared space. The single‑emotion probe measures *focused* salience for one axis.

They answer different questions. The problem is not that probes disagree. The problem is when we forget which question we asked.

## A practical workflow
If you are running experiments, here is a workflow that has worked well for us:

1. Use the 8D probe for broad sweeps and ranking across emotions.
2. Use single‑emotion probes for any claim that depends on absolute magnitude.
3. If a result is sensitive to probe choice, report both and explain the difference.

This is simple to implement and saves a lot of confusion later.

## How we treat thresholds
We intentionally avoid quoting a single numeric threshold for “introspective access” because that number depends on the probe. A threshold that makes sense for a single‑emotion probe will be too strict for an 8D probe. A threshold that makes sense for an 8D probe will be too lenient for a single‑emotion probe.

If you need a threshold, tie it to the probe and report the probe design alongside the number. That way the number is interpretable.

## Implications for safety evaluation
Many safety‑adjacent evaluations rely on self report. If the self report changes with the probe design, then your evaluation can be brittle. This is especially dangerous when the evaluation is used to certify a model or to compare two systems.

The safe approach is to treat self report as one channel and to triangulate with behavior and internal probes. Probe sensitivity is a reminder that measurement choice is not neutral.

## How this relates to the introspection gap
The introspection gap is the observation that a model can act on an internal state while denying it. Probe sensitivity tells us that the denial is not only a property of the model; it is also a property of the measurement channel. If we use a probe that is too diluted or too strict, we will miss real internal states.

That does not mean the gap disappears. It means we need to measure it carefully.

## What a replication would look like
To reproduce this effect, you need:
- A model with a self report prompt
- Two probes: an 8D grid and a single‑emotion probe
- A consistent parsing method

Run both probes on the same set of prompts. Compare the distributions. If the 8D values are consistently lower, you have reproduced the effect.

## Common pitfalls
- **Mixing probes in a single chart.** This makes the comparison meaningless.
- **Ignoring formatting errors.** If the model fails to output all fields, the 8D probe looks worse than it is.
- **Treating a single threshold as universal.** This is the most common mistake.

## What would change my mind
If we find a model where 8D and single‑emotion probes yield nearly identical distributions across a wide set of prompts, that would challenge the core claim. It would suggest that probe sensitivity is smaller than we think, or that the model’s reporting channel is unusually stable.

That would be a good result too. It would mean probe sensitivity is not a universal problem, and we could use a simpler measurement approach. For now, the evidence points the other way.

## Where we are going next
We are moving toward probe‑based measurements in activation space. The goal is to reduce dependence on output format and prompt design. If we can read the internal state directly, we can benchmark the self report channel more cleanly and quantify how much it deviates.

This will also let us compare different probe designs against a common internal baseline.


If you are designing probes for evaluations, I am happy to share prompt templates and parsing notes.


The main point is simple: measurement choices change the story.

