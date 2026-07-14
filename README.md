# Fantasy Football Predictions

Predict fantasy points for the upcoming NFL week during the regular season, defaulted to half-PPR scoring for running back, quarterback, wide receiver, and tight end.

---

## The Problem

Predicting fantasy football scores week in and week out is extremely difficult due to the high touchdown variance amongst all positions. While yardage can be easily predicted based on prior data, the touchdown variance is what swings a player's outcome from being a poor game to an average game, and an average game to a great game in terms of fantasy outlook. Touchdowns are a binary, random event that no pre-game feature can reliably predict, making a portion of weekly variance structurally unpredictable regardless of model sophistication.

---

## Architecture

The pipeline for predicting fantasy scores per position is broken out into 4 classes:

- **CreateFeatures:** Takes the raw dataset for a specific position and scoring target, and engineers features designed to help the model find signals within the data. The base features include a 17-week exponentially weighted rolling average. Other base features include a rolling average for how consistent a player is having above-average games, and flags for if a player is returning from injury or is a rookie during the season in which stats are recorded.

- **FeatureEngineer:** Takes the readied dataset passed through from the CreateFeatures class and displays correlations between the features and the target, collinearity between the features, executes pruning of features with low signal, and creates additional features based on recent (rolling prior 3-week average) and momentum (difference between 17-week and 3-week rolling averages) for select features.

- **ModelPipeline:** Takes the data from the FeatureEngineer class and prepares it for model training. Includes filtering the dataset to target-present rows, the most recent 51 weeks from multiple seasons, and splitting the dataset into training and testing subsets. Once filtered and split, the class conducts training using a hyperparameter tuned Random Forest Regressor model. Upon training, the model outputs the MAE and R² test scores.

- **Inference:** Takes the training data and the best model from hyperparameter tuning and makes predictions per player for the upcoming week based on the most recent week's features. Prior week predictions are merged to the final output prior to exporting the predictions to Excel.

---

## Data

The data used for making predictions comes from [nflreadpy](https://github.com/nflverse/nflreadpy), an open-source Python library containing actual in-game statistics over multiple seasons. For this project, the datasets used include player stats, team stats, player information (draft year), and Next Gen Stats.

The initial training data included information from the 2022 to 2025 regular seasons. The 2022 season was merely used for ensuring features for the beginning weeks of the 51-week training window included enough information for an accurate 17-week rolling average.

Because this was a regression problem, the data was not shuffled prior to splitting, as temporal ordering needed to remain intact for feature generation — shuffling would constitute data leakage given that rolling features depend on chronological row order. The 51-week window for training was chosen based on the variability in which players had bye weeks or missed games.

---

## Results

| Position | MAE  | R²   |
|:---------|-----:|-----:|
| RB       | 3.91 | 0.39 |
| QB       | 5.62 | 0.40 |
| WR       | 3.43 | 0.33 |
| TE       | 2.91 | 0.26 |

The QB MAE of 5.62 is expected and documented — quarterback scoring has higher inherent variance than skill positions due to touchdown clustering and game-script dependency. A QB can produce similar passing volume across two games and score vastly different fantasy totals depending entirely on red zone opportunities and turnover luck. This is a property of the problem, not a model failure.

---

## Key Findings

- Prior to selecting a Random Forest model, four other models were tested: Ridge Regression, Decision Tree, Huber Regressor, and Gradient Boosting. Testing on all models was done primarily on running backs.

- During testing, the best MAE scores across all models ranged from 4.10 to 4.45, which signaled that there was irreducible noise within the data inhibiting optimal predictions. This is largely due to the extreme touchdown variance mentioned above — something that cannot be picked up in even the most robust models. From plotting the target predictions and actuals in a residual plot, the shape showed heteroscedasticity, meaning as the actual target values increased, so did the variance. This touchdown variance greatly affected the QB model, where the optimal MAE hovered around 5.6 versus 2.9–3.9 for the other positions.

- For feature selection and pruning, it was evident that defensive stats did not provide much or any signal to a player's weekly fantasy score. Across all four positions, some of the highest-correlating features with the target had to be pruned due to a ‘logjam’ of collinearity they caused with other features. This increased the runtime of the models and decreased the accuracy of the scoring.

- MAE was chosen as the primary scoring metric over RMSE because the extreme variances did not need to be penalized based on the nature of the data. R² was used as a secondary metric to support how much signal the model captures despite the presence of noise.

---

## What's Next

- During model testing, I found myself chasing optimal scoring. These trials led to the conclusion that predicting fantasy points per week accurately is difficult. With additional time, I would like to explore more complex models for training, including deep learning approaches.

- The models will be run during the regular season prior to the beginning of each week's games. By season's end I plan on having a full 18 weeks of predictions and visualizing these predictions versus the actual outcomes for additional analysis.

