# League of Legends Gold Share Analysis

League of Legends Gold Share Analysis is a data science project delivered as a final for DSC 80 at the University of California, San Diego. The project aims to investigate resource prioritization and match predictibility within the 2025 League of Legends professional meta. By analyzing over thousands of matches within that year, this study explores how in-game gold is shared between Mid and Bot lane roles, assesses the impact of early-game objectives, and develops a predictive model to determine match results at the 15-minute mark.

Author: Daniel Vinh Vuong

## Introduction
### Background
League of Legends (LoL), often referred to simply as League, is a popular team-based multiplayer online battle arena (MOBA) game developed by Riot Games, an American video game publisher. Set primarily on the competitive map "Summoner's Rift", two teams of five players compete in a match to destroy the opposing team's "Nexus", a structure located within their base. Destroying the Nexus is the main objective and results in a victory for the team that does so first. Success in professional play is largely driven by accumulating resources such as gold and securing objectives like Dragons and Void Grubs, which provide advantages for a team and help players purchase items that increase their power throughout the match.

The distribution of these resources is highly strategic in professional play. Teams must decide how to distribute limited resources among their players to ensure their chances of winning. This project investigates the percentage of a team's total gold allocated to a specific player (which will be referred to as "gold share") in order to understand which roles in the game are prioritized as "carries". As such, the central research question for this project is: 

> Do Mid Laners or ADCs (Bot Lane) typically earn a higher share of the team's total gold in the 2025 professional meta?

### Dataset Overview
The dataset used for this analysis is retrieved from Oracle's Elixir, a highly reputable source for advanced League of Legends esports data since 2015. Using recorded professional match data from 2025, it contains 120,456 rows and 165 columns of important match data that is useful for understanding player behavior, team dynamics, and match results. Of those 165 columns, the following are the ones relevant to our research question and analysis at hand:

- `gold_share`: The percentage of a team's total gold earned by an individual player.
- `golddiffat15`: The total gold difference between teams at the 15-minute mark.
- `elementaldrakes`: The count of Elemental Dragons secured by a team
- `void_grubs`: The count of Void Grubs (early structural objectives) secured.
- `result`: The binary outcome of the match (1 for a win, 0 for a loss).
- `side`: The map side the team started on (Blue or Red).

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
To prepare the dataset for analysis, several cleaning steps were necessary. First, the raw dataset contains 12 rows per game: one for each of the 5 players on both teams, plus 2 team-level summary rows. Since our research question focuses on individual role prioritization, we filtered to keep only player rows by removing all rows where `position == 'team'`. We also converted `datacompleteness` to string type to avoid type-related issues during groupby operations.

The most critical step was creating the `gold_share` column. Raw gold totals are noisy as they scale with game length and regional playstyle. This means a player in a 40-minute game will naturally have more gold than one in a 25-minute game regardless of how prioritized they were. By calculating each player's gold as a percentage of their team's total gold at end of game, we created a normalized metric that purely reflects strategic priority independent of game duration. After cleaning, we filtered to only Mid and Bot lane rows for our core analysis.

### Univariate Analysis

<iframe
  src="assets/dist_player_gold.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of total gold earned across all players is roughly bell-shaped and centered around 12,000–15,000 gold, with a long right tail representing players in extended games who accumulate significantly more wealth. This confirms that raw gold is a poor basis for role comparison, as game length introduces substantial variance, which is exactly why we engineered the `gold_share` metric instead.

<iframe
  src="assets/dist_gold_share.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of gold share per player shows two clear peaks: one around 13–14% representing Support players, and a larger cluster around 20–24% representing the four carry roles. This bimodal shape reflects the well-known Support tax in League of Legends, where one player deliberately sacrifices personal resources to empower teammates. The carry cluster sitting above the 20% equal-split baseline confirms that teams systematically funnel extra resources into specific roles.

### Bivariate Analysis
Comparing Gold Share by Role, we found that ADCs (Bot) maintain a higher median and tighter distribution of gold share compared to Mid Laners. This suggests that while Mid Laners are influential, the 2025 meta consistently prioritizes the ADC as the primary resource beneficiary.

<iframe
  src="assets/dist_gold_share_pos.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Comparing gold share between Mid Laners and ADCs reveals a consistent edge for the Bot lane role. ADCs maintain a higher median gold share and a notably tighter interquartile range, meaning not only do they receive more resources on average, but this prioritization is more consistent across teams and regions. Mid Laners show more variance: some teams heavily invest in their Mid Laner while others treat the role as a secondary carry. This suggests that while Mid Lane carry potential exists in the 2025 meta, ADC resource priority is closer to a universal rule than a team-specific choice.

<iframe
  src="assets/dist_gold_share_pos_result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

When adding match result as a factor, winning teams show slightly higher gold shares for their carries compared to losing teams. This is consistent with a snowball effect: teams that are winning can afford to funnel even more resources into their carries, amplifying the lead. The pattern holds for both roles, but is more pronounced for ADCs.

### Interesting Aggregates
Grouping by position highlights the mean gold share for the 2025 season:
- Bot: 23.66%
- Mid: 22.10%
- Top: 20.88%
- Jungle: 19.78%
- Support: 13.57%

## Assessment of Missingness
### NMAR Analysis
We believe the column golddiffat15 is NMAR (Not Missing At Random). In the DGP, a gold difference at 15 minutes cannot be recorded if a match ends before that mark (e.g., an early game forfeit). Because the missingness is inherently tied to the value of the gamelength itself, it is NMAR. To move this toward MAR, we would need the gamelength column to explain why these values are absent.

### Missingness Dependency
Permutation tests (using 500 iterations and TVD as the test statistic) revealed that the missingness of golddiffat15 is dependent on the league (p-value: 0.0) but independent of map side (p-value: 1.0). This indicates regional differences in data reporting standards rather than competitive map biases.

**Distribution of Leagues by Missingness of 'golddiffat15'**: This histogram visualizes the proportion of missing data across different leagues. Some regional leagues show a significantly higher proportion of missing 15-minute stats than others, confirming that the missingness is not uniform and depends on the specific tournament's data reporting standards.

<iframe
  src="assets/dist_league_golddiffat15_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Hypothesis Testing
- **Null Hypothesis (H<sub>0</sub>)**: Mid Laners and ADCs earn an equal average share of the team's total gold.
- **Alternative Hypothesis (H<sub>1</sub>)**: ADCs earn a higher average share of team gold than Mid Laners.
- **Test Statistic**: Difference in means (Mean ADC gold share - Mean Mid gold share).
- **Result**: With an observed difference of 0.0156 and a p-value of 0.0, we reject the null hypothesis. There is a statistically significant resource prioritization for ADCs in the 2025 pro meta.

With a p-value of 0.0 and an observed difference of ~0.0156, we reject the null hypothesis. The data provides overwhelming evidence that ADCs earn a statistically significantly higher share of team gold than Mid Laners in the 2025 professional meta. This is not a marginal difference: 1.56 percentage points across thousands of professional games represents a deliberate and consistent strategic choice by teams worldwide. The 2025 meta has firmly established the ADC as the primary resource beneficiary, and this finding directly informs how we approach our predictive model: gold advantage is not distributed equally, and understanding where it flows matters.

## Framing a Prediction Problem
- Prediction Problem: Can we predict whether a team will win or lose a match based on their early-game resource and objective advantages (up to the 15-minute mark)?
- Type: Binary Classification (Response: result)
- Metric: Accuracy. Professional matches are zero-sum; the binary win/loss is the ultimate objective, making raw accuracy the most intuitive measure of success
- Time of Prediction: All features are strictly limited to data available at or before 15 minutes to avoid data leakage

## Baseline Model
Our baseline model uses Logistic Regression with two features: `golddiffat15` (quantitative) and `side` (nominal, one-hot encoded). This achieved a baseline accuracy of approximately **72.9%**. While this is a reasonable starting point, a limitation in this model is how it treats gold advantage as a static snapshot. For example, a team that is 2,000 gold ahead at 15 minutes but was 3,000 ahead at 10 minutes is actually losing momentum, while a team that clawed from -500 to +1,500 is on an upswing. Both look identical to this model. Additionally, the baseline does not take into account objective control. Dragons and Void Grubs, for example, provide permanent structural advantages that persist regardless of gold swings. A model blind to these factors is missing critical information about the true state of the game at 15 minutes.

## Final Model
### Feature Engineering and the DGP
To address the baseline's limitations, we engineered two new features grounded in how professional League of Legends is actually played.

One such feature we engineered is **Lead Momentum**, which attempts to capture the velocity of the gold lead between 10 and 15 minutes. More specifically, it captures how much the gold differential grew or shrank in that window. A team snowballing their lead is in a fundamentally different position than a team whose lead is stalling or being closed. This feature captures that trajectory in a way a static 15-minute snapshot cannot.

The other, **Objective Density**, attempts to sum the total Dragons and Void Grubs secured by a team by 15 minutes. These objectives provide permanent buffs that compound over time. For example, a team with 2 dragons and 3 grubs has structural advantages that will impact the rest of the game regardless of gold swings. By combining these into a single density metric, we capture how effectively a team converts early momentum into durable map control.

### Modeling and Performance
We also retained `golddiffat15` and `side` from the baseline, and applied a **StandardScaler** to all numeric features to ensure no single variable dominates the model. Using a **RandomForestClassifier** with **GridSearchCV** tuning across `max_depth`, `n_estimators`, `min_samples_split`, and `max_features,` our best model used `max_depth=5`, `max_features='sqrt'`, `min_samples_split=9`, and `n_estimators=6`. (CHANGE)

The final model achieved an accuracy of **76.21%**, an improvement of approximately 3.31 percentage points over the baseline. This improvement demonstrates that game trajectory and objective control carry genuine predictive signal beyond a gold snapshot alone, which is consistent with how analysts and coaches would evaluate team performance in professional play.

## Fairness Analysis
We investigated if our model was biased toward either the Blue Side or Red Side.
- Metric: Precision (Predictive reliability of a "Win")
- Null Hypothesis: The model is fair; precision for Blue and Red sides are roughly equal
- Result: The observed difference was a negligible 0.0134 with a p-value of 0.4560
- Conclusion: We fail to reject the null hypothesis. Our model is statistically fair and provides equally reliable predictions regardless of which side of the map a team starts on.

With an observed precision difference of only 0.0134 and a p-value of 0.4560, we fail to reject the null hypothesis. Our model produces equally reliable win predictions for both Blue and Red side teams, despite the well-known asymmetries in map design that give each side different objective access. This result means our model has learned generalizable patterns about early-game advantages rather than overfit to side-specific tendencies.

<iframe
  src="assets/fairness_permutation_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Null Distribution of Absolute Precision Differences**: This histogram shows the empirical distribution of differences in precision generated under the null hypothesis. Our observed difference of 0.0036 (indicated by the red dashed line) sits deep within the bulk of the distribution, visually confirming that such a difference is extremely common due to random chance. We fail to reject the null hypothesis, concluding the model is statistically fair.
