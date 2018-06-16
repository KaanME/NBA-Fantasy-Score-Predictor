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

<table border="1" class="dataframe"> <thead>   <tr style="text-align: right;">   <th></th>   <th>1</th>   <th>3</th>   <th>5</th>   <th>7</th>   <th>10</th>   <th>15</th>   <th>20</th>   </tr> </thead> <tbody>   <tr>   <th>0</th>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   <td>fscore_exp</td>   </tr>   <tr>   <th>1</th>   <td>mp_l1</td>   <td>mp_l3</td>   <td>mp_l5</td>   <td>mp_l7</td>   <td>mp_l10</td>   <td>mp_l15</td>   <td>mp_l20</td>   </tr>   <tr>   <th>2</th>   <td>fgm_l1</td>   <td>fgm_l3</td>   <td>fgm_l5</td>   <td>fgm_l7</td>   <td>fgm_l10</td>   <td>fgm_l15</td>   <td>fgm_l20</td>   </tr>   <tr>   <th>3</th>   <td>fga_l1</td>   <td>fga_l3</td>   <td>fga_l5</td>   <td>fga_l7</td>   <td>fga_l10</td>   <td>fga_l15</td>   <td>fga_l20</td>   </tr>   <tr>   <th>4</th>   <td>ftm_l1</td>   <td>ftm_l3</td>   <td>ftm_l5</td>   <td>ftm_l7</td>   <td>ftm_l10</td>   <td>ftm_l15</td>   <td>ftm_l20</td>   </tr>   <tr>   <th>5</th>   <td>fta_l1</td>   <td>fta_l3</td>   <td>fta_l5</td>   <td>fta_l7</td>   <td>fta_l10</td>   <td>fta_l15</td>   <td>fta_l20</td>   </tr>   <tr>   <th>6</th>   <td>dreb_l1</td>   <td>dreb_l3</td>   <td>dreb_l5</td>   <td>dreb_l7</td>   <td>ft_pct_l10</td>   <td>ft_pct_l15</td>   <td>ft_pct_l20</td>   </tr>   <tr>   <th>7</th>   <td>reb_l1</td>   <td>tov_l3</td>   <td>tov_l5</td>   <td>tov_l7</td>   <td>tov_l10</td>   <td>tov_l15</td>   <td>tov_l20</td>   </tr>   <tr>   <th>8</th>   <td>pts_l1</td>   <td>pts_l3</td>   <td>pts_l5</td>   <td>pts_l7</td>   <td>pts_l10</td>   <td>pts_l15</td>   <td>pts_l20</td>   </tr>   <tr>   <th>9</th>   <td>fscore_l1</td>   <td>fscore_l3</td>   <td>fscore_l5</td>   <td>fscore_l7</td>   <td>fscore_l10</td>   <td>fscore_l15</td>   <td>fscore_l20</td>   </tr> </tbody> </table>

The table shows that the top 10 features are pretty much the same for each different number of games. When the number of games swtiches from 7 to 10, defensive rebounds (dreb_) is replaced with free throw percentage (ft_pct_), other than that the features are all identical. The number of field goals and free throws attempted and made are all selected features, along with the average fantasy score leading up to the game, and minutes, points and turnovers as well. To clarify, fscore_exp is the expanding average fantasy score for each player resetting at the beginning of each season. 

The 2017-18 season will be left out as the test set and the target variable will the fantasy score (fscore).

### Comparing Regression Models

Now that we've selected our features and scaled our values, we can start testing out different regression models to see which one performs best. Since we are looking at 7 different datasets, the goal is to first find the number of matches that provides the best estimate. Then tune the parameters of the best model to get the best results. The scoring method to be used will be the mean squared error regression loss. Squaring the mean squared error will give us the average number of points the model is off by compared to the actual values. 


## Limitations

There are a lot of events during the course of an NBA season, and even just during the course of one game, that can affect the statistical performance of a player. A few of the limitations of the data here is that we do not know the positional matchups, or individual defensive ability of the opponent players. A great defensive team could have 1 player that is not so great on that side of ball, and the team playing them could have their best scorer playing the same position. Even though players generally play worse against good defenses, that particular matchup could be exploited. Another limitation of the data is that we do not have any injury information. A player that is out with a minor injury will come back to the court with minimal ill effects to his game. A player coming back from knee surgery could suffer greatly in performance, and we wouldn't be able to capture accurate projections when he returns. Coaching, team chemistry, trades, and a multitude of other variables exist in the real world, but we will stick to what we can extract from the box scores.

## Conclusion

There are a lot of factors that go into the statistical output of an NBA player game to game. After acquiring and cleaning the data, we were able to uncover some interesting trends through our data exploration. The minutes a player plays is a major factor in how he performs in terms of box score stats. However, not all minutes are created equal. If the team they are on generates most of its offense from the 3 point line, they are probably part of a more efficient offense that plays fast. If the opponent they are playing against ranks as a quality defensive team, they will have a harder time with filling the box score. Lastly, we confirmed that players do in fact play better at home than they do on the road. 
The next steps in the process will be to train a model taking into consideration everything we've learned from our data exploration. Making use of the advance stats could also create a more reliable model. In the end, we will be able to predict statistical performance for an NBA player on any given night. 
