---
title: "XD"
layout: post
excerpt: "Project aims to predict the app store rating of a mobile game using multiple linear regression, considering factors such as game size, genre, and price."
date: 2020-04-23
image:
  path: /images/Posts/GameRatingPost.jpg
read_time: true
---

[Link to Github Repository](https://github.com/ryanwonghc/regression-mobile-games)

## Data Cleaning
* I used [mobile strategy games dataset](https://www.kaggle.com/tristan581/17k-apple-app-store-strategy-games) from Kaggle to train and test my model. The dataset contained data on 16,847 different games. I performed data cleaning to extract data that I felt would be beneficial to train the model with and eliminate data that I deemed irrelevant to the model. I made the following changes:

* Removed all entries without user rating data
* Removed all entries with less than 100 user ratings to ensure that the ratings are credible (the higher volume of ratings the more likely they are a true representative of consumer opinion)

I then added the following columns:

* Subtitle (y/n): an indicator variable describing whether or not the game had a subtitle.
* In-app purchases: in-app purchases range in price from $0 to $99.99. I grouped games into 4 quadrants by price:
  * $0 - $24.99
  * $25 - $49.99
  * $50 - 74.99
  * $75 - $99.99
* Number of words in description
* Age Rating: in the original dataset, there are 4 possible values for this column- 4+, 9+, 12+, and 17+. It is important to note that 9+, 12+, and 17+ games are also 4+ games, 12+ and 17+ games are also 9+ games, and 17+ games are also 12+ games. Thus, when creating indicator variables, I decided to make the categories '< 9', '< 12', and '< 17'. 4+ games satisfy < 9, 4+ and 9+ games satisfy < 12, 4+, 9+, and 12+ games satisfy < 17, while 17+ games do not satisfy any of the categories.
* Number of languages the game is offered in
* Size of game (in bytes)
* Genre: I created indicator variables for each possible game classification genre (there are 35 in total, examples include "Education", "Finance", and "Lifestyle"; I disregarded the "Game" genre because all data entries were marked with the "Game" genre.
* Time since last update (in days, as of April 20, 2020)

After completing the data cleaning step of the project, I ended up with data from 2982 unique games to train and test my model with.

## Model Building and Performance
I split the data into training and testing sets (50/50 split). I built the model using multiple linear regression and evaluated it using R-squared and Mean Absolute Error. I used the R-squared value to evaluate how accurately my model is able to predict mobile game ratings. I chose to use MAE to evaluate my model because it is relatively easy to interpret and outliers aren’t particularly bad in for this type of model.

The model has an R-squared value of 0.1678 and a Mean Absolute Error of 0.37.

## Findings
I used this [backward elimination tutorial](https://medium.com/@mayankshah_85820/machine-learning-feature-selection-with-backward-elimination-955894654026) to perform backward elimination on the variables to find the most significant variables. Using an significance level of 0.05, the following variables were deemed to be significant (P-Value < Significance Level):

Positive Coefficient of Regression
* Subtitle (y/n)
* In-app purchases ranging from $0 to $24.99
* Age Rating < 12
* Size of game ranging from 46.5 MB to 112 MB
* Size of game ranging from 112 MB to 221 MB
* Size of game larger than 221 MB
* Genre: Magazines and Newspaper

Negative Coefficient of Regression
* Age Rating < 17
* Genre: Adventure
* Genre: Board
* Genre: Reference
* Days Since Last Update

We can make a few observations from the information above:
* Include a subtitle (something like "Original Brain Training" for Sudoku)
* Include in-app purchases, but the price range for these items should be from $0 to $24.99
* Cater to wider varieties of audiences, ensure that your game is playable by younger audiences (4+, 9+ range)
* Make games more detailed/ more content/ better graphics- larger install sizes are positively correlated with better ratings
* Genres such as "Magazines and Newspapers" are generally higher rated
* Genres such as "Adventure" and "Board" are generally lower rated. This could be because personal tastes vary more drastically for apps/games of this category, and people could leave distasteful ratings if the game is not built perfectly to their liking or if the game is frustrating to play for a person of average/below average skill level.
* Apps that were updated the most recently had lower ratings in general. I am unable to generate any concrete conclusions from this information, however a possible hypothesis is that there has been a recent trend in the game development industry of adding features that users dislike, such as increased advertisements or expensive in-app purchases that increase the disparity between paying and non-paying users. It would have been interesting to do an analysis on how the frequency of updates impacts average ratings. However, there was insufficient data to do so.

However, it is important to note that using only the aforementioned significant variables to test and train a multiple linear regression model resulted in an R-squared value of 0.1684 and a Mean Absolute Error of 0.367, which are not significant improvements, so the observations generated do not carry much weight.

## Thoughts and Conclusions
In conclusion, it appears that multiple linear regression may not be a good model when modeling this scenario, though it is possible that it is hard to predict mobile game ratings to a high degree of accuracy in general as there is an element of randomness to ratings (hence the low R-squared value). Often times, people only rate the game when they have to and do not put much time or thought into it. Speaking from personal experience, I believe that people often make impulsive extreme ratings (ratings on the extreme of both the positive and negative ends of the grading scale) depending on their first impressions of the game, which skews the ratings and makes them harder to predict.

An alternate possibility to the low R-squared value is that mobile game ratings are influenced by a myriad of other factors not included in the dataset, such as whether or not the game prompts the user to give it a review.

Although the model had a low R-squared value, I believe that the mean absolute error of 0.37 is not all that bad because in my opinion, it still reasonably conveys public opinion on the game. I personally subconsciously round mobile game ratings to the nearest whole number to associate it with a word (very bad, bad, average, good, very good) when gauging the quality of the game, and an error of 0.37 means that I would still be rounding to the same number in most cases. However, I understand that this could be different for others and this model may not be useful to others when they gauge the quality of a game.

From the game designer's point of view, when designing a game, it is important to keep the goals of designing the game in mind (to receive as high of a rating as possible, to reach as many people as possible, to generate as much revenue as possible, etc.). Hence, although it is never hurts to consider which factors positively or negatively impact game ratings (as stated in the "Findings" section), it is not the only thing to consider during game design.

## Limitations
A few limitations of my model are:
* When counting the length of the game description, I noticed that some descriptions contained unicode for special characters such as emojis, and I was unable to find a way to exclude those from the count.
* I believe that there are more meaningful ways to categorize the in-app purchase prices, as from personal experience, certain price points are more common than others. For example, a majority of the games I have played in the past have only had in-app purchases worth $9.99 or less, so it might be beneficial to make a category from $0 to $9.99 instead of $0 to $24.99.

## Future Work
* Try to use different models to train and test the data (not just multiple linear regression)
* Potentially add different kinds of data (different columns) to the dataset
