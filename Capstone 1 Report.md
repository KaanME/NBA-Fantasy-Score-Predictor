# NBA Fantasy Score Predictor
## Introduction

The legality of sports betting has been an issue that most sports organizations have been avoiding for quite some time now. Now, the NBA continues its reign as the most progressive league by taking an active stance on trying to push for legal, regulated and monitored sports betting. Whether the bill passes on a federal or state by state basis, there is going to be a big push trying to accurately estimate various outcomes in a game. The most fundamental of those would be setting lines on what team is going to win and by how much. Another popular outcome that people will likely be able to bet on is how many points/assists/rebounds, or any other statistic, that a certain player will hit on any given night. 
Daily Fantasy companies like DraftKings, FanDuel, and Draft predict daily fantasy scores every night, and now the NBA along with whoever they partner with will be looking for accurate projections for these numbers. There are many factors that go into modeling something as complex as player performance, such as their opponent, if they are playing at home or away, their past performance, etc. More accurate projection will lead to increased confidence on the lines to bet on. This could increase user participation as well as profit margin for companies that looking to benefit are from the legalization of sports gambling. 

## Data Acquisition and Cleaning
The data used for this project was all collected from the NBA.com API, using the python package nba_py. This package makes it very simple to extract data from the endpoints of the API and return them in a JSON or a pandas DataFrame format. Originally, I collected DraftKings data from RotoGuru.com, so I can use the salary values as well, but the data only goes back to the 2014-15 season. The fantasy scores can be derived from the NBA data using the DraftKings fantasy score system and the salaries don't really play a role in predicting the score, but could definitely be applied in future applications for optimized lineup selection. To see how to I scraped the data, see my Data Acquisition notebook, which also goes into detail of how I collected the NBA data. To see how I cleaned the RotoGuru data, head over to my [DraftKings Data Wrangling](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/DraftKings%20Data%20Wrangling.ipynb) notebook . 
Now let's dig into the process of acquiring and cleaning up the NBA data that I actually used. I focused on collecting stats since the turn of the century. There were 2 sets of data to be collected, game logs of every player for each season, as well as game logs for every game of each team. 

### Player Gamelogs
As mentioned earlier, the steps required to actually acquire the data was pretty straightforward used the GameLog function in the league module of the nba_py package. This returned a data frame containing all the box score stats of individual players for an entire season, as well as fields like team name, the matchup, the game date, etc.  All I had to do was loop through the seasons I was interested in and append each returned data frame to the previous. I saved the resulting data into a csv file that I can easily access later. The details of the process can be seen here,  [Data Acquisition](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/Data%20Acquisition.ipynb). Cleaning the data took required a bit more work.

Cleaning the data took required a bit more work. I will be summarizing the steps performed in my [NBA Wrangling](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/NBA%20Wrangling.ipynb) notebook. I started out by leaning up the syntax of the columns, making sure the statistic columns were numeric type, and converting the game_date column into a timestamp, to give me the option of analyzing the data in a time series. The main null values occurred in the shooting percentage field, and were because of division by 0. If the player didn't attempt any shots, I filled the percentage values with 0 as well. I added a few other trivial columns as well, such as a column representing whether the player played at home or on the road, the opponent team  name, and made converted the win loss column to binary. One decision I did make was to change the team abbreviations for old school teams like the Seattle Supersonics and Vancouver Grizzlies to the present day counterparts, the OKC Thunder and Memphis Grizzlies.  

The fun started when I started to derive statistics columns on my own. Adding a fantasy score column was simple enough, using the DraftKings scoring system to come up with the formula and applying to each row. From there, I decided to add columns for advanced statistics as well. Even though they do not matter much in the calculation of fantasy score, they could serve well for projection purposes. Instead of scraping game by game through Basketll-Reference.com, I used the formulas they provide here, [Basketball Reference Glossary](https://www.basketball-reference.com/about/glossary.html).  I added the following stats: Ast%, Blk%, Reb%, Stl%, Tov%, EFG%, and TS%. Explanations can be found in the notebook and the link. In order to use the team total stats, I had to groupby season, team, and gameID. The same process was used for the opponent totals, except grouping by opponent instead of team. Once I added all of the relevant advanced stats, my player logs dataset was ready to use.

### Team Gamelogs
Acquiring the Team game logs was almost exactly the same process as getting the player game logs. In fact, I just put all the steps used for the player logs into one function and applied it to the team game logs dataset. There were a few additional steps to be taken however. I didn't like the fact that I couldn't access the opponent's stats for a given game without indexing and slicing the dataset each time. So I created a function that created a data frame of the opponents stats index in the order of the team logs. Joining the opponent stats to the team stats enabled me to analyze any defensive metrics I want. From there, I added columns for the amount of possessions each team had using the formula from Dean Oliver's great book, Basketball On Paper. I use the possession stats to add pace, offensive efficiency, and defensive efficiency values as well. All of this can be found in the same [NBA Wrangling](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/NBA%20Wrangling.ipynb) notebook. 

## Data Exploration
There are several different factors to consider when looking at player performance. I explored 5 different aspects of the game, the amount of minutes played, the evolution of 3 point shooting and offense, the pace of play, defensive impact on statistics, and home court advantage for players. The following notebooks will show the process of the exploration in detail, [NBA Exploration](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/NBA%20Exploration.ipynb) and [NBA Inferential Statistics](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/NBA%20Inferential%20Statistics.ipynb). 
### Minutes Played
The first think I decided to look at in the data in the correlation between minutes played and the fantasy score of a player. I decided to use fantasy score in order to take into account all the counting stats. Intuitively, it makes a lot of sense that the more minutes a player plays, the more time they have to pick up counting stats. Figure 1 shows a plot of the average amount of minutes a player averaged for a season compared to their average Fantasy Score.

<p align="center"> 
<img src="/assets/min_per_fscore.png">
<p align="center"><b> Figure 1: Season average fantasy score against average minutes they played</b>
</p>

There is a clear correlation between the amount of minutes a player plays and the amount of stats that they are able to accumulate. This is only part of the puzzle however, and the style of play and the teams playing during those minutes are important factors as well.

### 3 Point Shooting and Offense

Any fan of the NBA knows that the amount of 3 pointers taken has been going up for quite some time now. Taking a quick look at the distribution of the number of 3 pointers taken by teams in the year 2000 vs. 2017 gives us a clear understanding of the difference. Figure 2 shows the stark difference, where the average team in the 2000-2001 season took 13.71 threes a game and in the most recent 2017-18 season, the average team took more than double at 29.0 threes a game. This is a very significant jump.  In fact, in Figure 3 we can see that the team that averaged the least amount of 3's in 2017 shot more threes than the league leader 18 year ago. 

<p align="center"> 
<img src="/assets/fg3a_2000_2017.png">
<p align="center"><b> Figure 2: Team 3 point attempt distribution in 2000 and 2017</b>

</p>

<p align="center"> 
<img src="/assets/min_max_3s.png">
<p align="center"><b> Figure 3: League highest and lowest average 3 pointers attempted by team</b>
</p>

The increase in 3 point attempts throughout the league has several different important impacts. One of which is the increased offensive efficiency that comes along with shooting more three pointers. Figure 4 below shows the positive correlation that the number of 3 pointers has with offensive efficiency. We can also see, by the color gradient, that the most efficient offenses mostly come from teams in the past 5 years. 


<p align="center"> 
<img src="/assets/off_eff_per_3s.png">
<p align="center"><b> Figure 4: Offensive Efficiency per 3 pointers attempted with a color gradient for the season</b>
</p>

This information tell us that a team's shot selection should be taken into consideration when trying to project a player's score. If they are in a system where 3 pointers are a focus, like the Houston Rockets of the past few years, then they are part of an efficient offense and are more likely to score more points. This brings us to our next factor.

### Pace of Play
The pace that a team plays at will also be a large indicator of the amount of opportunities any individual player will have to rack up more stats.   

<p align="center"> 
<img src="/assets/pace_over_time.png">
<p align="center"><b> Figure 5:  Change in the average pace of the league since the 2000-01 season</b>
</p>

Figure 5 shows the change in the average pace of the league since the turn of the century. Clearly, the league has gotten faster, and this may be due to the increase 3 pointers everyone is taking. Taking a look at Figure 6, there is clearly a strong correlation between the amount of 3 pointers taken and the pace that team plays at. 

<p align="center"> 
<img src="/assets/3s_pace.png">
<p align="center"><b> Figure 6: 3 pointers attempted vs. pace of play</b>
</p>

Not only are teams taking more threes, they are playing faster, and offences are getting more efficient. Of course, comparing across years doesn't necessarily inform a projection for a player today. However, understanding the correlation between all these factors gives us a sense of what to consider when modeling player performance.

### The Defense Effect
There's two sides to the game, and we've only taken a look at the offensive end of the ball. The opposing teams defense will play a large factor in how many points a player will score on a given night. To test this, I wanted to compare how much higher or lower than their average that a player scored in fantasy against the best and worst defenses. I split up the team stats into the top 15 and bottom 15 defenses for each season based on defensive efficiency. From there, I filtered the player logs dataset by the games played against the top and bottom defenses. The details of this process are located in the [NBA Inferential Statistics](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/NBA%20Inferential%20Statistics.ipynb). 
I tested the hypothesis that the average fantasy score against the best and worst defenses are equal. The total distribution is shown in Figure 7.

<p align="center"> 
<img src="/assets/fscore_deviation.png">
<p align="center"><b> Figure 7: Distribution of the difference between a players fantasy score from their season average</b>
</p>

The average deviation is predictably 0, since the total differences will cancel each other out in the end to get the actual average of each player. What's interesting is that the median value comes out to be -.767, meaning a player is more likely score under his average for the season.  The average is pulled back up to 0 because there is no upper limit to a fantasy score (permitting the game clock), however the lowest a player can score is 0. The average deviation against the top 15 defenses came out to be -.547 points and the +.550 against the bottom half defenses. Running a t-test against these two distributions provided a t-statistic of -36.66 and a p value of 1.16E-293, so essentially 0. The allows us to reject the null hypothesis and  state with almost 100% certainty that playing against a better defense effects statistical performance.

### Home Court Advantage

Using the same methods to test performance against different defenses, I tested the null hypothesis that a players average deviation from their season average fantasy score is equal whether they are at home or on the road. The resulting averages are as follows:

Average deviation from season fantasy score average at home: +0.439 
Average deviation from season fantasy score average on the road: -0.440

Again, not a surprising result, but I wanted to confirm in with another t-test. The resulting t statistic was 29.256 with another extremely small p-value of 5.95E-188. The null hypothesis is rejected with nearly 100% certainty and home court advantage does in fact appear to exist.

## Modeling

After acquiring, cleaning, and exploring the data, the next step is to find an appropriate model for the data. Since we are trying to predict the fantasy score of a player for a given night, several different regression techniques were examined including ridge, lasso, decision trees and random forest. I was also curious about the data's ability to predict whether or not a player would perform better or worse than his season average to date. While the regression method would help the client set lines for betting, the logistic model could help the user in setting lineups and placing bets. All of the models and code can be found in the [Models Notebook](https://github.com/KaanME/NBA-Fantasy-Score-Predictor/blob/master/Models.ipynb).

### Data Pre-Processing and Feature Selection

Before we can actually fit the data to a model, there are a few steps to be performed to pre-process the data, as well as a couple more wrangling steps. The data in the exploration looked at season averages for the most part, but our goal is to predict game by game. The last 3 games of a player might be a better indicator of how they are going to perform compared to the season average. Therefore, I looked at how different windows of statistical averages affected the prediction of the current game. That is, I created separate datasets that looked at only the game before, the average stats of the the last 3 games, as well as the last 5, 7, 10, 15, and 20 games. 

In order for our models to provide us with valuable information, the data also had to be scaled. This was accomplished through the use of sklearn's StandardScaler function. From there, the appropriate features need to be selected. The dataset consists of over 40 features, and  in order to save on computation time and the ill effects of multi-colinearity, I wanted to bring that number down to 10. 
I chose to use manually selected features based on my the data exporation as well as use the SelectKBest algorithm in sklearn and compare the results for each. The features I selected come from the variable that make up the fantasy score, the strong correlation between fantasy score and minutes played, as well as the opponents defense, the teams pace, and whether or not they played at home. The following table shows the features for the data using the avergae stats of the last 5 games, the 5 will change to correspond to the number of games being looked at. 

<p align="center"><b> Table 1:  Manually Picked Features</b>
<table border="1" class="dataframe"> <thead> <tr style="text-align: center;">
 <th></th> <th>fts_picked</th> </tr> </thead> <tbody> 
 <tr> <th>0</th> <td>mp_l5</td> </tr> 
 <tr> <th>1</th> <td>pts_l5</td> </tr> 
 <tr> <th>2</th> <td>ast_l5</td> </tr> 
 <tr> <th>3</th> <td>reb_l5</td> </tr> 
 <tr> <th>4</th> <td>stl_l5</td> </tr> 
 <tr> <th>5</th> <td>blk_l5</td> </tr> 
 <tr> <th>6</th> <td>tov_l5</td> </tr> 
 <tr> <th>7</th> <td>fg3m_l5</td> </tr> 
 <tr> <th>8</th> <td>opp_def_eff_l5</td> </tr> 
 <tr> <th>9</th> <td>tm_pace_l5</td> </tr> 
 <tr> <th>10</th> <td>home</td> </tr> </tbody> </table>

I also applied the SelectKBest algorithm to each of the datasets to get features that have the highest F-statistic, using sklearn's f_regression scoring method. I looped through each dataset and found the best 10 features for each, shown in the table below:

<p align="center"><b> Table 2:  Select KBest Features</b>
<table border="1" class="dataframe"> <thead>   <tr style="text-align: right;">   <th></th>   <th>1</th>   <th>3</th>   <th>5</th>   <th>7</th>   <th>10</th>   <th>15</th>   <th>20</th>   </tr> </thead> <tbody>   <tr>   <th>0</th>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   </tr>   <tr>   <th>1</th>   <td>mp_l1</td>   <td>mp_l3</td>   <td>mp_l5</td>   <td>mp_l7</td>   <td>mp_l10</td>   <td>mp_l15</td>   <td>mp_l20</td>   </tr>   <tr>   <th>2</th>   <td>fgm_l1</td>   <td>fgm_l3</td>   <td>fgm_l5</td>   <td>fgm_l7</td>   <td>fgm_l10</td>   <td>fgm_l15</td>   <td>fgm_l20</td>   </tr>   <tr>   <th>3</th>   <td>fga_l1</td>   <td>fga_l3</td>   <td>fga_l5</td>   <td>fga_l7</td>   <td>fga_l10</td>   <td>fga_l15</td>   <td>fga_l20</td>   </tr>   <tr>   <th>4</th>   <td>ftm_l1</td>   <td>ftm_l3</td>   <td>ftm_l5</td>   <td>ftm_l7</td>   <td>ftm_l10</td>   <td>ftm_l15</td>   <td>ftm_l20</td>   </tr>   <tr>   <th>5</th>   <td>fta_l1</td>   <td>fta_l3</td>   <td>fta_l5</td>   <td>fta_l7</td>   <td>fta_l10</td>   <td>fta_l15</td>   <td>fta_l20</td>   </tr>   <tr>   <th>6</th>   <td>dreb_l1</td>   <td>dreb_l3</td>   <td>dreb_l5</td>   <td>dreb_l7</td>   <td>ft_pct_l10</td>   <td>ft_pct_l15</td>   <td>ft_pct_l20</td>   </tr>   <tr>   <th>7</th>   <td>reb_l1</td>   <td>tov_l3</td>   <td>tov_l5</td>   <td>tov_l7</td>   <td>tov_l10</td>   <td>tov_l15</td>   <td>tov_l20</td>   </tr>   <tr>   <th>8</th>   <td>pts_l1</td>   <td>pts_l3</td>   <td>pts_l5</td>   <td>pts_l7</td>   <td>pts_l10</td>   <td>pts_l15</td>   <td>pts_l20</td>   </tr>   <tr>   <th>9</th>   <td>fscore_l1</td>   <td>fscore_l3</td>   <td>fscore_l5</td>   <td>fscore_l7</td>   <td>fscore_l10</td>   <td>fscore_l15</td>   <td>fscore_l20</td>   </tr> </tbody> </table>

The table shows that the top 10 features are pretty much the same for each different number of games. When the number of games swtiches from 7 to 10, defensive rebounds (dreb_) is replaced with free throw percentage (ft_pct_), other than that the features are all identical. The number of field goals and free throws attempted and made are all selected features, along with the average fantasy score leading up to the game, and minutes, points and turnovers as well. To clarify, fscore_exp is the expanding average fantasy score for each player resetting at the beginning of each season. 

The 2017-18 season will be left out as the test set and the target variable will the fantasy score (fscore).

### Comparing Regression Models

Now that we've selected our features and scaled our values, we can start testing out different regression models to see which one performs best. Since we are looking at 7 different datasets, the goal is to first find the number of matches that provides the best estimate. Then tune the parameters of the best model to get the best results. The scoring method to be used will be the mean squared error regression loss. Squaring the mean squared error will give us the average number of points the model is off by compared to the actual values. 

Now that we've selected our features and scaled our values, we can start testing out different regression models to see which one performs best. Since we are looking at 7 different datasets, the goal is to first  find the number of matches that provides the best estimate. Then tune the parameters of the best model to get the best results. The scoring method to be used will be the mean squared error regression loss. Squaring the mean squared error will give us the average number of points the model is off by compared to the actual values.

Since we are looking to identify the dataset that works best, I used the default parameters for all of the models to see how they fared with the different data. I looked at several different methods including the standard linear regression, 2 regularized methods in Ridge and Lasso regression, as well as looking at a decision tree and random forest.  I used 5-fold cross-validation for each method and averaged the negative mean squared error score for each one. I did this for both the KBest set of features and the features I chose manually. The following 2 tables show the average score for both the training and test sets for each method tested.

<p align="center"><b> Table 3:  KBest Features Train and Test Average Errors</b>
<table border="1" class="dataframe"> <thead>   <tr style="text-align: right;">   <th></th>   <th>ols_train</th>   <th>ols_test</th>   <th>ridge_train</th>   <th>ridge_test</th>   <th>lasso_train</th>   <th>lasso_test</th>   <th>tree_train</th>   <th>tree_test</th>   <th>forest_train</th>   <th>forest_test</th>   </tr> </thead> <tbody>   <tr>   <th>1</th>   <td>9.432398</td>   <td>9.423937</td>   <td>9.432398</td>   <td>9.423930</td>   <td>9.504568</td>   <td>9.496406</td>   <td>13.769615</td>   <td>13.792870</td>   <td>10.016866</td>   <td>9.905068</td>   </tr>   <tr>   <th>3</th>   <td>9.365836</td>   <td>9.352333</td>   <td>9.365835</td>   <td>9.352326</td>   <td>9.445799</td>   <td>9.438040</td>   <td>13.729015</td>   <td>13.747288</td>   <td>10.004357</td>   <td>9.913258</td>   </tr>   <tr>   <th>5</th>   <td>9.360033</td>   <td>9.348744</td>   <td>9.360032</td>   <td>9.348738</td>   <td>9.434975</td>   <td>9.428140</td>   <td>13.751196</td>   <td>13.921481</td>   <td>9.978022</td>   <td>9.882173</td>   </tr>   <tr>   <th>7</th>   <td>9.365096</td>   <td>9.359807</td>   <td>9.365095</td>   <td>9.359800</td>   <td>9.436364</td>   <td>9.431789</td>   <td>13.820169</td>   <td>13.939535</td>   <td>9.911670</td>   <td>9.875239</td>   </tr>   <tr>   <th>10</th>   <td>9.381708</td>   <td>9.372892</td>   <td>9.381708</td>   <td>9.372880</td>   <td>9.446511</td>   <td>9.437335</td>   <td>13.873281</td>   <td>14.162114</td>   <td>9.951670</td>   <td>9.824104</td>   </tr>   <tr>   <th>15</th>   <td>9.409444</td>   <td>9.408252</td>   <td>9.409444</td>   <td>9.408220</td>   <td>9.468973</td>   <td>9.467752</td>   <td>14.002799</td>   <td>14.284953</td>   <td>9.921099</td>   <td>9.901394</td>   </tr>   <tr>   <th>20</th>   <td>9.437600</td>   <td>9.434441</td>   <td>9.437600</td>   <td>9.434393</td>   <td>9.495126</td>   <td>9.492944</td>   <td>14.090772</td>   <td>14.396226</td>   <td>9.944842</td>   <td>9.961960</td>   </tr> </tbody> </table>

<p align="center"><b> Table 4:  Manually Picked Features Train and Test Average Errors</b>
<table border="1" class="dataframe"> <thead>   <tr style="text-align: right;">   <th></th>   <th>ols_train</th>   <th>ols_test</th>   <th>ridge_train</th>   <th>ridge_test</th>   <th>lasso_train</th>   <th>lasso_test</th>   <th>tree_train</th>   <th>tree_test</th>   <th>forest_train</th>   <th>forest_test</th>   </tr> </thead> <tbody>   <tr>   <th>1</th>   <td>10.593620</td>   <td>10.601052</td>   <td>10.593620</td>   <td>10.601051</td>   <td>10.752367</td>   <td>10.752141</td>   <td>15.399055</td>   <td>15.532806</td>   <td>11.233325</td>   <td>11.405555</td>   </tr>   <tr>   <th>3</th>   <td>9.812464</td>   <td>9.798975</td>   <td>9.812464</td>   <td>9.798975</td>   <td>9.962610</td>   <td>9.925484</td>   <td>14.341132</td>   <td>14.460830</td>   <td>10.617135</td>   <td>10.679961</td>   </tr>   <tr>   <th>5</th>   <td>9.620792</td>   <td>9.615975</td>   <td>9.620792</td>   <td>9.615975</td>   <td>9.769589</td>   <td>9.738204</td>   <td>14.145577</td>   <td>14.192733</td>   <td>10.494153</td>   <td>10.598532</td>   </tr>   <tr>   <th>7</th>   <td>9.543132</td>   <td>9.553289</td>   <td>9.543132</td>   <td>9.553289</td>   <td>9.695275</td>   <td>9.672675</td>   <td>14.059812</td>   <td>14.016581</td>   <td>10.441388</td>   <td>10.515698</td>   </tr>   <tr>   <th>10</th>   <td>9.504331</td>   <td>9.510566</td>   <td>9.504331</td>   <td>9.510566</td>   <td>9.658544</td>   <td>9.629812</td>   <td>14.130930</td>   <td>14.190924</td>   <td>10.422918</td>   <td>10.433071</td>   </tr>   <tr>   <th>15</th>   <td>9.501312</td>   <td>9.516957</td>   <td>9.501312</td>   <td>9.516957</td>   <td>9.655845</td>   <td>9.635474</td>   <td>14.257261</td>   <td>14.228241</td>   <td>10.405058</td>   <td>10.441194</td>   </tr>   <tr>   <th>20</th>   <td>9.529071</td>   <td>9.546412</td>   <td>9.529071</td>   <td>9.546411</td>   <td>9.684158</td>   <td>9.661374</td>   <td>14.397789</td>   <td>14.381092</td>   <td>10.369019</td>   <td>10.436327</td>   </tr> </tbody> </table>

Examining the results from both of the tables, there are some patterns that emerge in each. The values in the KBest table are all slightly lower than the values in the table with hand-picked features.  We can also quickly see that ion tree model is far more inaccurate than the others, being in the 14-15 range in instead of the 9-10 range. Another item to note is the fact that model seems to have performed better on the test set than the training set. This could be due to the training set containing 17 seasons of data and the test set only containing 1 season. The change in the style of play throughout the league over such large span could be another possible cause.  Figures 8 and 9 show the model performances on the test sets compared to each other, excluding the decision tree values.

<p align="center"> 
<img src="/assets/kbest_fts_errors.png">
<p align="center"><b> Figure 8: Kbest Features Average Errors by Dataset</b>
</p>

<p align="center"> 
<img src="/assets/picked_fts_errors.png">
<p align="center"><b> Figure 9: Manually Picked Features Average Errors by Dataset</b>
</p>

In Figure 8, the kbest features minimize their MSE when looking at data from the past 5 games, while the hand-picked features in Figure 9 minimize at 10 and 15 games, with the actual minimum somewhere in between those values. The standard linear regression numbers seem to be missing from the graph, but that is due them almost exactly matching the ridge regression values, so the lines overlap. From these results, the highest accuracy (or lowest error) is obtained with ridge regression applied to the data set looking at the average of the past 5 games. We will use this model and tune it's parameters to see if we can get better results. 

### Hyper-parameter Tuning 
The best accuracy number we obtained with the out of the box sklearn algorithm's was an average error of 9.349 points using a ridge regression. The sklearn algorithm defaults to an alpha value of 1. Using the GridSearchCV function in sklearn, I tested out 4 different alpha values of .01, .1, 1, and 10. The best performing alpha value turned out to be .1. The error that the new model gave was 9.333, which is better than our previous best model, but only by about .1%. This isn't an amazing improvement, but we'll stick with an alpha value of .1. 
This method could be used to test the parameters of all of the other algorithms. Due to time and computing restraints, this report will not go into the parameter tuning of all of those models. 

### Results
The model is trained with 17 seasons of data and now we can look at how it performs for each player specifically.  Figure 10 shows the actual Fantasy scores of James Harden in red, and the predicted values in blue. 

<p align="center"> 
<img src="/assets/harden_act_pred.png">
<p align="center"><b> Figure 10: James Harden Actual Fscore vs. Predicted</b>
</p>

The actual values are a lot more sporadic than the predictions, which explains the somewhat large margin of error. James Harden averaged 55.66 fantasy points in the 2017-18 season and the model predicted his performance on an average of within 9.739 point. This is a 17.5% difference from his season average. However, if you break down the pieces that go into the fantasy score calculation, 9 fantasy points is the equivalent of being within 1 of each of the stats that go into the calculation. If you add one 3 pointer (3 points plus half a point for the made three pointer), 1 assist (1.5pts), 1 rebound (1.25), 1 steal (2), and 1 block (2), as well as 1 less turnover (.5), the prediction would be off by 10.75 points. This is more than the average error in our model, and since the goal is to set lines for sports betting, our model could serve that purpose. Looking at Figure 11, we can see difference between the actual and predicted values, as well as a shaded region showing the average error for James Harden. 

<p align="center"> 
<img src="/assets/harden_resplot.png">
<p align="center"><b> Figure 11: James Harden Actual Fscore vs. Predicted Residual Plot</b>
</p>

Digging in to Figure 11 a little bit more, we can calculate that 62.5% of the predictions are within the shaded area. The percentage over and under the line of no error is a 54.2/46.8 split for Harden.  Table 5 below shows the top 10 players with the highest errors, as well as their fscore standard deviation, average fscore, the percentage of predictions within their average error, and the percent of the predictions over the actual values. 

<p align="center"><b> Table 5:  Top 10 Players by Average Error</b>
<table border="1" class="dataframe"> <thead> <tr style="text-align: right;"> <th></th> <th>avg_diff</th> <th>fscore_std</th> <th>avg_fscore</th> <th>inrange_pct</th> <th>over_pct</th> </tr> </thead> <tbody> <tr> <th>Anthony Davis</th> <td>12.617262</td> <td>15.722370</td> <td>54.020000</td> <td>0.600</td> <td>0.640</td> </tr> <tr> <th>Nikola Jokic</th> <td>11.891398</td> <td>14.593531</td> <td>45.536667</td> <td>0.547</td> <td>0.587</td> </tr> <tr> <th>Rajon Rondo</th> <td>11.130151</td> <td>13.822423</td> <td>27.765385</td> <td>0.569</td> <td>0.462</td> </tr> <tr> <th>LeBron James</th> <td>11.117405</td> <td>13.737475</td> <td>56.887195</td> <td>0.524</td> <td>0.634</td> </tr> <tr> <th>Devin Booker</th> <td>10.973059</td> <td>12.811978</td> <td>39.537037</td> <td>0.593</td> <td>0.481</td> </tr> <tr> <th>D\'Angelo Russell</th> <td>10.870212</td> <td>13.134561</td> <td>30.015625</td> <td>0.562</td> <td>0.479</td> </tr> <tr> <th>Nikola Vucevic</th> <td>10.568266</td> <td>13.466501</td> <td>37.530702</td> <td>0.596</td> <td>0.509</td> </tr> <tr> <th>Nikola Mirotic</th> <td>10.567980</td> <td>12.451307</td> <td>31.013636</td> <td>0.545</td> <td>0.545</td> </tr> <tr> <th>Dwight Howard</th> <td>10.519608</td> <td>12.678648</td> <td>38.314815</td> <td>0.556</td> <td>0.531</td> </tr> <tr> <th>Andre Drummond</th> <td>10.494522</td> <td>12.013915</td> <td>45.644231</td> <td>0.564</td> <td>0.564</td> </tr> </tbody> </table>

We can see that Anthony Davis has the highest error, and that besides Rajon Rondo, every player in the top 10 averaged over 30 fantasy points. In order to see the correlation between the average error and the average fantasy score and the standard deviation of a players fantasy score, take a look at Figure 12. The higher the standard deviation, the higher the average error. The correlation is less strong with the average fantasy score, but it is still apparent. 

<p align="center"> 
<img src="/assets/error_vs_avg_std.png">
<p align="center"><b> Figure 12: Average Error vs. Fscore Standard Deviation and Average Fscore</b>
</p>

### Logistic Regression

Even though the main objective of this project is to predict the fantasy score of a player in a given game, it could also be valuable to predict whether or not a player will outperform his season-to-date average. To do this, I created an "above_avg_fscore" column that is 1 when a player is above his average for the season to date, and 0 if he's below. The training set remained the same as the linear regression methods, as I didn't include this variable in those training sets. I used the 10 best features selected by the SelectKBest algorithm in sklearn, and I trained a LogisticRegression model with the default parameters on each of the datasets (one for each window of averages like before). The following table shows the area under the receiver operating characteristic curve (ROC AUC) score for each of the datasets, with Figure 13 showing the ROC curves for each. This tells us what percent of the time the model correctly predicted whether or not a player would score above their season average fantasy score.

<p align="center"><b> Table 6: ROC AUC Score for Each Dataset</b>
<table border="1" class="dataframe"> <thead>   <tr style="text-align: right;">   <th></th>   <th>ROC AUC Score</th>   </tr> </thead> <tbody>   <tr>   <th>1</th>   <td>0.594052</td>   </tr>   <tr>   <th>3</th>   <td>0.602958</td>   </tr>   <tr>   <th>5</th>   <td>0.604614</td>   </tr>   <tr>   <th>7</th>   <td>0.602125</td>   </tr>   <tr>   <th>10</th>   <td>0.599752</td>   </tr>   <tr>   <th>15</th>   <td>0.593928</td>   </tr>   <tr>   <th>20</th>   <td>0.589409</td>   </tr> </tbody> </table>

<p align="center"> 
<img src="/assets/roc_curves.png">
<p align="center"><b> Figure 13: ROC Curve for Each Dateset</b>
</p>

Much like the linear regression, the 5 game window gives us the best results with a 60.4% accuracy. I used this dataset to tuned the Logistic Regression C parameter as well as the penalty (L1 or L2). Performing a grid search on several iterations of these parameters, a C value of 10 with L1 regularization performed the best. However, the ROC AUC score of this "optimized" model was almost identical to the default version. The ROC curve of the final model and dataset is shown in Figure 14.

<p align="center"> 
<img src="/assets/roc_curve.png">
<p align="center"><b> Figure 14: Final ROC Curve</b>
</p>

Normally, we would want the area under the ROC curve to be higher, and we could dig deeper into several different classifying methods like Stochastic Gradient Decsent and even Neural Networks. This is beyond the scope of this project and can be looked at as future avenues to pursue. However, being able to predict if a player will perform above their season average could provide beneficial to a daily fantasy player because of the ability to pick high performing lineups.

## Limitations

There are a lot of events during the course of an NBA season, and even just during the course of one game, that can affect the statistical performance of a player. A few of the limitations of the data here is that we do not know the positional matchups, or individual defensive ability of the opponent players. A great defensive team could have 1 player that is not so great on that side of ball, and the team playing them could have their best scorer playing the same position. Even though players generally play worse against good defenses, that particular matchup could be exploited. Another limitation of the data is that we do not have any injury information. A player that is out with a minor injury will come back to the court with minimal ill effects to his game. A player coming back from knee surgery could suffer greatly in performance, and we wouldn't be able to capture accurate projections when he returns. Coaching, team chemistry, trades, and a multitude of other variables exist in the real world, but here we stuck to what we can extract from the box scores.

## Next Steps

There is always more data available when it comes to basketball. We can collect data from the 70's, 80's, and 90's to see if that could improve out results. Trying out different models such as Time-Series analysis, SVM Regression, and Deep Learning techniques to achieve better results. Play-by-play data can be collected to see if players who take shots earlier or later in the shot clock tend to perform better. Position specific analysis could provide more accurate predictions as well. 

## Conclusion

There are a lot of factors that go into the statistical output of an NBA player game to game. After acquiring and cleaning the data, we were able to uncover some interesting trends through our data exploration. The minutes a player plays is a major factor in how he performs in terms of box score stats. However, not all minutes are created equal. If the team they are on generates most of its offense from the 3 point line, they are probably part of a more efficient offense that plays fast. If the opponent they are playing against ranks as a quality defensive team, they will have a harder time with filling the box score. Lastly, we confirmed that players do in fact play better at home than they do on the road. 
The models showed us that the most appropriate window of time to look at in order to predict a players statistical performance is his average stats over the last 5 games. an Optimized Ridge Regression provided the most accurate results, with an average error of 9.3 fantasy points per player when testing the model on the 2017-18 season. We also looked at a logistic regression model's ability to predict whether or not a player will outperform their season average. The accuracy achieved was about 60%. There are a lot more methods to try and data to be tested, but this project provides a base line for future endeavors. 
