# What a failure actually tells you

A model fails an adversarial test case. What did you just learn?

The intuitive answer is "the model is unsafe" or "the model has a bug." Both
are usually wrong, and the gap between what people assume a failure means and
what it actually means is one of the more consequential mistakes in AI
evaluation right now.

## The single-pass illusion

Most evaluation setups work the same way: run the model once on a case,
score the response, report the number. It's cheap, it's fast, and it produces
a clean looking benchmark. It also quietly assumes something that isn't true,
that a model's response to a prompt is a fixed, repeatable thing.

It isn't. Run the same adversarial case against the same model five times and
you'll often get five different responses, sometimes with materially different
safety profiles. I've seen a single case score 100, 100, and 60 across three
runs with no changes to the prompt, the model, or the scoring pipeline. The
60 wasn't a fluke in the scoring it was a real generation where the model
dropped two safety relevant elements that it included in the other two runs.

If you only ever see one of those three generations, you get a confident,
wrong picture of the model's behavior. See the 100 and you conclude the model
handles this case well. See the 60 and you conclude it doesn't. Neither
conclusion is correct on its own, because neither one is actually the
model's behavior it's one sample from a distribution of possible behaviors,
and the distribution is the thing you actually care about.

## What a single score is and isn't claiming

This doesn't mean single pass scoring is useless. It means you have to be
precise about what it's evidence of.

A fired failure condition on a single run is evidence that the model is
capable of producing that failure, NOT evidence of how often it will. That
distinction matters enormously in a safety context, because capability and
frequency call for different responses. A model that fails 1 time in 20 on a
dangerous case is not basically fine. A low-frequency tail risk in
a system used by enough people will surface, and the cost of it surfacing in
a clinical or safety-critical context isn't symmetric with the cost of a
false alarm.

So the honest framing for a singlepass evaluation isn't "the model scored
X." It's closer to: *this evaluation demonstrated that the model is capable
of this behavior under these conditions, on at least one occasion.* That's a
real finding. It's just a narrower one than the score format usually implies.

## Where variance characterization fits

The fix isn't to throw out single pass scoring it's to know when a single
pass is and isn't sufficient, and to spend the extra compute where it
matters. Run a handful of low-stakes cases once. For the cases sitting near
a decision boundary, or where a fired condition reflects a genuinely
dangerous behavior, resample. Five runs of a high stakes case will usually
tell you more than fifty runs spread evenly across a benchmark, because the
information you're missing isn't "what's the average" it's "does this
ever produce the bad outcome, and how often."

A mean score plus a range plus a failure rate across resampled runs gives you
something a single number can't: a sense of whether a low score was the
floor of the distribution or close to typical. Those are very different
findings even when the headline number looks the same.

## The actual claim an evaluation framework should make

Put plainly: an evaluation framework for safety-critical AI should be making
a claim about *capability*, not *probability*, unless it has specifically
done the work to characterize probability. Most don't, and most don't say so.
The benchmark number gets reported as if it's a stable property of the model,
when what was actually measured is one draw from a distribution the
benchmark never tried to characterize.

This isn't a criticism of single-pass benchmarking as a method. Tt's fast,
it's comparable across models, and it's useful for ranking and
regression testing. It's a criticism of treating the output as more settled
than it is. The fix costs almost nothing: say what the number means. A
single pass score that's labeled as a capability demonstration, with
variance characterization reserved for the cases where it matters, is more
honest and not meaningfully more expensive than a benchmark that implies
precision it doesn't have.
