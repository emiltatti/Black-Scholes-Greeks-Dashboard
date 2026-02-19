# Black–Scholes Greeks Dashboard (Calls & Puts)

This project builds a small “Greeks dashboard” to visualize how **delta**, **gamma**, and **vega** change across a grid of **spot prices** and **times to maturity** under the **Black–Scholes** model.  
The goal is not just to compute closed-form formulas, but to build intuition for how option risk behaves across regimes (ATM vs OTM, long-dated vs near expiry).

---

## What this does

I fix a single set of model inputs:

- Strike: $K$
- Risk-free rate: $r$
- Volatility: $\sigma$

Then I build a 2D grid over:

- Spot prices: $S \in [S_{\min}, S_{\max}]$
- Maturities: $T \in [T_{\min}, T_{\max}]$

On every $(S, T)$ point, I compute:

- Option price (call and put)
- Delta $\Delta$
- Gamma $\Gamma$
- Vega $V$

Finally, I plot **heatmaps** (calls on the first row, puts on the second row), and I annotate the key intuition regions.

---

## Why a grid is useful

Greeks are functions of $(S, T)$, so the same option can behave very differently depending on where you are:

- A deep ITM call has $\Delta \approx 1$, but that is not true when it is ATM.
- Gamma can be small for long maturities, then explode as you approach expiry.
- Vega is usually largest around ATM, but it behaves differently near expiry.

A grid makes these relationships visible instantly.

---

## Model and formulas

I assume Black–Scholes with **no dividends**.

First I compute:

$$
d_1 = \frac{\ln(S/K) + \left(r + \frac{1}{2}\sigma^2\right)T}{\sigma\sqrt{T}}
$$

$$
d_2 = d_1 - \sigma\sqrt{T}
$$

Let $\Phi(\cdot)$ be the standard normal CDF and $\phi(\cdot)$ the standard normal PDF.

### Call price

$$
C = S\Phi(d_1) - Ke^{-rT}\Phi(d_2)
$$

### Put price

$$
P = Ke^{-rT}\Phi(-d_2) - S\Phi(-d_1)
$$

### Delta

Call:

$$
\Delta_{\text{call}} = \Phi(d_1)
$$

Put:

$$
\Delta_{\text{put}} = \Phi(d_1) - 1
$$

### Gamma (same for calls and puts)

$$
\Gamma = \frac{\phi(d_1)}{S\sigma\sqrt{T}}
$$

### Vega (same for calls and puts)

$$
V = S\phi(d_1)\sqrt{T}
$$

Note: in my code this vega is **per +1.00 change in volatility** (i.e., $\sigma$ units).  
If you want vega **per 1 vol point (1%)**, you can divide it by $100$.

---

## Logic flow (what I do and why)

1. **I set the parameters $(K, r, \sigma)$**  
   I fix them because the dashboard is meant to isolate how greeks respond to only $S$ and $T$.

2. **I build a grid of $(S, T)$ values**  
   I create vectors for $S$ and $T$, then use `meshgrid` so I can compute everything in a fully vectorized way (fast and clean).

3. **I compute greeks on the entire grid**  
   I apply the closed-form formulas above to get arrays for price, delta, gamma, and vega for both calls and puts.

4. **I plot heatmaps**  
   Heatmaps are the quickest way to see where greeks concentrate (especially gamma and vega).

5. **I annotate the key region**  
   I explicitly mark the classic behavior:

   - **Gamma peaks near ATM and near expiry**
   - Delta transitions fastest around the same region

---

## Intuition: what you should see in the plots

### Delta ($\Delta$)
- **Call delta** goes from near $0$ (deep OTM) to near $1$ (deep ITM).
- **Put delta** goes from near $-1$ (deep ITM put) to near $0$ (deep OTM put).
- The transition is sharpest near **ATM** and gets sharper as $T$ approaches $0$.

### Gamma ($\Gamma$)
- Peaks around **ATM** because that’s where delta changes the fastest when spot moves.
- Becomes very large as **expiry approaches** (the option payoff becomes more “digital” in the limit).

### Vega ($V$)
- Highest around **ATM** because option value is most sensitive to volatility when intrinsic value is uncertain.
- Typically declines as $T \to 0$, since there is less time for volatility to matter.

---

## Dependencies

- `numpy`
- `matplotlib`
- `scipy`

I use `scipy.stats.norm` because it gives a reliable and standard implementation of $\Phi(\cdot)$ and $\phi(\cdot)$, which keeps the code short and avoids custom approximations.

---

## How to run

Clone the repo, install dependencies, and run the script:

```bash
pip install numpy matplotlib scipy
python greeks_dashboard.py
