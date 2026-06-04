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
To isolate player-specific prioritization, we filtered the raw dataset to include only individual player rows (`position != 'team'`), removing all team-level summary rows for the initial analysis. We converted `datacompleteness` to strings and fixed boolean-adjacent types to ensure accurate processing.

A critical part of the cleaning involved engineering the `gold_share` column. In the context of the data generating process, raw gold is often a "noisy" metric because it scales with game length and regional playstyles. By calculating a player's gold as a percentage of their team's total at the time of the game, we created a stable metric that purely reflects strategic priority rather than just game duration.

### Univariate Analysis
**Distribution of Total Gold Earned by All Players**: This histogram displays the wide distribution of total gold across all roles and match lengths. The long tail on the right represents players in "marathon" games who have accumulated massive wealth, illustrating the lack of a "gold cap" in the LoL ecosystem.

<iframe
  src="assets/dist_player_gold.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Distribution of Team Gold Share per Player**: This plot shows how team resources are partitioned. While a perfectly equal distribution would sit at 20% per player, the data is clearly centered slightly higher for carry roles, with peaks near roughly 24%, proving that teams systematically funnel more resources into specific members.

<iframe
  src="assets/dist_gold_share.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis
Comparing Gold Share by Role, we found that ADCs (Bot) maintain a higher median and tighter distribution of gold share compared to Mid Laners. This suggests that while Mid Laners are influential, the 2025 meta consistently prioritizes the ADC as the primary resource beneficiary.

**Comparison of Gold Share - Mid Laners vs. ADCs**: This box plot directly compares our two roles of interest. ADCs (Bot) show a higher median gold share and a tighter interquartile range compared to Mid Laners, suggesting that ADC resource priority is a more consistent "rule" in the 2025 meta.

<iframe
  src="assets/dist_gold_share_pos.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Gold Share by Role and Match Result**: By adding the match result as a factor, we see that winning teams tend to have slightly higher gold shares for their carries. This could indicate that successful "snowballing" allows carries to take an even larger slice of the team's economic pie.

<iframe
  src="assets/dist_gold_share_pos_result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

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

## Framing a Prediction Problem
- Prediction Problem: Can we predict whether a team will win or lose a match based on their early-game resource and objective advantages (up to the 15-minute mark)?
- Type: Binary Classification (Response: result)
- Metric: Accuracy. Professional matches are zero-sum; the binary win/loss is the ultimate objective, making raw accuracy the most intuitive measure of success
- Time of Prediction: All features are strictly limited to data available at or before 15 minutes to avoid data leakage

## Baseline Model
Our baseline utilizes a Logistic Regression model with two features: `golddiffat15` (Quantitative) and `side` (Nominal, One-Hot Encoded). The model achieved a Baseline Accuracy of 0.7289. While this provides a strong foundation, it relies purely on a single snapshot of gold and ignores the strategic pressure of map objectives.

## Final Model
### Feature Engineering and the DGP
To capture the complexity of the 2025 meta, we engineered two original features:
- **Lead Momentum**: Calculated as the growth in gold lead between 10 and 15 minutes. This captures the velocity of the game—whether a team is actively snowballing or their lead is stalling.
- **Objective Density**: A sum of Dragons and Void Grubs secured by 15 minutes. This reflects how effectively a team translates their gold leads into permanent, structural map advantages.

### Modeling and Performance
We implemented a **RandomForestClassifier** and used **GridSearchCV** to tune `max_depth`, `n_estimators`, `min_samples_split`, and `max_features`. Our final model reached an accuracy of **0.7610**, a clear improvement over the baseline. This demonstrates that game trajectory (Momentum) and structural pressure (Density) are more predictive than a simple gold snapshot.

## Fairness Analysis
We investigated if our model was biased toward either the Blue Side or Red Side.
- Metric: Precision (Predictive reliability of a "Win")
- Null Hypothesis: The model is fair; precision for Blue and Red sides are roughly equal
- Result: The observed difference was a negligible 0.0036 with a p-value of 0.8330
- Conclusion: We fail to reject the null hypothesis. Our model is statistically fair and provides equally reliable predictions regardless of which side of the map a team starts on.

**Null Distribution of Absolute Precision Differences**: This histogram shows the empirical distribution of differences in precision generated under the null hypothesis. Our observed difference of 0.0036 (indicated by the red dashed line) sits deep within the bulk of the distribution, visually confirming that such a difference is extremely common due to random chance. We fail to reject the null hypothesis, concluding the model is statistically fair.

<iframe
  src="assets/fairness_permutation_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
