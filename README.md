# League of Legends Gold Share Analysis

League of Legends Gold Share Analysis is a data science project completed as a final for DSC 80 at the University of California, San Diego. The project aims to investigate resource prioritization and match predictibility within the 2025 League of Legends professional meta. Through exploratory data analysis, statistical hypothesis testing, and predictive modeling, the focus is to examine how gold share differs between Mid Laners and ADCs (Bot Lane), assess what early-game advantages are most predictive of winning, and construct a classifier that predicts match outcomes using only information available at the 15-minute mark.

Author: Daniel Vinh Vuong

## Introduction
### Background
League of Legends (LoL), often referred to simply as League, is a popular team-based multiplayer online battle arena (MOBA) game developed by Riot Games, an American video game publisher. Set primarily on the competitive map "Summoner's Rift", two teams of five players compete in a match to destroy the opposing team's "Nexus", a structure located within their base. Destroying the Nexus is the main objective and results in a victory for the team that does so first. Success in professional play is largely driven by accumulating resources such as gold and securing objectives like Dragons and Void Grubs, which provide advantages for a team and help players purchase items that increase their power throughout the match.

Gold is one of the most critical resources in the game. It is the in-game currency used to purchase items that make players stronger. Though earned passively over time, it is accelerated by killing minions, defeating objectives, and eliminating enemies. Because no two players earn the same amount of gold, every team implicitly makes a strategic decision about who receives priority resources. This project investigates that decision using data from the 2025 professional League of Legends season. Specifically, our question is: 

> Do Mid Laners or ADCs (Bot Lane) typically earn a higher share of the team's total gold in the 2025 professional meta?

This question matters because resource allocation is one of the most consequential choices a coaching staff can make. Funneling gold into the wrong role or failing to identify which role the meta rewards can cost a team championships. By analyzing gold share across thousands of professional games, we can identify whether the 2025 meta has a clear carry priority, and what that means for how teams should draft and play.

### Dataset Overview
The dataset used for this analysis is retrieved from Oracle's Elixir, a highly reputable source for advanced League of Legends esports data since 2015. Using recorded professional match data from 2025, it contains **120,456 rows**, with 12 rows per game (one per player, plus two team-summary rows). Of the 165 columns, the following are the ones most relevant to our research question and analysis at hand:

- `position`: The role of the player (top, jng, mid, bot, sup) or 'team' for team-level rows.
- `side`: Blue or Red side assignment.
- `totalgold`: The total gold earned by the player at the end of the game.
- `gold_share`: The percentage of a team's total gold earned by an individual player.
- `golddiffat15`: The total gold difference between teams at the 15-minute mark.
- `elementaldrakes`: The count of Elemental Dragons secured by a team
- `void_grubs`: The count of Void Grubs (early structural objectives) secured.
- `result`: The binary outcome of the match (1 for a win, 0 for a loss).
- `side`: The map side the team started on (Blue or Red).

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
To prepare the dataset for analysis, several cleaning steps were necessary. First, the raw dataset contains 12 rows per game: one for each of the 5 players on both teams, plus 2 team-level summary rows. Since our research question focuses on individual role prioritization, we filtered to keep only player rows by removing all rows where `position == 'team'`. We also converted `datacompleteness` to string type to avoid type-related issues during groupby operations.

The most important step was creating the `gold_share` column. Raw gold totals are noisy as they scale with game length and regional playstyle. This means a player in a 40-minute game will naturally have more gold than one in a 25-minute game regardless of how prioritized they were. By calculating each player's gold as a percentage of their team's total gold at end of game, we created a normalized metric that purely reflects strategic priority independent of game duration. After cleaning, we filtered to only Mid and Bot lane rows for our core analysis.

|    | gameid           | side   | position   |   totalgold |   gold_share |
|---:|:-----------------|:-------|:-----------|------------:|-------------:|
|  2 | LOLTMNT03_179647 | Blue   | mid        |        9032 |     0.21375  |
|  3 | LOLTMNT03_179647 | Blue   | bot        |        9407 |     0.222625 |
|  7 | LOLTMNT03_179647 | Red    | mid        |       11477 |     0.212789 |
|  8 | LOLTMNT03_179647 | Red    | bot        |       13632 |     0.252744 |
| 14 | LOLTMNT06_96134  | Blue   | mid        |       15931 |     0.246347 |

### Univariate Analysis
We first examined the distribution of raw gold earned across all players. This gives us a baseline understanding of how gold is distributed before normalizing.

<iframe
  src="assets/dist_player_gold.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of total gold earned across all players is roughly bell-shaped and centered around 12,000–15,000 gold, with a long right tail representing players in longer games who accumulate significantly more wealth. This confirms that raw gold is a poor basis for role comparison, as game length introduces substantial variance. A player with 18,000 gold in a 40-minute game may have actually been deprioritized relative to a player with 14,000 gold in a 25-minute game. This is exactly why we engineered the `gold_share` metric instead.

<iframe
  src="assets/dist_gold_share.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of gold share per player reveals a clear bimodal structure. There is a small cluster around 13–14% representing Support players, who deliberately sacrifice personal resources to empower teammates. The much larger cluster between 19–26% represents the four carry roles. Notably, the carry cluster sits above the theoretical 20% equal-split baseline, confirming that teams intentionally funnel extra gold into specific roles, and that this prioritization is consistent across the entire dataset.

### Bivariate Analysis
We then compared gold share specifically between Mid Laners and ADCs (Bot Lane) to identify which role receives more resources.

<iframe
  src="assets/dist_gold_share_pos.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

ADCs show a consistently higher median gold share than Mid Laners. The ADC distribution also has a tighter interquartile range, meaning this prioritization is more consistent across teams and regions. It is closer to a universal rule than a team-specific choice. Mid Laners show more variance, suggesting that while some teams heavily invest in their Mid Laner, others treat it as a secondary carry role. The 2025 meta appears to heavily favor the ADC as the primary gold beneficiary.

<iframe
  src="assets/dist_gold_share_pos_result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Splitting by match result, we observe an interesting pattern. Winning teams show slightly higher gold shares for their ADCs compared to losing teams, while the difference for Mid Laners is less pronounced. This is consistent with a snowball effect, in which teams that are winning can afford to funnel even more gold into their primary carry, amplifying the lead. The fact that this effect is stronger for ADCs than Mid Laners further supports the idea that the ADC is the meta's designated carry role.

### Interesting Aggregates
The table below shows the mean and median gold share broken down by all five positions:

| position   |     mean |   median |        std |
|:-----------|---------:|---------:|-----------:|
| bot        | 0.236631 | 0.235945 | 0.0209985  |
| jng        | 0.197836 | 0.196349 | 0.0187563  |
| mid        | 0.221045 | 0.220304 | 0.0199085  |
| sup        | 0.135679 | 0.134532 | 0.00982972 |
| top        | 0.208809 | 0.20769  | 0.0211867  |

It appears that bot lane leads all roles with a mean gold share of 23.7%, more than 1.5 percentage points above Mid at 22.1%. Support has by far the lowest share at 13.6% and also the lowest standard deviation, reflecting how that role consistently sacrifices its gold. Jungle and Top cluster together in the 19–21% range. The gap between Bot and Mid, while seemingly small, is highly consistent across thousands of games, which our hypothesis test will confirm.

## Assessment of Missingness
### NMAR Analysis
The column `golddiffat15` is likely **NMAR (Not Missing At Random)**. When a match ends before the 15-minute mark (due to an early forfeit, for example) there is no 15-minute gold difference to record. The missingness is directly caused by the missing value itself being impossible to observe: a game that ended at 12 minutes never had a 15-minute state. This makes it NMAR rather than MAR, because the reason for missingness is tied to the unobserved value.

To make this MAR, we would need additional data such as `gamelength`. If we knew a game ended at 12 minutes, we could explain the missing `golddiffat15` without needing to know what the value would have been. With that column available, missingness would be explainable by an observed variable, thus satisfying the MAR condition.

### Missingness Dependency
We ran two permutation tests to determine whether the missingness of `golddiffat15` depends on other columns. For both tests, we used **Total Variation Distance (TVD)** as the test statistic and a significance level of **0.05**.

**Test 1: Does missingness of `golddiffat15` depend on `league`?**
We shuffled the missingness labels 500 times and compared the TVD of each shuffle to the observed TVD. The result was a p-value of **0.0**, meaning none of the 500 permutations produced a TVD as extreme as what we observed. We reject the null hypothesis: the missingness of `golddiffat15` does depend on `league`. This makes sense as certain leagues (like LPL) have incomplete data coverage, so the missingness is systematically associated with which tournament the match was played in.

<iframe
  src="assets/dist_league_golddiffat15_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Test 2: Does missingness of `golddiffat15` depend on `side`?**
We ran the same permutation test using `side` instead of `league`. The result was a p-value of **1.0**, meaning the observed TVD was essentially zero. Thus, we fail to reject the null hypothesis: the missingness of `golddiffat15` does not depend on `side`. This makes intuitive sense as both Blue and Red side teams play in the same games, so if one side's data is missing, the other's is too. The missingness is therefore symmetric across sides.

## Hypothesis Testing
Having observed a difference in gold share between ADCs and Mid Laners in Step 2, we now formally test whether this difference is statistically significant or could be explained by random chance.

- **Null Hypothesis (H<sub>0</sub>)**: Mid Laners and ADCs earn an equal average share of the team's total gold. Any observed difference is due to random chance.
- **Alternative Hypothesis (H<sub>1</sub>)**: ADCs earn a higher average share of team gold than Mid Laners.
- **Test Statistic**: Difference in means (Mean ADC gold share - Mean Mid gold share).
- **Significance Level**: 0.05

We chose a one-sided test because we have a directional hypothesis: based on the EDA, ADCs appear to be prioritized. The permutation test works by shuffling position labels 1,000 times and computing the difference in means each time, building a null distribution of what the difference would look like if position had no effect on gold share.

**Observed Difference**: 0.01559 | **P-value**: 0.0

Since p < 0.05, we **reject the null hypothesis**. With an observed difference of around 1.56 percentage points and a p-value of 0.0 across 1,000 permutations, the data provides overwhelming evidence that ADCs systematically receive more gold than Mid Laners in the 2025 professional meta. This is not noise, but rather reflets a deliberate and consistent strategic choice made by professional teams worldwide. This finding directly motivates our prediction model: gold advantages are not distributed equally across roles, and understanding where resources flow is key to predicting match outcomes.

## Framing a Prediction Problem
- **Prediction Problem**: Can we predict whether a team will win or lose a match based on their early-game resource and objective advantages (up to the 15-minute mark)?
- **Type**: Binary Classification (the response variable `result` takes value 1 (win) or 0 (loss), with no ties possible in League of Legends).
- **Response Variable**: We chose `result` as the response variable because it is the only outcome that matters. Every strategic decision in the game is ultimately made in service of winning.
- **Metric**: Accuracy is chosen as our metric because professional matches are zero-sum, making raw accuracy the most intuitive measure of success.
- **Time of Prediction**: All features are strictly limited to data available at or before 15 minutes to avoid data leakage.

The 15-minute mark is chosen because it a natural checkpoint in professional play. It is when teams can first formally surrender, when most early objectives have been contested, and when draft advantages begin to manifest in gold and map control. Predicting outcomes at this point tests whether early advantages are truly meaningful, not just whether a team that is already winning will continue to win.

All features used in the model (gold differences, objective counts, side) are observable at or before 15 minutes, so there is no data leakage.

Note: The dataset used from this point forward differs from the one used in the hypothesis test. While the hypothesis test examined individual player rows filtered to Mid and Bot lane, the prediction model operates on team-level summary rows (`position == 'team'`), since we are predicting the outcome of the match as a whole rather than individual player behavior.

## Baseline Model
The baseline model uses **Logistic Regression** with two features: `golddiffat15` (quantitative) and `side` (nominal). We applied `OneHotEncoder` to convert `side` into a binary numeric format, and used `train_test_split` with 75/25 to evaluate generalization to unseen data.

**Features:**
- Quantitative: 1 (`golddiffat15`)
- Nominal: 1 (`side`, one-hot encoded)
- Ordinal: 0

**Baseline Accuracy**: **~73.5%**

While 73.5% is a reasonable starting point, this model has a fundamental limitation. Essentially, it treats the game state as a static snapshot. For example, a team that is 2,000 gold ahead at 15 minutes but was 3,000 ahead at 10 minutes is actually losing momentum, while a team that clawed from -500 to +1,500 is on an upswing. Both look identical to this model. Additionally, the baseline completely ignores objective control. Dragons and Void Grubs, for example, grant permanent map pressure and structural advantages that persist for the rest of the game regardless of gold fluctuations. A model blind to these factors is missing critical information about the true state of the game at 15 minutes, which motivates the feature engineering in our final model.

## Final Model
### Feature Engineering
To address the baseline's limitations, we engineered two new features rooted in how professional League of Legends is actually played.

The first is **Lead Momentum**, derived from the columns `goldat10`, `opp_goldat10`, `goldat15`, and `opp_goldat15`, and calculated as the change in gold differential between 10 and 15 minutes, specifically `(goldat15 - opp_goldat15) - (goldat10 - opp_goldat10)`. Rather than treating the game state as a static snapshot, this feature captures the velocity of the gold lead. A team that is 2,000 gold ahead but was 3,500 ahead five minutes ago is in a very different position than a team that just swung from -500 to +1,500, yet both would look similar to the baseline model. Lead Momentum distinguishes these scenarios by measuring whether a lead is actively snowballing or being clawed back.

The second is **Objective Density**, derived from the columns `elementaldrakes` and `void_grubs`, and calculated as their sum at the 15-minute mark. These objectives provide permanent buffs that compound over time: dragon stacks grant elemental bonuses that grow in impact, while Void Grubs accelerate structural damage to the enemy base. By combining them into a single density metric, we capture how effectively a team has converted early map pressure into durable advantages that persist regardless of future gold swings.

We retained `golddiffat15` and `side` from the baseline to preserve the signal we know works, and applied `StandardScaler` to all numeric features to ensure no single variable dominates the model.

### Modeling and Hyperparameter Tuning
We chose a `RandomForestClassifier` because it can capture non-linear interactions between features. For example, the combination of high momentum and high objective density is likely more predictive than either feature alone, which a linear model like Logistic Regression cannot represent.

We tuned four hyperparameters using **GridSearchCV with 5-fold cross validation**. For `n_estimators`, we searched across 100, 200, and 300 trees, with 300 selected (more trees produce a more stable ensemble with lower variance). `For max_depth`, we searched 5, 10, 20, and None, with 10 selected, which is deep enough to capture non-linear interactions between features, but constrained enough to avoid overfitting to specific outlier games. For `min_samples_split`, we searched 7 and 9, with 7 selected; this acts as a regularization tool, forcing the model to learn broader trends rather than memorizing individual games. For `max_features`, we searched sqrt and log2, with sqrt selected; this ensures each tree considers a diverse subset of features at each split, preventing the model from over-relying on `golddiffat15` at the expense of the engineered features.

**Final Model Accuracy**: **76.21%**

Test accuracy improved from 73.5% in the baseline to 76.21% in the final model, demonstrating that game trajectory and objective control carry genuine predictive signal beyond a simple gold snapshot alone.

## Fairness Analysis
We investigated whether our model produces equally reliable predictions for Blue Side and Red Side teams, given that map asymmetries give each side different objective access.

- **Metric**: Precision for predicting a win.
- **Null Hypothesis (H<sub>0</sub>)**: The model is fair; precision for Blue and Red sides are roughly equal, and any difference is due to random chance.
- **Alternative Hypothesis (H<sub>1</sub>)**: The model is unfair; precision differs between Blue and Red sides.
- **Test Statistic**: Absolute difference in precision
- **Significance Level**: 0.05

**Observed Difference**: 0.0134 | **P-value**: 0.4420

Since p = 0.442 > 0.05, we **fail to reject the null hypothesis**. The observed precision difference falls well within the range of what we would expect by chance. We conclude that the model produces equally reliable win predictions for both Blue and Red side teams, despite the well-known map asymmetries. This suggests the model has learned generalizable patterns about early-game advantages rather than overfit to side-specific tendencies.

<iframe
  src="assets/fairness_permutation_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Null Distribution of Absolute Precision Differences**: This histogram shows the empirical distribution of differences in precision generated under the null hypothesis. Our observed difference of 0.0036 (indicated by the red dashed line) sits deep within the bulk of the distribution, visually confirming that such a difference is extremely common due to random chance. We fail to reject the null hypothesis, concluding the model is statistically fair.
