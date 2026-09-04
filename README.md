# ⚽ Hypothesis Testing: Are More Goals Scored in Women's Than Men's International Soccer Matches?

**A statistical study of FIFA World Cup matches (2002-present)**

---

## 📌 Background

As a sports journalist who has followed men's and women's international soccer for years, I had a gut feeling that **women's international soccer matches score more goals** than men's. Rather than publishing that as an opinion, this hypothesis is tested statistically so it can be turned into a data-driven investigative article.

Since the sport has changed a lot over time and performance varies significantly across tournaments, the analysis is limited to:
- Official **FIFA World Cup** matches (excluding qualifiers)
- Matches since **2002-01-01**

## ❓ Business Question

> Is the average number of goals scored in women's international soccer matches higher than in men's?

## 🧪 Hypotheses

| | |
|---|---|
| **H₀ (Null)** | The mean number of goals scored in women's matches is **the same as** men's |
| **H₁ (Alternative)** | The mean number of goals scored in women's matches is **greater than** men's |
| **Significance level (α)** | 10% (0.1) - one-tailed test |

## 🗂️ Data

Two historical datasets of international match results dating back to the 19th century:
- `men_results.csv`
- `women_results.csv`

Key columns: `date`, `home_team`, `away_team`, `home_score`, `away_score`, `tournament`.

**Filters applied:**
```python
men_subset = men[(men["date"] > "2002-01-01") & (men["tournament"] == "FIFA World Cup")]
women_subset = women[(women["date"] > "2002-01-01") & (women["tournament"] == "FIFA World Cup")]
```

New column created: `goals_scored = home_score + away_score` (total goals per match).

## 🔍 Methodology

### 1. Normality Check (Exploratory Analysis)

Before choosing a statistical test, the distribution of `goals_scored` for each group was inspected using histograms:

**Goals distribution - Men's World Cup (2002–present)**
Goals per match range from 0–8, with most matches concentrated around 1–3 goals. The distribution is **right-skewed**, not a symmetric bell shape.

**Goals distribution - Women's World Cup (2002–present)**
Goals per match span a much wider range, 0–13, with a long right tail including matches with 10-13 goals. Also **right-skewed**, but with greater spread and a noticeably higher maximum than men's.

➡️ **Early observation:** the maximum goals per match in women's matches (up to 13) is far higher than in men's (up to ~8) - an early visual signal consistent with the initial hypothesis, though this is only a visual observation, not statistical proof yet.

**Normality conclusion:** both distributions are **not normal** → a t-test would violate the normality assumption. A non-parametric test is used instead.

### 2. Choice of Statistical Test

Since the data is not normally distributed, the **Wilcoxon-Mann-Whitney U Test** (a non-parametric test for comparing two independent groups) is used, with a one-tailed direction (`alternative="greater"`) matching H₁.

## 💻 Code

```python
import pandas as pd
import matplotlib.pyplot as plt
import pingouin
from scipy.stats import mannwhitneyu

# 1. Load data
men = pd.read_csv("men_results.csv")
women = pd.read_csv("women_results.csv")

# 2. Filter: FIFA World Cup, since 2002-01-01
men["date"] = pd.to_datetime(men["date"])
men_subset = men[(men["date"] > "2002-01-01") & (men["tournament"] == "FIFA World Cup")].copy()

women["date"] = pd.to_datetime(women["date"])
women_subset = women[(women["date"] > "2002-01-01") & (women["tournament"] == "FIFA World Cup")].copy()

# 3. Feature engineering
men_subset["group"] = "men"
women_subset["group"] = "women"
men_subset["goals_scored"] = men_subset["home_score"] + men_subset["away_score"]
women_subset["goals_scored"] = women_subset["home_score"] + women_subset["away_score"]

# 4. Normality check (visual)
men_subset["goals_scored"].hist()
plt.title("Men's World Cup — Goals Scored")
plt.show()
plt.clf()

women_subset["goals_scored"].hist()
plt.title("Women's World Cup — Goals Scored")
plt.show()
plt.clf()

# 5. Combine data for the statistical test
both = pd.concat([women_subset, men_subset], axis=0, ignore_index=True)
both_subset = both[["goals_scored", "group"]]
both_subset_wide = both_subset.pivot(columns="group", values="goals_scored")

# 6. Wilcoxon-Mann-Whitney U Test (one-tailed, "greater")
results_pg = pingouin.mwu(
    x=both_subset_wide["women"],
    y=both_subset_wide["men"],
    alternative="greater"
)

# Cross-check with scipy
results_scipy = mannwhitneyu(
    x=women_subset["goals_scored"],
    y=men_subset["goals_scored"],
    alternative="greater"
)

# 7. Extract p-value & determine the decision
p_val = results_pg["p-val"].values[0]
result = "reject" if p_val <= 0.1 else "fail to reject"

result_dict = {"p_val": p_val, "result": result}
print(result_dict)
```

## 📊 Results
<table align="center">
  <tr>
    <td align="center">
      <h3>Goal Distribution Men</h3>
      <img src="goal%20distribution_men.png" width="400">
    </td>
    <td align="center">
      <h3>Goal Distribution Women</h3>
      <img src="goal%20distribution_women.png" width="400">
    </td>
  </tr>
</table>

The result from our test:
| Metric | Value |
|---|---|
| p-value | `0.0051` |
| α (significance level) | 0.10 |
| Decision | `reject` |

### How to read the result
- **If p-value ≤ 0.10** → reject H₀. Statistically, the mean number of goals in women's matches is **significantly higher** than men's at the FIFA World Cup since 2002.
- **If p-value > 0.10** → fail to reject H₀. There isn't enough statistical evidence to claim women's matches score more — the difference could just be random variation.

## 💡 Insight & Business Interpretation

*(p-value below assume the result is "reject H₀"):*

1. **Key finding:** There is fairly strong statistical evidence (at α = 10%) that women's FIFA World Cup matches score more goals than men's since 2002.
2. **Possible drivers (need further data to confirm):** defensive quality that hasn't yet matched men's game at the international level, a more open/attacking style of play, or a larger competitive gap between teams in the women's game - leading to more frequent blow-out scorelines.
3. **Value for the newsroom:** the "women's soccer is statistically more entertaining" angle now has data behind it rather than just opinion - a solid basis for a data-journalism piece, supported by the histograms and test result as evidence.
4. **Limitations:** the analysis is limited to one tournament (FIFA World Cup) and one time window (2002-present) - results may not generalize to other competitions (domestic leagues, qualifiers, etc.).

## 🛠️ Tools & Skills Used
- **Python:** `pandas` (data wrangling), `matplotlib` (EDA/visualization), `pingouin` & `scipy.stats` (non-parametric hypothesis testing)
- **Statistics:** visual normality checks, Wilcoxon-Mann-Whitney U Test, one-tailed hypothesis testing
- **Business framing:** translating a statistical result into an insight a newsroom/decision-maker can actually use

## 📁 Project Structure
```
├── men_results.csv
├── women_results.csv
├── hypothesis_testing_soccer.ipynb   # analysis notebook
└── README.md                         # this documentation
```
