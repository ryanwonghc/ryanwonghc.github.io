---
title: "Premier League Game Recommender"
layout: post
excerpt: "Project aims to classify whether or not a Premier League game is memorable/worth watching and generate a list of recommended Premier League games to watch."
date: 2020-07-14
image:
  path: /images/Posts/PLPost.jpg
  thumbnail: /images/Posts/PLPreview.jpg
read_time: true
---

[Project Github Repository](https://github.com/ryanwonghc/Premier-League-Game-Recommender)

Techniques/tools used:
- Data Scraping: BeautifulSoup
- Python: numpy, pandas, matplotlib, seaborn, scikit-learn (Decision Trees, Random Forests, Linear Regression, Cross Validation, Pipeline), imblearn (SMOTE)

## Overview
I analyzed [this](https://www.kaggle.com/thefc17/epl-results-19932018) dataset of Premier League match statistics as well as data scraped from [this](https://www.fourfourtwo.com/us/100-best-premier-league-matches-ever) FourFourTwo article about the 100 best Premier League matches of all time in order to determine the factors that make a Premier League match "good" or "memorable". I then **built a tool to recommend a list of the best Premier League matches of the past to watch**. I performed feature engineering with a combination of the aforementioned dataset and additional data I scraped to find the strongest predictors regarding what makes a game memorable. I tested three different classifier models on the data: logistic regression, random forest, and decision trees. I used F1 score to evaluate the precision and recall of the models and concluded that a decision tree is the best model to use. I then fit a decision tree onto the entire dataset, performed hyperparameter tuning, and obtained an F1 score of 0.39 (recall score: 0.26, precision score: 0.81).

## Motivation
Football (soccer) has always been a passion of mine. I grew up a Manchester United fan and idolized players such as Cristiano Ronaldo and Wayne Rooney, hoping to one day fulfill my dreams of playing in the Premier League. I fell well short of my lofty childhood ambitions but my love for the sport never faded and I follow the [Premier League](https://www.premierleague.com/) religiously to this day. I began following the Premier League during the 2010-2011 season, and I became used to watching games every week. During the COVID-19 pandemic, all Premier League games were postponed, and, craving my weekly football fix, I decided to build a tool to recommend past matches (prior to 2010-2011) to watch.

## Methodology
To complete this project, I took the following steps:
1. Data Collection
2. Data Cleaning
3. Data Exploration
4. Model Building

## Data Collection
In this section, I describe the data that I obtained and the csv files they are stored in (accessible via [Github Repository](https://github.com/ryanwonghc/Premier-League-Game-Recommender))

1. EPL_Set.csv
[This](https://www.kaggle.com/thefc17/epl-results-19932018) kaggle dataset contains information from all Premier League games played since its inception (1992-1993). The information is only complete from the '95 - '96 season onwards and thus I discarded all data before this season. I stored this data in the file EPL_Set.csv.

2. best_games.csv
This file contains data from [this](https://www.fourfourtwo.com/us/100-best-premier-league-matches-ever) FourFourTwo article about the 100 best Premier League matches of all time. I obtained this data through web scraping. I analyzed this the data from the EPL_Set.csv dataset corresponding to each game in this list in order to determine the factors that make a Premier League match "good" or "memorable". The relevant code is in the ["best_games_scraper.py"](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/best_games_scraper.py) file.

3. teams.csv
This file contains a list of all teams that have played in the Premier League since its inception. I obtained this data by scraping [this](https://en.wikipedia.org/wiki/List_of_Premier_League_clubs) wikipedia page. The relevant code is in the ["team_scraper.py"](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/team_scraper.py) file.

4. rivalries.csv
This file contains a list of all known Premier League rivalries. This is important because I later explored whether or not a rivalry game made a game more likely to be memorable. I obtained this data by scraping [this](https://en.wikipedia.org/wiki/List_of_sports_rivalries_in_the_United_Kingdom) wikipedia page. The relevant code is in the ["rival_scraper.py"](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/rival_scraper.py) file.

5. points_history.csv
This file contains a list of the total Premier League points accumulated by each Premier League team since the inception of the Premier League. This is important because I used points accumulated as a metric to measure team success/popularity, then explored whether or not team popularity/success contributes to making a match memorable. I obtained this data by scraping [this](https://en.wikipedia.org/wiki/Premier_League_records_and_statistics) wikipedia page. The relevant code is in the ["total_points_scraper.py"](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/total_points_scraper.py) file.

## Data Cleaning
In order to perform prepare the data for analysis, I performed the following steps:

1. Drop Data Prior to '95 - '96 Season
Data was incomplete prior to this season, so I could not use earlier data. Thus I removed the data from the dataset.

2. Drop "Div" Column
This column in the dataset indicated which division the game was played in. Since the games in this dataset are Premier League games, this column is irrelevant.

3. Standardize Team Names
As I collected data from several different sources, some team names were inconsistent. For example, "Queens Park Rangers" was named "QPR" and "Queens Park" in my data. To resolve this issue, I used the list of team names from the "teams.csv" file as the standard naming convention and used difflib library to find the closest name from the list of standard names to each name not in the list in the data. The relevant code can be found in the ["standardize_names.py"](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/standardize_names.py) file.

4. Standardize Dates
There was some inconsistency in the collected data as some dates are recorded in the format DD/MM/YYYY, while some are in the format MM/DD/YYYY. Furthermore, some dates only have two digits to represent the year. This posed a problem when I was trying to compare dates of different matches. To resolve this issue, I converted all dates to DD/MM/YYYY format.

5. "Best Game" Column
I added a column to represent whether a game is in the list of "Best Games" (binary variable). This will be the column I try to predict with my classification models. A problem that I had with this was when comparing game information between the games in the dataset and the games in the "Best Games" set, I found that the two sets of data did not always have matching dates for the same game (sometimes the dates were +/- one day). To resolve this issue, I used datetime and timedelta objects to allow for dates that differed by at most one day.

After this stage, I ended up with a dataframe with 8740 entries and 9 features. The relevant code is in the ["Data Cleaning and Exploration.ipynb"](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/Data%20Cleaning%20and%20Exploration.ipynb) file.

## Data Exploration
In this section, I made 6 hypotheses regarding what constitutes a "Best Game" and added column features to the dataframe to test these hypotheses. The hypotheses are as follows:

A statistically significant portion of games in the "Best Games" set ...
1. Involve popular/successful teams
2. Are games played between rivals
3. Involve comeback wins
4. Are high scoring games
5. Are close games
6. Are played during the latter stages of the season

#### Hypothesis 1
A statistically significant portion of games in the "Best Games" set involve popular/successful teams
To test this hypothesis, I first calculated the frequency at which each team appeared in the set of "Best Games". I then charted the data in a bar chart, shown below:

![Fig1](image.jpg){: .align-center}

<p align="center">
  <img src="/images/Posts/PL_images/fig1.png"/>
</p>
From the chart above, it seems as though the more popular/successful teams appear in the "best games" set more often, confirming my hypothesis. The three teams with the highest count - Manchester United, Arsenal, and Liverpool - are three of most successful teams in Premier League history. I believe that this makes sense for the following reasons:
1. More successful teams tend to play in more high-stake games (games that have a large impact on the outcome of that year's premier league title race).
2. More successful teams tend to have higher viewership numbers for their games (more neutrals/rivals watch their games, and they usually have larger fanbases) and thus their games are more likely to remain in the collective memories of football fans for longer.
3. More successful teams attract the most skillful players who produce the most highlight reel worthy moments, making their games more memorable.
4. More successful teams are more likely to have more years in the premier league (3 teams get relegated to lower league divisions every year) and thus they play more premier league games, increasing their odds of playing in a memorable game.

To quantify the "popularity" or "successfulness" of the teams in each game, I gave each team a "popularity score", and summed up the two team's popularity scores to derive the popularity score for each game. I made the assumption here that the most successful teams are the most popular, and the metric I used to determine the popularity score was the total number of points the team has earned in Premier League history (Teams earn 3 points for each match won, 1 for each draw, and 0 for losses). This data was scraped from [this](https://en.wikipedia.org/wiki/Premier_League_records_and_statistics) wikipedia page.

Charting the popularity scores for each game in the "Best Games" set versus the rest of the games, I found that the median popularity score for games in the "Best Games" set is much higher than games not in the set, proving that memorable games involve successful/popular teams more often than not. The chart is shown below:
<p align="center">
  <img src="/images/Posts/PL_images/fig2.png"/>
</p>

**Verdict: Correct**

#### Hypothesis 2
A statistically significant portion of games in the "Best Games" set are games played between rivals
Using collected data on a list of Premier League rivalries, I used pie charts to show the ratio of rival games to non-rival games in both the "Best Games" set and the rest of the games. The charts are shown below:
<p align="center">
  <img src="/images/Posts/PL_images/fig3.png"/>
</p>
Statistics:
- Number of rival games in best games set:  26
- Number of non-rival games in best games set:  60
- Number of rival games in rest of data:  582
- Number of non-rival games in rest of data:  8072

As can be seen from the chart, the proportion of rival games in the "Best Games" set is much greater than the proportion of rival games in the rest of the data. This may be because there is extra emotional significance when it comes to games in which a team beats their fierce rivals, making the games more memorable. Rival games could possible also receive more media coverage, and higher viewership numbers mean that rival games are more likely to leave a greater impression on football audiences as a whole.

In terms of absolute numbers, there are a lot more rival games not in the 'Best Games' set than there are in the rest of the data. A possible reason for this may be because when teams play in rival games, there is a lot more to lose (more pride at stake). As a result, teams may play extra cautiously in order not to lose the game. Defensive football rarely results in highlights or memorable plays and as a result the game as a whole is not memorable and not worthy of the 'Best Games' title.

**Verdict: Correct**

#### Hypothesis 3
A statistically significant portion of games in the "Best Games" set involve comeback wins
I defined a comeback a game in which the team losing at half time wins the game, even though this is not always the case (teams can turn around a deficit in the same half that they fell behind in), as it was the best I could do given the collected data. I hypothesize that comeback games are more memorable than regular games because of the emotional rollercoasters that fans go through when watching these games.

I again used pie charts to show the ratio of comeback to non-comeback games in both the "Best Games" set and the rest of the games. The charts are shown below:
<p align="center">
  <img src="/images/Posts/PL_images/fig4.png"/>
</p>
Statistics:
- Number of comeback games in best games set:  18
- Number of non-comeback games in best games set:  68
- Number of comeback games in rest of data:  321
- Number of non-comeback games in rest of data:  8333

As shown in the chart above, comeback wins do appear in the "Best games" set at higher rates. However, it is important to note that the proportion of comeback games in the 'Best Games' set is quite small. A reason for this could be that comebacks are rare, and there are other more significant elements of a game that make it memorable. Furthermore, there is also variance between comebacks. A team is much more likely to come back from one goal down than they are to come back from three goals down (which would make the game much more memorable than a one goal comeback).

**Verdict: Correct, but statistically insignificant**

#### Hypothesis 4
A statistically significant portion of games in the "Best Games" set are high scoring games
Football has traditionally been a low scoring game, especially in comparison to other popular sports such as basketball. As a result, high scoring games are often more memorable to fans. In order to test this hypothesis, I summed up the total number of goals for each game and charted this information for the "Best Games" set as well as the rest of the data in bar charts, shown below:
<p align="center">
  <img src="/images/Posts/PL_images/fig5.png"/>
</p>

Statistics:
|   | 0  | 1  |  2 | 3  | 4  | 5  | 6  |  7 |  8 | 9  | 10  |  11 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Best Games  | 2  |  5 |  1 |  10 | 7  |  21 | 8  | 11  | 11  |  5 | 4  | 1  |
| Rest of Data | 726 | 1598 | 2082 | 1843 | 1313 | 637 | 285 | 115 | 43 | 11  | 1  | 0  |

From the charts above, we can come to several conclusions:
- The 'Rest of Data' set is positively skewed while the 'Best Games' set is more normally distributed.
- Only a small portion of games (9%) in the 'Best Games' set involve low numbers of goals ([low defined as less than the average number of goals scored during a football match, which is 2.6](https://www.statista.com/statistics/269031/goals-scored-per-game-at-the-fifa-world-cup-since-1930/#:~:text=FIFA%20soccer%20World%20Cup%3A%20Average,scored%20per%20games%201930%2D2018&text=At%20the%20latest%20World%20Cup,of%202.6%20goals%20per%20game.)).
    - Only ~2% of games involve 0 goals.
- The mode of the 'Best Games' set (5) is much greater than the mode of the 'Rest of Data' set (2). This suggests that the 'Best Games' are typically high scoring. The mode is almost double the average number of goals scored in an football match.
- A larger proportion of total goals scored in the 'Best Games' set are in the higher end of the range, suggesting that more often than not, 'Best Games' are high scoring games
    - The absolute number of games in the 'Best Games' set with 10 or more goals is 5 compared to 1 in the rest of the data, even though the ratio of the number of games in the 'Best Games' set to the number of games in the rest of the data is 8654:86

**Verdict: Correct**

#### Hypothesis 5
A statistically significant portion of games in the "Best Games" set are close games
Close games keep fans on the edge of their seats, and it can be more fun to see your favorite team come out on top after a close game as opposed to a one-sided game. In order to test this hypothesis, I calculated the goal difference of each game (goals scored by winner - goals scored by loser). Drawn games have a goal difference of 0. I then charted this information for the "Best Games" set as well as the rest of the data in bar charts, shown below:
<p align="center">
  <img src="/images/Posts/PL_images/fig6.png"/>
</p>

Statistics:
|   | 0  | 1  |  2 | 3  | 4  | 5  | 6  |  7 |  8 |
|---|---|---|---|---|---|---|---|---|---|
| Best Games  | 18  |  41 |  11 |  6 | 1  |  5 | 1  | 1  | 2  |
| Rest of Data | 2252 | 3273 | 1825 | 807 | 333 | 117 | 37 | 7 | 3 |

Both graphs are positively skewed. The mode of the 'Best Games' set is 1 goal, which suggests that a large proportion of the 'Best Games' were close games (the winning team won by one goal). However, as can be seen from the rest of the data, the majority of the games have a goal difference of one so the large number of games with a goal diffrence of one in the 'Best Games' set could just be due to a large sample size. The proportion of games with a goal difference of one in the 'Best Games' set (48%) is greater than it's counterpart in the rest of the data (38%), which suggests that my hypothesis is correct to a degree, although the two factors (appearance in 'Best Games' set, small goal difference) may not be strongly correlated.

**Verdict: Correct, but not statistically significant**

#### Hypothesis 6
A statistically significant portion of games in the "Best Games" set are played during the latter stages of the season
Some Premier League games matter more than others. Games played towards the end of the season can have a large impact on the league table because at the latter stages of the season, players become fatigued both mentally and physically, so results at this stage can have a large impact on the morale of players. These games are more high-stakes as a result and can be more memorable. Furthermore, results at this stage can mathematically confirm a team's place in the league table (confirmed as champions, confirmed spot in European competition, confirmed relegation, etc.) and the fate of their rivals or teams close to them in the table, which can have a major emotional and financial impact on all relevant teams. As a Manchester United fan, Manchester City's 3-2 win over QPR in the final game of the 2011-12 season to win the premier league comes to mind.

To test this hypothesis, I assigned each month a number from 0 (August) to 9 (May), as Premier league seasons are played from August to May. I then charted the number of games played in each month. The results are as follows:
<p align="center">
  <img src="/images/Posts/PL_images/fig7.png"/>
</p>

Statistics:
|   | 0  | 1  |  2 | 3  | 4  | 5  | 6  |  7 |  8 | 9 |
|---|---|---|---|---|---|---|---|---|---|---|
| Best Games  | 4  |  13 |  12 |  10 | 6  |  3 | 8  | 4  | 10  | 16 |
| Rest of Data | 762 | 805 | 790 | 876 | 1265 | 862 | 770 | 850 | 1083 | 591 |

The mode of the 'Best Games' data is 9 (May), suggesting that a large proportion of 'Best Games' (~19%) are played towards the end of the season. The data is distributed bimodally, suggesting that my hypothesis is not quite correct. It appears that the 'Best Games' are predominantly occur near the beginning and end of the season. This data surprises me a little as I would have expected more 'Best Games' to occur in December (4). This is because firstly, more games are played in December, and secondly, a lot of big games (games starring high profile rivals) are played during the festive season to capitalize on the fact that there are more viewers and thus more revenue potential. An explanation for this could be that since December has the most games played, perform below their best due to accumulated fatigue.

**Verdict: Disproved**
- 'Best Games' occur predominantly near the beginning and the end of the season


After this stage, I ended up with a dataframe with 8740 entries and 16 features. The relevant code is in the ["Data Cleaning and Exploration.ipynb"](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/Data%20Cleaning%20and%20Exploration.ipynb) file.

## Model Building and Performance
#### Methodology
I used classification algorithms to classify games as either in or out of the 'Best Games' set. The list of games classified as part of the "Best Games" set served as the list of recommended games to rewatch during quarantine. I compared the results of three classification algorithms: Logistic Regression, Decision Tree, and Random Forest.

The metric I used to evaluate the models was the F1 score, which weighs precision and recall equally. I believe recall is an important metric because there is no point in creating a list of recommendations if a minimal amount of games on the list are actually worth rewatching. At the same time, precision is important because I do not want to waste my time watching a large number of games that should have been classified as unmemorable. I anticipated the precision of the model to be fairly low due to the highly imbalanced nature of the data.

#### Choosing Features
To minimize overfitting and training time, I only trained the model with features that correlate the most with the dependent variable. To do so, I decided to plot a correlation heatmap and take the 5 most highly correlated features.

<p align="center">
  <img src="/images/Posts/PL_images/corrmat.png"/>
</p>

According to the heatmap, the top five features are:

- Total Goals
- FTHG (Full Time Home Goals)
- FTAG (Full Time Away Goals)
- Rival Game
- Comeback

However, we also see that Total Goals is very highly correlated to FTAG and FTHG. This is because Total Goals is the sum of FTAG and FTHG. To ensure collinearity, we will not consider FTAG and FTHG. We take the next 4 most highly correlated variables, resulting in the following features:

- Total Goals
- Rival Game
- Comeback
- Popularity Score
- ~~HTAG (Half Time Away Goals): also highly correlated with Total Goals~~
- ~~HTHG (Half Time Hway Goals): also highly correlated with Total Goals~~
- ~~Goal Difference: also highly correlated with Total Goals~~
- HTR (Half Time Result)

#### Balancing Classes using SMOTE
As previously mentioned, the data is highly imbalanced. Only 100 samples that are classified as a "Best Game" while 8640 samples are not. Since there was a clear imbalance between the number of samples in each class, I needed to either oversample data from the minority class (the 'Best Games' class), or undersample data from the majority class to compensate. I chose to oversample from the minority class using SMOTE (Synthetic Minority Oversampling TEchnique) because if I undersample the classifier may not have enough data (only 200 samples) to classify the samples with high precision and recall.

#### Pipelines and KFold Cross Validation
I created a Pipeline for each classifier (decision tree, random forest, logistic regression) to simplify the process of transforming the data (using SMOTE) then applying the classifier, as well as to ensure that I could cross validate the F1 scores of each classifier type by providing it with different parameters (input data). I performed cross validation by using the KFolds cross-validator to split the data into train/test splits. As per the Pareto Principle, I used an 80/20 train-test split, splitting the data into 5 folds.

#### Choosing the Best Performing Classifier
The performance of each classifier is as follows:
```
Logistic Regression F1: 0.08881155233993385
Logistic Regression precision: 0.04732568063882119
Logistic Regression recall: 0.8291491841491843


Decision Tree F1: 0.0952839031489884
Decision Tree precision: 0.06264183157268263
Decision Tree recall: 0.25952547452547453


Random Forest F1: 0.08021591254261323
Random Forest precision: 0.052910848549946286
Random Forest recall: 0.24271228771228773
```
Since the decision tree had the highest F1 score, my final model was fitted using a decision tree.

#### Final Model Building
I used GridSearchCV along with 5-fold cross validation to tune the hyperparameters. After fitting the model and outputting a prediction, the model's performance was determined to be as follows:
```
F1 score: 0.3893805309734513
Recall score: 0.2558139534883721
Precision score: 0.8148148148148148
```
Additionally, the confusion matrix is as follows:

<p align="center">
  <img src="/images/Posts/PL_images/conmat.png"/>
</p>

From the scores and the confusion matrix, we see that this model has a high precision and a low recall. This is surprising as it was the other way around previously (when deciding which model to choose). I think this was due to the hyperparameter tuning limiting the maximum depth of the decision tree, preventing overfitting. Overall, I think this model performed quite well; it is very precise, meaning that I do not have to waste my time watching boring games. While it was unable to correctly classify a majority of the minority class, I am not too concerned as the main objective of this project is to find any games worthy of watching- the number of games found does not matter too much (unless I finish watching all the recommended games and run out of games to watch).

#### List of Recommendations and Observations
The following are the games that were not initially in the "Best Games" set but were predicted to be a memorable game.
Date       | HomeTeam          | AwayTeam          | FTHG | FTAG | FTR | HTHG | HTAG | HTR | Season  | DateTime | Popularity Score | Rival Game | Comeback | Total Goals | Goal Difference | Month | Predicted |
| ---- | --------- | ---------- | ----------------- | ----------------- | ---- | ---- | --- | ---- | ---- | --- | ------- | -------- | ---------------- | ---------- | -------- | ----------- | --------------- | ----- | --------- |
| 3660 | 0         | 1/2/05     | Arsenal           | Manchester United | 2    | 4    | \-1 | 2    | 1    | 1   | 2004-05 | 2/1/05   | 1.90175277       | 1          | 1        | 6           | 2               | 6     | 1         |
| 5273 | 0         | 25/04/2009 | Manchester United | Tottenham Hotspur | 5    | 2    | 1   | 0    | 2    | \-1 | 2008-09 | 4/25/09  | 1.73570111       | 0          | 1        | 7           | 3               | 8     | 1         |
| 6336 | 0         | 26/02/2012 | Arsenal           | Tottenham Hotspur | 5    | 2    | 1   | 2    | 2    | 0   | 2011-12 | 2/26/12  | 1.63745387       | 1          | 0        | 7           | 3               | 6     | 1         |
| 6569 | 0         | 17/11/2012 | Arsenal           | Tottenham Hotspur | 5    | 2    | 1   | 3    | 1    | 1   | 2012-13 | 11/17/12 | 1.63745387       | 1          | 0        | 7           | 3               | 3     | 1         |
| 7987 | 0         | 14/08/2016 | Arsenal           | Liverpool         | 3    | 4    | \-1 | 1    | 1    | 0   | 2016-17 | 8/14/16  | 1.75461255       | 0          | 0        | 7           | 1               | 0     | 1         |

Looking at these games, it seems that one common factor amongst them is the high-scoring nature of these games (at least 6 total goals score, which is very rare for football). The popularity scores for these teams are also relatively high- the range of scores is (0,2), and each of these games have popularity scores of at least 1.63.

The final list of games predicted to be a memorable game are as follows:
| Date       | HomeTeam          | AwayTeam          |
| ---------- | ----------------- | ----------------- |
| 23/10/1999 | Chelsea           | Arsenal           |
| 12/2/00    | West Ham United   | Bradford City     |
| 25/02/2001 | Manchester United | Arsenal           |
| 29/09/2001 | Tottenham Hotspur | Manchester United |
| 19/11/2001 | Charlton Athletic | West Ham United   |
| 7/2/04     | Everton           | Manchester United |
| 9/4/04     | Arsenal           | Liverpool         |
| 22/01/2005 | Norwich City      | Middlesbrough     |
| 1/2/05     | Arsenal           | Manchester United |
| 28/04/2007 | Everton           | Manchester United |
| 29/09/2007 | Portsmouth        | Reading           |
| 29/12/2007 | Tottenham Hotspur | Reading           |
| 21/04/2009 | Liverpool         | Arsenal           |
| 25/04/2009 | Manchester United | Tottenham Hotspur |
| 20/09/2009 | Manchester United | Manchester City   |
| 22/11/2009 | Tottenham Hotspur | Wigan Athletic    |
| 20/11/2010 | Arsenal           | Tottenham Hotspur |
| 28/08/2011 | Manchester United | Arsenal           |
| 23/10/2011 | Manchester United | Manchester City   |
| 29/10/2011 | Chelsea           | Arsenal           |
| 26/02/2012 | Arsenal           | Tottenham Hotspur |
| 22/04/2012 | Manchester United | Everton           |
| 17/11/2012 | Arsenal           | Tottenham Hotspur |
| 19/05/2013 | West Bromwich     | Manchester United |
| 23/01/2016 | Norwich City      | Liverpool         |
| 14/08/2016 | Arsenal           | Liverpool         |
| 26/11/2016 | Swansea City      | Crystal Palace    |

By looking at the complete list of recommended matches, we can see that the trend of the games being high scoring carries over; however, the popularity score does not. We can thus conclude that the largest predictor of whether or not a game is worthy of rewatching (according to the model) is the total number of goals scored in that game.

The jupyter notebook and relevant code can be found [here](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/Model%20Building.ipynb). After this stage, I was able to generate a dataframe (which I converted to a csv file that can be found [here](https://github.com/ryanwonghc/Premier-League-Game-Recommender/blob/master/recommended_matches.csv)) containing 27 recommended matches (not in any order).

## Future Work
1. Betting odds
I believe that another predictor of whether or not a game is considered a "Best Game" is whether or not the game involved an upset, which is when a team defies the odds to beat the team favored to win the game. Data for betting odds for each of the matches was only available going back to 2000 so the 1995-1999 seasons would have to have been excluded from the analysis. It would be interesting to test this hypothesis and analyze the effect of the magnitude of the upset (a team with winning odds of 15/1 winning a game would be a much larger upset than a team with winning odds of 2/1)

2. Ranking the order of games in terms of rewatchability
This project generates a list of recommended games to rewatch. What it does not do is differentiate between which games are more enjoyable to rewatch than others in the set of recommended games, which is important for people with limited time to rewatch games. A future project could involve ranking each game by rewatchability.
