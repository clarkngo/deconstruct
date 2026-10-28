# Null Hypothesis (Neyman–Pearson Testing)

> **The 10-Second Elevator Pitch:** A null hypothesis is the default story, usually "nothing special is going on," that you put on trial so the data can knock it down. Neyman and Pearson's insight was that no test can tell you whether one particular hypothesis is true. A test is a *rule of behavior* that keeps you from being wrong too often in the long run. You can be wrong in two ways, raising a false alarm or missing a real effect. So you fix how often you'll tolerate false alarms, then pick the test that misses the least.

---

## 1. The Pain Point (Why was this invented?)

- **The Old/Status Quo Approach:**
  - **The common method.** The paper traces it to the French mathematician Bertrand and calls it the method "in common use". Pick some feature of the data, calculate the probability P of seeing it if the hypothesis H were true, and treat a very small P as a sign that H is probably false. One of Bertrand's examples: do these stars cluster on the sky because they form a real group in space, or by chance?
  - **Which feature to measure was left to taste.** The paper says the measures people used seemed chosen by "a happy intuition". Usually they were averages and spreads of the sample, and they were fine as long as the data looked roughly bell-shaped.

- **Why It Sucked:**
  - **A small probability alone proves nothing.** This was Bertrand's own objection. With a continuous measurement, *every* exact outcome has probability zero, so "this would be unlikely by chance" is true of any result. Neyman and Pearson agreed that, for one particular hypothesis, a probability calculation can't by itself give valuable evidence of truth or falsehood.
  - **No principled way to pick the test.** Different measures could test the same hypothesis and give different answers, and the authors say the statistical literature was "full of examples of confusion of thought" on this choice.
  - **Example: comparing two percentages.** The standard recipe computes each group's error separately, as if the two groups had different underlying rates. But the hypothesis being tested *says the rates are equal*, so the consistent move is to pool both samples to estimate the shared rate. Habit produced a test that contradicted its own hypothesis.

---

## 2. The First-Principles Insight (The "Aha!" Moment)

- **The Core Intuition:** Change the question. Stop asking "is this hypothesis true?", which no probability calculation can answer for a single case. Ask instead: **"What decision rule, followed over and over, keeps my mistakes rare?"** Once tests are viewed as rules, two facts fall out:
  - **There are exactly two ways to be wrong.** You can reject the hypothesis when it's actually true, which is a false alarm. Or you can accept it when it's false, which is a miss. You usually can't eliminate both.
  - **Controlling false alarms alone is easy, and it isn't enough.** Every test amounts to marking out a "reject" zone of possible results. You can make the false-alarm chance as small as you like just by shrinking that zone. Many differently shaped zones share the *same* false-alarm rate, and nothing about false alarms tells them apart. What separates a good test from a useless one is **how often it catches the truth when the hypothesis is false**. That means you must say what the alternatives are.

  So the recipe becomes: **fix the false-alarm rate, then pick the reject zone that misses least against the alternative.** That zone turns out to be the set of results that are much more likely under the alternative than under the null. In other words, reject where the *ratio* of the two likelihoods tips far enough toward the alternative. The paper speaks of a test's "power of discrimination" between true and false. *(Outside the source: the result is now known as the Neyman–Pearson lemma, and the catch rate is simply called a test's **power**.)*

- **Real-World Metaphor:** The paper's own example is a court. How many judges' votes should it take to convict? Convicting an innocent person and freeing a guilty one are both errors, and which is worse depends on the stakes: execution or a fine, the danger of releasing a criminal, current views on punishment. The null hypothesis is the defendant's innocence. Statistics can't decide where to strike the balance, but it can make sure that once you choose how many innocent people you're willing to wrongly convict, you free as few guilty ones as possible.

---

## 3. How It Works Under the Hood (Step-by-Step)

1. **State the hypothesis so precisely that it predicts every outcome.** The hypothesis being tested, which the paper calls H₀, must fix the probability of every possible sample. If it pins down everything (for example, "the mean is a₀ and the spread is σ₀"), it's **simple**. If it leaves some details open (for example, "the mean is a₀, spread unknown"), it's **composite**. That second case is exactly the problem behind Student's [t-test](entry.html?e=t-test).

2. **List the alternatives.** Write down which other explanations, H₁, H₂, and so on, you consider possible. Without them there is no way to tell a good test from a bad one.

3. **Choose your false-alarm risk, ε.** This is how often you'll accept rejecting H₀ when it's true. The paper's examples use 5% and 1%. How to balance the two kinds of error depends on the consequences, and the authors leave it to the investigator.

4. **Draw the best reject zone.** For a given alternative, reject when the data are at least some multiple more likely under the alternative than under H₀. Then tune that multiple until the zone's false-alarm chance equals ε.
   - **Worked example from the paper:** testing whether a normal population with known spread σ₀ has mean a₀, against alternatives with a larger mean, at ε = 5%. The best zone is "reject if the sample average exceeds a₀ + 1.645·σ₀/√n."
   - That is just the familiar textbook test. The new part is the **proof that no other test using the same data can do better**.

5. **Handle harder cases.**
   - **Alternatives on both sides.** If the alternatives lie in both directions (mean higher *or* lower), no single zone is best against all of them. The paper's practical advice is to test against the side the data actually point to.
   - **Composite hypotheses.** For these, the zone must keep the same false-alarm rate whatever the unspecified details turn out to be. The paper calls such zones "similar regions," and the t-test's zone is one of them.
   - **No single best zone.** When none exists, the zone picked by the [likelihood ratio](entry.html?e=likelihood-ratio-test) method is the paper's recommended compromise.

6. **Decide, and read the decision correctly.** Reject H₀, accept it, or stay in doubt. The rule says nothing about whether H₀ is true *this time*. What it guarantees is that, over many uses, you'll wrongly reject a true H₀ no more than ε of the time, while catching false ones as often as possible.

> Note: "Accept H₀" here means "act as if H₀ holds," not "H₀ has been shown true." The guarantee is about the long-run behavior of the rule, never about the single case in front of you.

---

## 4. Key Trade-offs & Limitations

- **When to use it:**
  - **Repeated decisions where long-run error rates matter**, such as screening, inspection, or any process that runs the same test many times.
  - **Choosing between candidate tests.** Fix the false-alarm rate and compare how often each one misses.
    - **Example from the paper:** testing whether a population's spread is σ₀ when its mean is already known, with 5 observations and a 1% false-alarm rate. The usual sample variance *ignores* the known mean, and it misses **49%** of the time when the true spread is twice σ₀ and **17%** when it's three times σ₀. The paper shows that measuring spread around the *known* mean misses less often at every alternative, because it uses all the information.
  - **When you can name the alternatives.** That's what makes "best" well defined.

- **When it fails:**
  - **It can't judge the single case, by design.** A rejection doesn't mean H₀ is probably false. It means you followed a rule with a known error rate.
  - **It needs alternatives.** If you can't say what else might be true, you can't measure misses, and the theory has nothing to optimize.
  - **No universal best test in many problems.** When different alternatives call for different reject zones, which "good" zone to use was left open. The likelihood-ratio choice was intuitively sound but not proven best.
  - **Composite hypotheses are only partly solved.** The authors could find the best zones for hypotheses with unknown details only under limiting conditions.
  - **The balance between errors isn't math.** How much false-alarm risk to accept, and how costly misses are, are judgment calls about consequences.
  - *(Outside the source:)* The phrase "null hypothesis" doesn't appear in this paper; it calls H₀ the "hypothesis tested". R. A. Fisher coined the term in *The Design of Experiments* (1935). Fisher saw a test as evidence against the null, summarized by its p-value, not as a rule of behavior, and he and Neyman disputed this for decades. Today's standard practice blends the two views: a p-value compared to α, plus power calculations. Critics say the blend is a common source of misinterpretation.

---

## 5. Speak Like an Expert (Cheat Sheet)

- **If someone asks:** "What is a null hypothesis, really?"
- **Say this:** "It's the default explanation we put on trial, usually 'no effect', stated precisely enough that we can calculate how likely any result would be if it were true. We reject it only when the data land in a zone that would rarely occur under it and is much more likely under the alternatives we care about."

- **If someone asks:** "We got a significant result at 5%. Does that mean there's a 95% chance the effect is real?"
- **Say this:** "No. The 5% describes the rule, not this result. If there were truly no effect, a rule like ours would raise a false alarm about 5% of the time. Whether this particular finding is real also depends on how often the test catches real effects and how plausible the effect was to begin with."

- **If someone asks:** "Why do we need an alternative hypothesis if we're only testing the null?"
- **Say this:** "Because the false-alarm rate alone can't tell good tests from bad ones. You can hit 5% with a useless test. What makes a test good is how often it catches a real effect, and that only means something once you say what the real effect might look like."

---

<p class="source">Source: J. Neyman &amp; E. S. Pearson, "On the Problem of the Most Efficient Tests of Statistical Hypotheses," Philosophical Transactions of the Royal Society A 231 (1933), 289–337. The notes on the Neyman–Pearson lemma and power as named terms, Fisher's coining of "null hypothesis," and the Fisher–Neyman dispute come from outside the paper.</p>
