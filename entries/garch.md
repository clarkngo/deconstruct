# GARCH (Generalized Autoregressive Conditional Heteroskedasticity)

> **The 10-Second Elevator Pitch:** Some numbers, like inflation or prices, don't just move up and down. How *wildly* they move changes over time, and calm stretches and stormy stretches each tend to last a while. GARCH forecasts the next period's jumpiness from two things: how big the latest surprise was, and how jumpy you already expected things to be. Feeding that previous expectation back in gives the model a long memory using only three numbers.

---

## 1. The Pain Point (Why was this invented?)

- **The Old/Status Quo Approach:**
  - **Constant variance.** Standard time series and regression models assume the size of the random error never changes. The forecast gets one fixed error bar, the same in a crisis as in a quiet decade.
  - **ARCH (1982).** Robert Engle's [ARCH](entry.html?e=arch) model broke that assumption. It lets this period's expected spread of surprises (the *conditional variance*) depend on the squared sizes of the last few surprises. A big shock widens the error bars for the next few periods.

- **Why It Sucked:**
  - **A fixed error bar ignores the calendar.** US inflation was volatile and hard to predict from the late 1940s to the mid-1950s, then stable through the 1960s. In the paper's example, the constant-variance model's 95% bands look "much too wide" for the calm 1960s.
  - **ARCH needs a long memory to fit real data.** Uncertainty in real economic series fades slowly, so ARCH needs many past surprises (many lags), each with its own weight.
  - **Estimating all those weights freely breaks the model.** Variance can't be negative, but free estimates often came out negative. So practitioners fixed the shape by hand. The standard inflation model used 8 quarters of past surprises with weights forced to decline in a straight line. Bollerslev calls that choice "rather ad hoc."

---

## 2. The First-Principles Insight (The "Aha!" Moment)

- **The Core Intuition:** Don't just feed in past *surprises*. Also feed in the model's own previous *estimate of variance*. Last period's estimate already summarizes every surprise before it, so adding that one term gives the model a memory that reaches back indefinitely, with each older surprise counting a bit less. One extra number replaces a long, hand-shaped list of lags.
  - Bollerslev points out this is the same step time series analysts took when they went from autoregressive (AR) models to [ARMA](entry.html?e=arma) models: a short recursive formula replaces a long list of lags.
  - He describes the result as "some sort of adaptive learning mechanism". Each period, your sense of how risky things are gets updated partly by the newest surprise, and mostly carries over from what you believed before.

- **Real-World Metaphor:** Picture a ship's lookout estimating how rough the sea is. Every hour they revise the estimate: mostly keep last hour's number, nudge it toward the size of the wave that just hit, and drift slowly back toward the sea's usual state. One huge wave raises the estimate for many hours, and the effect fades a little each hour instead of vanishing at once. That gradual fade is why rough patches cluster in time, and it's exactly what GARCH writes down.

---

## 3. How It Works Under the Hood (Step-by-Step)

1. **Predict the value first, then measure the surprise.** Fit an ordinary model for the level of the series. In the paper, this quarter's inflation is predicted from the previous four quarters. Each period's **surprise** is the actual value minus the prediction.

2. **Check whether the size of the surprises is predictable.** The surprises can look like pure noise, with no pattern in their signs, while their *squares* still follow a pattern. Big surprises follow big ones and small follow small. Bollerslev's diagnostic is to compute autocorrelations on the **squared** surprises, the same tools analysts already use on the surprises themselves. For pure ARCH with q lags, the partial autocorrelations of the squared surprises drop to zero after lag q. For GARCH, they fade out gradually instead.

3. **Update the variance forecast with three numbers.** The workhorse GARCH(1,1) model is:

   ```
   variance_now = ω + α · (last surprise)² + β · variance_last
   ```

   - **ω (omega):** a small baseline that keeps the variance above zero.
   - **α (alpha):** how hard the newest surprise pushes the estimate. It is the "react to news" term.
   - **β (beta):** how much of the previous estimate carries over. It is the "memory" term.
   - All three must be non-negative, and ω must be positive. Each surprise is then modeled as drawn from a bell curve with this variance.
   - In the general GARCH(p, q), q is the number of past squared surprises and p is the number of past variance estimates included. Setting p = 0 gives back plain ARCH.

4. **Check that it settles down.** If **α + β < 1**, the variance has a stable long-run level, equal to **ω ÷ (1 − α − β)**, and the forecast always drifts back toward it. The sum α + β measures how persistent shocks are. The closer it is to 1, the longer a burst of turbulence lingers. The **median lag**, −log 2 ÷ log β, says how far back you must look to account for half the total weight given to past surprises.

5. **Fit it by maximum likelihood.** Choose the parameters that make the observed surprises most probable, using an iterative search. The paper uses the BHHH algorithm. The update in step 3 refers to the previous period, so the first period needs a starting value; the average squared surprise over the sample works. Bollerslev shows two practical facts:
   - **The prediction model and the variance model can be estimated separately** without losing accuracy in large samples.
   - **The estimates stay valid if the bell-curve assumption is wrong,** as long as the standard errors are computed in a robust way.

> Note: Even though each surprise is drawn from a bell curve, the *long-run* mix of surprises is **heavy-tailed** under GARCH(1,1). Extreme values show up more often than a single fixed bell curve would predict, because some periods have a much larger variance than others.

---

## 4. Key Trade-offs & Limitations

- **When to use it:**
  - **Whenever the size of the swings clusters in time.** The clearest sign is surprises that look random while their squares are clearly correlated. The paper's applications are inflation uncertainty, plus earlier ARCH work on interest-rate risk premiums, exchange rates, and US economic data.
  - **When you need error bars that widen and narrow with conditions.** The paper's GARCH(1,1) gives wide 95% bands for the volatile late 1940s to mid-1950s, tight bands for the stable 1960s and early 1970s, and a modest widening after 1974.
  - **When you need long memory without hand-tuning.** On quarterly US inflation from 1948 to 1983 (143 quarters), the GARCH(1,1) fit gave **α = 0.135** and **β = 0.829**. Its persistence is 0.964, and its median lag is about 3.7 quarters. The 8-quarter ARCH model had its lag shape set by hand, with a median lag fixed at 2½ quarters.
    - **The test results favor GARCH.** Adding last quarter's variance estimate to the hand-shaped 8-quarter ARCH model was statistically significant (test statistic 4.57, significant at the 5% level). Adding the 8-quarter ARCH structure to GARCH(1,1) was not (2.33).
    - **GARCH leaves the most bell-shaped leftovers.** After dividing each surprise by its predicted spread, the kurtosis is 3.81 for GARCH(1,1), against 4.07 for the 8-quarter ARCH model and 6.90 for the constant-variance model. A perfect bell curve scores 3. Bollerslev's verdict is that GARCH fits slightly better *and* has a more sensible lag shape.

- **When it fails:**
  - **It must be stable to make sense.** If α + β ≥ 1, there is no finite long-run variance. Stricter conditions are needed for higher moments. For example, GARCH(1,1) has a finite fourth moment, which you need to measure how heavy the tails are, only if 3α² + 2αβ + β² < 1.
  - **It's hard to test for directly.** You can't run one general test of "GARCH vs. no volatility clustering at all", because the math breaks down. A test for GARCH also can't tell GARCH apart from a longer ARCH model. In practice you check squared-surprise autocorrelations and ARCH tests, then compare specific models.
  - **Fitting is more work than ARCH.** The recursive variance term makes estimation iterative, and you have to supply starting values.
  - **It assumes a bell curve each period.** The paper follows Engle in assuming normal surprises, while noting other distributions could be used. The parameter estimates hold up, but the standard errors need correcting.
  - **Up and down surprises count the same.** The surprise is squared, so its sign is thrown away. The paper notes that using the absolute size instead of the square may suit some applications. *(Outside the source: later work found stock market volatility often reacts more to falls than to rises, and variants such as EGARCH and GJR-GARCH were built to capture that.)*
  - *(Outside the source:)* GARCH(1,1) later became the default way to forecast volatility in finance, for risk limits and option pricing. The paper itself studies only inflation.

---

## 5. Speak Like an Expert (Cheat Sheet)

- **If someone asks:** "What does GARCH actually do?"
- **Say this:** "It forecasts how jumpy a series will be next period. The new estimate is a baseline, plus a reaction to the latest surprise, plus most of the previous estimate carried forward. That carry-over is what makes calm and stormy periods cluster, and it gives the model a long memory with only three parameters."

- **If someone asks:** "Why use GARCH(1,1) instead of ARCH with a lot of lags?"
- **Say this:** "Real volatility fades slowly, so ARCH needs many lags. Estimating them freely tends to produce negative variance weights, so people used to fix the lag shape by hand. GARCH gets the same long memory from one extra term, and in Bollerslev's inflation example it fit slightly better than the hand-shaped 8-lag ARCH model."

- **If someone asks:** "Our forecast errors look like white noise. Why would we need this?"
- **Say this:** "The errors can be unpredictable in direction but predictable in size. Check the autocorrelation of the *squared* errors. In the original paper, the inflation model's errors looked clean, but their squares were clearly correlated, and that's exactly the signal GARCH is built to model."

---

<p class="source">Source: Tim Bollerslev, "Generalized Autoregressive Conditional Heteroskedasticity," Journal of Econometrics 31 (1986), 307–327 (<a href="https://public.econ.duke.edu/~boller/Published_Papers/joe_86.pdf">author's copy</a>). The notes on asymmetric variants and use in finance come from outside the paper.</p>
