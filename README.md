# NBA Spread, Total, and OREB Models
---------------------------------------------------------------------------------------
By: Will Kenerly, Kevin Sullivan, Barrett Buhler, Ghasan Ahmed, Kai White,
Wesley Lemons

I contributed most of the modelling and the data cleaning process but this work was a result of group project

I. Data

A. Cleaning the Data

To clean up the data, we first loaded up game information for the 2021-22 and 2022-23
seasons using the nbastatR library. The data was separated by team rather than by game on every
row and had many variables that we were not interested in. We selected the variables of interest
(idGame, dateGame, nameTeam, locationGame, orebTeam, ptsTeam) and then split it into
individual Home and Away data sets by filtering the locationGame and consequently dropping it
after. We then renamed the nameTeam variable to whether it was the home or away team and full
joined by idGame. In the process of joining, we used Mutate() to create additional variables for
Spread, Total, and OREB.

After loading and cleaning the nbastatR data, we loaded up the “games_excel”, filtered
for season >= 2021, and then renamed the GAME_ID variable to idGame so that we can full join
it to the nbastatR data. After doing so, we read in the “games_details.csv” and grouped it by
GAME_ID and TEAM_ID to extract individual team performances for each game. We then used
Summarize () to generate avg_plus_minus, avg_FG_PCT, and avg_FG3_PCT by taking the
means of the respective variables. Then, the data “games_excel” file was utilized again to pull
out team ids for home and away teams for every game id. We created two data frames, one
containing GAME_ID and TEAM_ID for both the home and away teams. After doing so, we used
left_join() to combine the home id’s data and the data frame generated from “games_details.csv”
earlier by GAME_ID and TEAM_ID. The process was repeated for the away team, and then both
data frames were joined to create a table including game id’s, team id’s average plus minus,
average field goal percentage, and average 3-point percentage. Finally, we renamed GAME_ID
to idGame to full join with the final data from nbastatR.

This dataset laid the groundwork for our future models. We opted to present each row
represented as different games and because of that broke up most of our variables into home and
away metrics due to the varied performance for home versus away teams. This dataset featured
every game from the 2021 season to the current week. From here, we added our new variables
and outside data listed below. We downloaded further statistics from the nbastatR package and
other sources to include the simple variables such as blocks and turnovers, to represent the
number that both teams accumulated for each specific game.

B. New Engineered Variables

1) Rivalry Score
For our first engineered variable, we attempted to quantify the idea of a rivalry. Rivalries
are a major part of sports, and are a key reason why fans tune in to certain games. Our hypothesis
was that teams who are rivals place more emphasis on their games against each other, and in turn
play a “higher quality” game. This rivalry effect could impact game strategy such as types of
shots attempted and rebounding from the psychological perspective: an effect that may have a
direct impact on our three variables of interest (Spread, Total, and OREB). To quantify this idea,
we created the following formula with SC standing for “same conference”, SD “same division”,
and SS “same state” and assigned a rivalry score to all 435 distinct possible NBA matchups (with
home/away and away/home considered the same):

𝑅𝑖𝑣𝑎𝑙𝑟𝑦 𝑆𝑐𝑜𝑟𝑒 = 2(𝑆𝐶) + 3(𝑆𝐷) + 4(𝑆𝑆) + 5(𝑃𝑙𝑎𝑦𝑜𝑓𝑓 𝑠𝑒𝑟𝑖𝑒𝑠 𝑖𝑛 𝑝𝑎𝑠𝑡 10 𝑦𝑒𝑎𝑟𝑠) + 6(𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑁𝐵𝐴 𝐹𝑖𝑛𝑎𝑙𝑠 𝑎𝑔𝑎𝑖𝑛𝑠𝑡 𝑒𝑎𝑐ℎ 𝑜𝑡ℎ𝑒𝑟)(𝐷𝑖𝑓𝑓𝑒𝑟𝑒𝑛𝑐𝑒 𝑖𝑛 𝑤𝑖𝑛𝑛𝑖𝑛𝑔 %) + (𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑡𝑖𝑚𝑒𝑠 𝑡ℎ𝑒 𝑡𝑤𝑜 𝑡𝑒𝑎𝑚𝑠 ℎ𝑎𝑣𝑒 𝑝𝑙𝑎𝑦𝑒𝑑 ℎ𝑖𝑠𝑡𝑜𝑟𝑖𝑐𝑎𝑙𝑙𝑦)

We decided that questions such as whether they are in the same conference, whether they
are in the same division, whether they are in the same state, whether they have played each other
in a playoff series during the past 10 years, and how many times the two teams have played each
other in the NBA finals impact how much of a rival two teams are. We assigned weights to each
of these measures subjectively based on our knowledge of the NBA and rivalries in general. We
then captured two other main features of a rivalry in the denominator. First, rivals generally have
a close winning percentage against each other. In other words, one team does not historically
dominate the other. To capture this, our denominator features the difference in the teams winning
percentage against each other. This penalizes teams who have consistently beaten another
historically by increasing the denominator. This difference is added to the total number of times
the two teams have played each other, as this adds historical depth to the calculation and again
penalizes teams who have not played any “meaningful” games historically against each other.
Lastly, to ensure every pairwise matchup had a rivalry score greater than 0, if none of the
numerator criteria was met we assigned a numerator of 1.

This formula is admittedly subjective, however, the idea of a rivalry is hard to quantify
and is subjective in nature. The numerator of our formula features four binary variables and one
numeric variable that we as a group considered the foundations of any rivalry. We then assigned
a subjective weight to each variable to quantify the subjective importance of each. We then
created a denominator that penalizes teams who either fail to play meaningful games against
each other or who have a disparity in winning percentage against each other. Overall, we created
the Rivalry Score variable to attempt to quantify a subjective idea that has a very real presence in
the way NBA teams play against each other in the hope that it results in stronger models to
predict Spread, Total, and OREB.

2) Shooting Advantage
For our second engineered variable, we created a metric that measures whether the home
or away team had a better shooting performance for each game. The only way to win an NBA
game is to score points, and apart from free throws, the only way to score points is to shoot the
basketball. A team’s field goal percentage measures how effective a team is at shooting. This
simple ratio provides a measure to determine whether the home or away team had an advantage
in shooting during any game. A value greater than 1 indicates the home team had the advantage,
and a value less than 1 indicates the away team had the advantage. Our group hypothesized that
whichever team shot the basketball at a higher percentage would have a higher chance of
winning a game, and in turn would be a useful predictor on our variables of interest, especially
our Spread and Total variables.

𝑆ℎ𝑜𝑜𝑡𝑖𝑛𝑔 𝐴𝑑𝑣𝑎𝑛𝑡𝑎𝑔𝑒 = 𝐹𝑖𝑒𝑙𝑑 𝐺𝑜𝑎𝑙 𝑃𝑒𝑟𝑐𝑒𝑛𝑡𝑎𝑔𝑒 (𝐻𝑜𝑚𝑒) * 𝐹𝑖𝑒𝑙𝑑 𝐺𝑜𝑎𝑙 𝑃𝑒𝑟𝑐𝑒𝑛𝑡𝑎𝑔𝑒 (𝐴𝑤𝑎𝑦)

C. Outside Data

In addition to the new variables listed above, as well as the statistics given to us by the
original “Games” and “Games_Details” packages, we also included some additional, outside
data to help us construct detailed models and make predictions. Because basketball teams only
have five people playing at one time, we believed that the individual size and age of each player
could be a very important factor, especially in the offensive rebound model. Thus, we found the
average height, weight, and age of each team from an R package called nbastatR and chose to
include these as additional variables for our models. From this same package we also chose to
include the number of shots missed in a game as this would likely be a good predictor for
offensive rebounds as well. Other predictors we brought in include defensive rebounds,
turnovers, and blocks. Our thought process was as follows: a good defensive rebounding team
may indicate a good offensive rebounding team. Similarly, a team with a lot of turnovers may
suggest a lower total number of points per game. Mirroring that, a team who is good defensively
and is prone to blocking a lot of shots may also contribute to a lower overall total.
Another piece of outside data that we included was the number of fast break points
recorded in the game. In recent years, there has been an overall decrease in offensive rebounds
league-wide which is largely attributed to the preference of having more players on offense on
fast breaks due to the high point conversion. Hence, we decided to add the fast break point
variable.

We also decided to include team power rankings for each season from NBC Sports. We
believed that the higher ranked teams would have an increased likelihood of winning over lower
ranked teams which we hypothesized would improve our spread, total, and offensive rebound
models.

Lastly, data for the calculation of the Rivalry_Score variable was taken from the website
“landofbasketball.com” which features an in-depth, head to head breakdown of every NBA
pairwise matchup throughout NBA history.

D. Variable Names
The variables in our dataset are as follows:

idGame, dateGame, Home, OREB_H, PTS_H, block_H, TO_H, heightH,
weightH, ageH, Away, OREB_A, PTS_A, block_A, TO_A, heightA, weightA,
ageA, Spread, Total, OREB, Rivalry_Score, TEAM_ID_H, avg_plus_minus_H,
FG_PCT_H, FG3_PCT_H, TEAM_ID_A, avg_plus_minus_A, FG_PCT_A,
FG3_PCT_A, year, Shooting_Advantage, Power_Ranking_H,
Power_Ranking_A, dreb_H, shots_missed_H, dreb_A, shots_missed_A,
Fast_Break_H, Fast_Break_A

The variable abbreviations and definitions are below. Variables ending with an _H
signifies the home team, and variables ending with an _A signify the away team.
idGame: Unique game ID
from nbastatr
dateGame: Date of game
Home: Name of home
team
PTS: Points scored
block: Number of blocks
recorded
TO: Turnovers
height: Average height of
team in inches
weight: Average weight of
team in pounds
age: Average age of team
Away: Name of away team
Spread: Home points -
Away points
Total: Home points +
Away points
OREB: Home offensive
rebounds + Away offensive
rebounds
Rivalry_Score: See
engineered variables
section
Shooting_Advantage: See
engineered variables
section
6
TEAM_ID: Team id
number given by nbastatr
avg_plus_minus: Average
+/- score of all players on
team
FG_PCT: Field goal
percentage
FG3_PCT: 3 point field
goal percentage
year: Year of game
Power_Ranking: Power
ranking rating for each
NBA team
dreb: Defensive rebounds
shots_missed: Number of
shots missed in game
Fast_Break: Number of
fast break points recorded
in game

E. Dealing with NAs in the Data
Early in our data cleaning process we found we had some incomplete game data with
nine rows of our data containing NA values. Since the number was so few we opted to just
remove those games from the dataset. Days later, when pulling data from the nbastatR package,
those empty data entries were filled in, so we were able to reintroduce those games into the data.
Also, while collecting average height and weight data for teams, we encountered one player who
did not have this information recorded. Since it was just one player we simply removed him from
the dataset before incorporating these new variables.

F. Multicollinearity and Perfect Predictors
In dealing with the problem of multicollinearity, many of the points variables such as
home points and away points could be used to derive either spread or total thus we had to remove
them from our data while making our models.
The same also holds true for the offensive rebound models where we removed OREB_A
and OREB_H from the data set to avoid having a perfectly predicting model with R squared of
one.

II. Methodology

After our dataset was cleaned we divided the data into training and testing datasets. We
first set the seed to 538 to make sure that our models were built from the same randomly selected
data. We decided to randomly allot 80% of the dataset for training and building our models. The
other 20% of the dataset was designated for testing our models. We had a total of 3454
observations, with 2763 observations used for training and 691 observations used for testing.

Before we began creating our models, we created histograms for the Spread and Total
variables to check their distributions and ensure normality.
Both distributions appear to be normal although the distribution of Total is slightly right
skewed. Because of this, we did not remove any potential outliers from our data.

A. Spread

Spread is equal to the number of points scored by the home team minus the number of
points scored by the away team.

Methodology for Spread

In order to select the best model to predict Spread, we trained our models based on 2763
observations. The remaining test observations were then used to calculate the below mean
absolute error values. The model with the lowest mean absolute error value was then selected to
make our predictions. When we created our models for Spread, we began by checking the
conditions of linearity and removed the at home points, away points, and the total variables for
these models. We then checked to see which of our predictors had a correlation above .5 and
found that the FG_PCT variables have a correlation of .78. After this basic data exploration we
decided to emphasize shooting variables in constructing our predictors and devised our shooting
variable from this. The significance of shooting percentages was also confirmed in the summary
tables of all of our models.

One model we attempted to create for predicting Spread was a K-Nearest Neighbor
regression model (KNN). Prior to working with the dataset we first scaled the data since KNN is
a distance based algorithm. Once the data was split into test and train data we looped through 20
different k values to find the optimal k value for our dataset using the knn.reg() function from the
FNN package in R and by calculating the smallest mean absolute error for the 20 k values. From
this model, we found the optimal KNN regression model to have a k = 1 and a mean absolute
error of 2.03488. The mean absolute error for this model was larger than most of the other
models we used to predict Spread so we did not consider this model for our predictions.
Another technique we employed for model creation was linear regression. We began with
a baseline model which predicted Spread based on all of our possible predictors aside from
PTS_H and PTS_A as they would be perfect predictors. We also used backwards selection to
remove insignificant predictors to create another linear model. Similar to our approach to linear
modeling, we also created a baseline polynomial model which included the same predictors as
the baseline linear model but raised to both the first and second powers. We then used the same
backwards selection method to narrow down our predictors. Each of these models were created
using the training data mentioned above and then evaluated via the mean absolute error created
when the models were used to predict Spread in our test data.

Our model that gave the lowest mean absolute error for Spread however, was the deep
learning neural network linear regression we created using Tensorflow. As mentioned above, we
broke eighty percent of our dataset in to our training data and the rest into our test data. Next, we
split our training and test data into features and labels to isolate our explanatory and predictor
variables. Due to different metrics in our predictor variables, we added a normalization layer
using Keras preprocessing layers to normalize our data. Next, we added a dense layer with a
linear activation function to produce our predicted spread values. We used the Adam Optimizer
and set our loss function to mean absolute error when we compiled the model and set our
learning rate equal to .001.

Next, we fit our model to the training data using an epoch size of 220 and a batch size
equal to 32. Evaluating our model from our test data, we ended up with a mean absolute error of
1.22395 which ended up being the lowest out of our Spread models. We also ended up with an
adjusted R squared of .987 in our final model which was identical to our baseline and backwards
linear models for spread.

Spread Model MAE
Baseline Linear 1.25859
Backwards Selection Linear 1.25857
Baseline Polynomial 1.24982
Backwards Selection Polynomial 1.24985
K-Nearest Neighbors 2.03488
Neural Network 1.22395

Best Model for Spread
The Spread model which we found to produce the lowest MAE was the deep learning
neural network linear regression. This model produced a mean absolute error of 1.22395 when
used on our testing data. Our Spread model training summary is as follows:

B. Total

Methodology for Total

To be exhaustive in our model creation efforts, we created similar linear and polynomial
models to the ones described above in the Spread section. The same predictors were used as
above (all possible aside from idGame, dateGame, Spread, TEAM_ID_A, TEAM_ID_H, Away,
Home, PTS_H and PTS_A) to create baselines, and then backwards selection was used to cut
down on the insignificant predictors. Again, training data was used to create the models and test
data yielded the mean absolute errors that we used to evaluate each model.
Once again, we attempted to create a KNN model for predicting Total. Similar to the
KNN model for Spread, we used the scaled dataset to create the model for Total. Once the data
was split into test and train data, we looped through 20 different k values to find the optimal k
value for our dataset using the knn.reg() function from the FNN package in R and by calculating
for the smallest mean absolute error for the 20 k values. From this loop we found the optimal
KNN to regression model to have a k=1 and a mean absolute error of 7.94477. This mean
absolute error for this model was larger than the most of the other models we used to predict
total, so we did not consider this model for our predictions.

Additionally, since Total is an array of positive variables that can be seen as random and
independent, we wanted to run a Poisson regression. Poisson regressions models assume that
error follows a Poisson distribution rather than a normal curve, so we figured it could be
significant. We built the model using all of the variables except: idGame, dateGame, Spread,
TEAM_ID_A, TEAM_ID_H, Away, Home, PTS_H, PTS_A. We dropped these variables because
they added heavy collinearity that immensely skewed the data. Once we ran the regression, we
calculated an mean absolute error of 9.48972. We also ran a Quasi-Poisson regression to double
check that our assumption of constant variance equaling the mean didn’t negatively impact our
result. Our mean absolute error for the Quasi-Poisson was identical, and thus we decided to scrap
it and assume very minute differences.

Our model that gave the lowest mean absolute error for total was using the same deep
learning neural network linear regression we created using Tensorflow for spread. The procedure
remains nearly identical but instead of using Spread as our explanatory we exchanged it with
Total. We started off breaking eighty percent of our dataset in to our training data and the rest for
our test data. Next, we split our training and test data into features and labels to isolate our
explanatory and predictor variables.

Due to different metrics in our predictor variables, we added a normalization layer using
Keras preprocessing layers to normalize our data. Next, we added a dense layer with a linear
activation function to produce our predicted Total values. In order to correct for the increased
standard deviation in our total model we adjusted the learning rate to 0.01. Next, we fit our
model to the training data using an epoch size of 300 and a batch size equal to 32. Evaluating our
model from our test data, we ended up with a mean absolute error of 7.58238 which also ended
up being our lowest mean absolute error for our total models as well as an R squared of .7714.

Total Model MAE
Baseline Linear 7.76662
Backwards Selection Linear 7.78282
Baseline Polynomial 7.86682
Backwards Selection Polynomial 7.86856
K-Nearest Neighbors 7.94477
Poisson 9.48972
Neural Network 7.58238

Best Model for Total
The Total model which we found to produce the lowest MAE was the deep learning
neural network linear regression. This model produced a mean absolute error of 7.58238 when
used on our testing data. Our Total model training summary is as follows:

C. OREB

Methodology for OREB

OREB is an array of positive variables that can be seen as random and independent, so we
also ran a Poisson regression. We built the model using all the variables except: idGame,
dateGame, Spread, TEAM_ID_A, TEAM_ID_H, Away, Home, OREB_H, and OREB_A. We
dropped these variables because the collinearity negatively impacted the data. Once we ran the
regression, we calculated an mean absolute error of 3.55871. We also ran a Quasi-Poisson
regression to check the assumption of variance equaling the mean. Our mean absolute error for
the Quasi-Poisson was identical to the ‘regular’ Poisson regression so we decided to not mind it.
Like with Spread and Total, we also tried baseline and backwards selected linear and
polynomial models to predict OREB. This time we excluded the same predictors as above, as
well as OREB_H and OREB_A due to the fact that they would be perfect predictors. Again, we
created these four models with the training data and then evaluated them based on the mean
absolute errors we calculated with the test data. Similar to the Spread and Total, we also created a
deep learning neural network linear regression and found a mean absolute error of 2.46655

One model we once again attempted to create for predicting OREB was KNN. Similar to
the previous KNN models we used the scaled dataset to model for OREB. We followed a similar
loop as the previous models by looping through 20 values. From this loop we found the optimal
KNN to regression model to have a k=1 and a mean absolute error of 2.81686. This mean
absolute error for this model was slightly larger than some of the other models we used to predict
OREB so we did not consider this model for our predictions.

OREB Model MAE
Baseline Linear 2.42870
Backwards Selection Linear 2.42836
Baseline Polynomial 2.39502
Backwards Selection Polynomial 2.39375
K-Nearest Neighbors 2.81686
Poisson 3.55871
Neural Network 2.46655

Best Model for OREB

The OREB model which we found to produce the lowest MAE was the backwards
selected stepwise polynomial regression model. This model produced a mean absolute error of
2.39375 when used on our testing data. Our OREB model is as follows:
Coefficients:
Estimate Std. Error t value Pr(>|t|)
(Intercept) -2.046e+03 1.271e+03 -1.610 0.107532
poly(heightH, 2, raw = T)1 4.229e+01 2.263e+01 1.868 0.061819 .
poly(heightH, 2, raw = T)2 -2.694e-01 1.447e-01 -1.862 0.062641 .
poly(heightA, 2, raw = T)1 9.576e+00 2.441e+01 0.392 0.694924
poly(heightA, 2, raw = T)2 -6.028e-02 1.561e-01 -0.386 0.699319
poly(weightH, 2, raw = T)1 9.240e-01 8.995e-01 1.027 0.304434
poly(weightH, 2, raw = T)2 -2.156e-03 2.077e-03 -1.038 0.299360
poly(weightA, 2, raw = T)1 -2.383e-01 9.261e-01 -0.257 0.796936
poly(weightA, 2, raw = T)2 5.265e-04 2.142e-03 0.246 0.805867
poly(ageH, 2, raw = T)1 -2.360e+00 1.210e+00 -1.950 0.051292 .
poly(ageH, 2, raw = T)2 4.296e-02 2.307e-02 1.862 0.062667 .
poly(ageA, 2, raw = T)1 -9.453e-01 1.231e+00 -0.768 0.442510
poly(ageA, 2, raw = T)2 1.614e-02 2.347e-02 0.688 0.491786
poly(Rivalry.Score, 2, raw = T)1 -1.614e-02 4.760e-02 -0.339 0.734589
poly(Rivalry.Score, 2, raw = T)2 -1.463e-04 3.385e-03 -0.043 0.965540
13
poly(avg_plus_minus_H, 2, raw = T)1 -5.541e-02 5.782e-02 -0.958 0.338031
poly(avg_plus_minus_H, 2, raw = T)2 5.072e-03 2.379e-03 2.132 0.033119 *
poly(FG_PCT_H, 2, raw = T)1 1.164e+02 7.914e+01 1.471 0.141451
poly(FG_PCT_H, 2, raw = T)2 -9.046e+01 4.329e+01 -2.090 0.036748 *
poly(FG3_PCT_H, 2, raw = T)1 -5.140e+00 4.862e+00 -1.057 0.290584
poly(FG3_PCT_H, 2, raw = T)2 7.933e+00 6.573e+00 1.207 0.227555
poly(avg_plus_minus_A, 2, raw = T)1 -4.219e-02 5.859e-02 -0.720 0.471551
poly(avg_plus_minus_A, 2, raw = T)2 -2.871e-03 2.263e-03 -1.269 0.204493
poly(FG_PCT_A, 2, raw = T)1 -3.685e+01 8.580e+01 -0.429 0.667618
poly(FG_PCT_A, 2, raw = T)2 1.021e+00 5.005e+01 0.020 0.983723
poly(Shooting_Advantage, 2, raw = T)1 -3.235e+01 3.522e+01 -0.918 0.358442
poly(Shooting_Advantage, 2, raw = T)2 7.538e+00 8.052e+00 0.936 0.349276
poly(block_H, 2, raw = T)1 -2.396e-02 7.593e-02 -0.316 0.752401
poly(block_H, 2, raw = T)2 4.644e-03 6.297e-03 0.738 0.460861
poly(block_A, 2, raw = T)1 -7.892e-02 8.152e-02 -0.968 0.333094
poly(block_A, 2, raw = T)2 3.250e-03 7.104e-03 0.458 0.647297
poly(TO_H, 2, raw = T)1 -7.682e-02 8.634e-02 -0.890 0.373670
poly(TO_H, 2, raw = T)2 6.573e-04 2.895e-03 0.227 0.820405
poly(TO_A, 2, raw = T)1 9.518e-02 8.421e-02 1.130 0.258475
poly(TO_A, 2, raw = T)2 -3.562e-03 2.802e-03 -1.271 0.203846
poly(Power_Ranking_A, 2, raw = T)1 1.561e-02 3.227e-02 0.484 0.628623
poly(Power_Ranking_A, 2, raw = T)2 -9.455e-04 9.666e-04 -0.978 0.328052
poly(dreb_H, 2, raw = T)1 -4.986e-01 1.280e-01 -3.897 9.99e-05
***
poly(dreb_H, 2, raw = T)2 -6.547e-04 1.829e-03 -0.358 0.720440
poly(dreb_A, 2, raw = T)1 -4.442e-01 1.138e-01 -3.903 9.75e-05
***
poly(dreb_A, 2, raw = T)2 -2.604e-03 1.661e-03 -1.568 0.117001
poly(shots_missed_H, 2, raw = T)1 1.865e-01 1.158e-01 1.610 0.107435
poly(shots_missed_H, 2, raw = T)2 3.651e-03 1.066e-03 3.425 0.000625
***
poly(shots_missed_A, 2, raw = T)1 3.550e-01 1.242e-01 2.857 0.004308 **
poly(shots_missed_A, 2, raw = T)2 2.091e-03 1.137e-03 1.839 0.066031 .
poly(Fast_Break_H, 2, raw = T)1 2.881e-01 1.804e-01 1.597 0.110313
poly(Fast_Break_H, 2, raw = T)2 -1.006e-02 6.611e-03 -1.522 0.128003
---
Signif. codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
Residual standard error: 3.094 on 2716 degrees of freedom
Multiple R-squared: 0.6765, Adjusted R-squared: 0.671
F-statistic: 123.5 on 46 and 2716 DF, p-value: < 2.2e-16

Making Predictions
In order to make our predictions, we initially tried averaging each of our predictors for
every home/away team matchup possible in the current season but quickly realized not every

home/away matchup had occurred to that point. Instead, we averaged all predictor values for
each team when they were away and at home and then merged those with the prediction dataset
which included the matchups we were predicting in order to then run our predictions. We then
used our best models to create our predictions. Once those predictions were made all the average
data was removed from the prediction file to be in the correct format for submission.


References and Data Sources
https://www.teamrankings.com/nba/ - This is the site we used to find fast break points for
each NBA team during the 2021-2023 seasons.
https://nba.nbcsports.com/2022/10/18/nba-power-rankings-warriors-start-season-on-top-wi
th-bucks-on-their-heels/ - This is the site we used to find the preseason power rankings for
each NBA season from 2021-2023.
https://www.rdocumentation.org/packages/nbastatR/versions/0.1.10131 - This is the
documentation page for the nbastatR package we used for much of our data collection and
cleaning.
https://www.landofbasketball.com/ - This is where all data used in the calculation for the
Rivalry_Score variable was pulled from.
