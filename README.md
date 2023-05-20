# Top Lane Vs. Bot Lane: Who Actually Carries?
### Vishwak Pabba and Manav Jairam

## **Introduction**
League of Legends is an online, multiplayer game where players in teams of 5 battle each other in Summoner's Rift, a massive arena that has an expansive environment and monsters. Players fight against each other to destroy the enemy by toppling turrets and non-playable minions, slowly advancing through 1 of 3 lanes (top, bottom, middle) towards their enemies' nexus crystal. When a team destroys their enemy nexus crystal located in their base, they ultimatly win the game.

The dataset we used for analysis was found on https://oracleselixir.com/tools/downloads, and contained competitive match data from 2022. The dataset included information about the team composition in each competitive match, an individuals game statistics in each respective match, and match details (date, split, game ID, etc.). 

Over the course of the project, we cleaned the dataset for the necessary columns that we would use for analysis and performed a Kolmogrov-Smirnov Test to determine a solution to the question we wish to answer.


## About the Dataset

There were originally 123 columns within the dataset and 149,400 rows of data. Because we only required a subset of the columns for our analysis, we ended up reducing the number of columns to 13.

| Syntax             | Description                            | 
| :---               |    :----:                              |
| gameid             | The unique id for the match                                  | 
| datacompleteness   | Informs whether the data is complete or partial                             |
| game               | The match number in the series of the specific set of games                                  | 
| side               | The side that the individual playing on (red or blue)                                   |
| position           | The position that the individual plays                                 |  
| barons             | The number of times the team killed the Baron Nashor monster                                   |
| opp_barons         | The number of times the opposing team killed the Baron Nashor monster                                  | 
| ban2               | The second character the team banned in the match                                  | 
| kills              | The number of kills the individual got in the match                                   |
| deaths             | The number of times the individual died in the match                                   | 
| assists            | The number of assists the individual got in the match                                   |
| teamkills          | The total number of kills the team of the individual got                                  | 
| teamdeaths         | The number of times the team had died in the match                                  |
| minionkills        | The number of minions the individual killed in the match                                  | 

## **Question** 
The question we decided to study was: Do players that play in the top lane and players that play in the bottom (bot) lane "carry" the same amount during matches? 

We chose this question because we wanted to determine whether the top lane or bot lane played an equal role in the success of their team based off a performance metric, which we calculated using the ratio of their proportion of kills, assists and minion kills to their proportion of deaths. 

Generally, it is a hot topic among League of Legends players on which lane is the most important, so our tests will be to determine which lane does the most for their team in the overall match. We based the performance metric off the popular Kills-Deaths-Assists (KDA) ratio, but instead include the proportion of minions kills in the calculation because players that kill a larger majority of their teams minions tend to also contribute more to the success of their team and perform better.

## Importing Relevant Packages
- **pandas** : Allows us to manipulate tabular data using Python
- **numpy**: Allows us to perform operations with arrays and calculate new columns for the dataframe
- **plotly**: Allows us to create informative and interactive data visualizations and relevant plots

## **Data Cleaning and Exploratory Data Analysis**

### Data Cleaning

There were many missing values in the minionkills column that would negatively affect how we calculate our proportion of minions kills of each player, which would also affect how we calculate our performance metric. To rectify this, we randomly chose values for each of the missing values in the minionkills column using a `quantitative distribution`. We basically binned all the existing minionkills values into 10 bins, and for each missing value we chose a random bin from the 10 available and then a random value within the range of the chosen bin. We added these to a numpy array and applied these values to the missing values in the minionkills column.

Next, to calculate individual proportions we needed to also have a `teamminionkills` and a `teamassists` column, two columns that we were missing from the original dataset that we would also need to calculate proportions for our performance metric. To calculate the total minion kills and assists per team per game, we did a groupby on the `gameid` and `side` columns and applied a `sum` aggregation on the `minionkills` and `assists` column. From there, we merged the resulting groupby dataframe with our original dataframe, using a `left` merge so every row that has the same game ID and side have the same team minion kills and team assists values. 

Next, to calculate the proportions, we wrote a function called `calc_prop` which would take in a individual row, a column that represents an individuals specific game statistic, a column that represent the teams total for the specific game statistic, and return a proportion that represents the contribution an individual has to the team's total game statisitic. If the team had a total of 0, the function would return 0. From there, we applied the function to each row of the dataframe using a `lambda` function, and assigned the resultant proportions to 4 new columns: `prop_kills`, `prop_assists`, `prop_deaths`, `prop_minions`. 

Finally, the last thing we had to do was use boolean indexing on the dataframe to get only the columns that has a `position` value being either `top` or `bot`. 

**The top 5 rows of our resulting dataframe are shown here**: 

<div class="table-wrapper" markdown="block">

| gameid                | datacompleteness   |   game | side   | position   |   barons |   opp_barons |   kills |   deaths |   assists |   teamkills |   teamdeaths |   minionkills |   teamminionkills |   teamassists |   prop_kills |   prop_deaths |   prop_assists |   prop_minions |
|:----------------------|:-------------------|-------:|:-------|:-----------|---------:|-------------:|--------:|---------:|----------:|------------:|-------------:|--------------:|------------------:|--------------:|-------------:|--------------:|---------------:|---------------:|
| ESPORTSTMNT01_2690210 | complete           |      1 | Blue   | top        |        0 |            0 |       2 |        3 |         2 |           9 |           19 |           220 |              1360 |            38 |    0.222222  |      0.157895 |      0.0526316 |       0.161765 |
| ESPORTSTMNT01_2690210 | complete           |      1 | Blue   | bot        |        0 |            0 |       2 |        4 |         2 |           9 |           19 |           208 |              1360 |            38 |    0.222222  |      0.210526 |      0.0526316 |       0.152941 |
| ESPORTSTMNT01_2690210 | complete           |      1 | Red    | top        |        0 |            0 |       1 |        1 |        12 |          19 |            9 |           221 |              1584 |           124 |    0.0526316 |      0.111111 |      0.0967742 |       0.13952  |
| ESPORTSTMNT01_2690210 | complete           |      1 | Red    | bot        |        0 |            0 |       8 |        2 |        10 |          19 |            9 |           299 |              1584 |           124 |    0.421053  |      0.222222 |      0.0806452 |       0.188763 |
| ESPORTSTMNT01_2690219 | complete           |      1 | Blue   | top        |        0 |            0 |       0 |        5 |         2 |           3 |           16 |           241 |              1988 |            14 |    0         |      0.3125   |      0.142857  |       0.121227 |

</div>

### Univariate Analysis

To demonstrate the relationship between the number of kills and the position a player plays, we created the histograms below. The histograms displays the frequeuncy of different number of kills, where the blue histogram represents players in the top lane and the red histogram represents players in the bottom lane. As the histograms demonstrate, players that play in the bottom lane generally have a higher frequency as the number of kills increases, but players that play in top lane consistently have a higher frequency as the number of kills decreases.

<iframe src="assets/kills_uni.html" width=800 height=600 frameBorder=0></iframe>

### Bivariate Analysis

To demonstrate the relationship between the proportion of deaths to the proportion of kills, we created the catterplot below. The individual points represent the relationship between the proportion of deaths to the proportions of killds, where the blue points represents players in the top lane and the red points represents players in the bottom lane. To reduce the clutter of the plot, we grouped by the proportion of kills and got the mean proportion of deaths of each group. This way, we are able to view the relationship for each lane in a more clear manner. Although there is little correlation, it can be reasonably said that players in the top lane generally had an equivalent ratio of their proportion of deaths to their proportion of kills to players in the bottom lane. Although there are some differences between the two groups, the ratios are equivalent for a large majority of the time.

<iframe src="assets/kills_bivar.html" width=800 height=600 frameBorder=0></iframe>

### Interesting Aggregates

We did a groupby on `position` for either top lane or bot lane, and applied two aggregates, `mean` and `median`, to the `kills`, `assists`, `minionkills` and `deaths` columns. This grouped dataframe shows the average number and median value of the game statisitcs we are exploring by the position. As the dataframe shows, the bot lane has a higher average assists and minionkills, but a lower average of kills and deaths. It also shows that the bot lane has a higher median value of kills, assists and minion kills, but a lower median in the deaths column relative to the top lane. 

<div class="table-wrapper" markdown="block">

|                   |   kills |   assists |   minionkills |   deaths |
|:------------------|--------:|----------:|--------------:|---------:|
| ('bot', 'mean')   | 4.2588  |   5.37153 |       256.718 |  2.54763 |
| ('bot', 'median') | 4       |   5       |       256     |  2       |
| ('top', 'mean')   | 2.79474 |   5.02498 |       232.843 |  2.95333 |
| ('top', 'median') | 2       |   4       |       230     |  3       |

</div>

## **Assessment of Missingness**

### NMAR Analysis

The column that could be **Not Missing at Random (NMAR)** is `damagemitigatedperminute` in the League of Legends dataset. This is because we believe that the values in this column depend on the way the data is designed, and not another column in the dataset. Different champions in League of Legends have varying abilities and roles. Some champions are designed to mitigate damage, such as tanks or supports, while others are focused on dealing damage. This leads to inherent variations in the amount of damage mitigated per minute across different champions and roles. 

In order to mitigate this, a `type` column could be introduced that describes a particular quality of each champion. This type column would specify whether the champion mitigates or deals damage, or whether it mitigates it at all. Thus, the damage mitigated can be justified, making the column **Missing at Random (MAR)**. 

<iframe src="assets/baron_missing.html" width=800 height=600 frameBorder=0></iframe>

### Missingness Analysis

The column that could be Not Missing at Random (NMAR) is damagemitigatedperminute in the League of Legends dataset. This is because we believe that the values in this column depend on the way the data is designed, and not another column in the dataset. Different champions in League of Legends have varying abilities and roles. Some champions are designed to mitigate damage, such as tanks or supports, while others are focused on dealing damage. This leads to inherent variations in the amount of damage mitigated per minute across different champions and roles.

In order to mitigate this, a type column could be introduced that describes a particular quality of each champion. This type column would specify whether the champion mitigates or deals damage, or whether it mitigates it at all. Thus, the damage mitigated can be justified, making the column Missing at Random (MAR).

The columns in the League of Legends dataset that contain missing values are baron, opp_baron, and ban2. Therefore, we are conducting two permutation tests against the baron’ column to check whether the missingness in the baron column is MAR or MCAR with respect to the other columns. We will conduct these tests with an alpha value of 0.05 (replace p-values here).

Here, we will be using TVD as our test statistic in the permutation test as we are dealing with categorical values.

- **Null Hypothesis**: There is no significant difference between the distribution of the ‘baron’ column whether the other column is missing or not missing.

- **Alternate Hypothesis**: There is a significant difference between the distribution of the ‘baron’ column whether the other column is missing versus not missing.

These are the following p-values from the experiments that we conducted: 

`opp_barons` : 0.04

<iframe src="assets/baron_missing.html" width=800 height=600 frameBorder=0></iframe>

`ban3` : 0.82

<iframe src="assets/bm.html" width=800 height=600 frameBorder=0></iframe>

Our observation while testing for missing values was that we were not able to get accurate results where we could reject the null hypothesis for most columns we chose to check for missingness. We noticed that most columns were either MD or MCAR, and not necessarily totally dependant on another column. Hence, we could not get a statistically significant conclusion based on this test.

## Hypothesis Testing

### Hypotheses

To answer our original question of which lane "carries" their team more, we will conduct a hypothesis test using a significance level of 𝛼 = 0.05. Our hypotheses are below:

- **Null Hypothesis** : There is no significant difference between the distributions of the performance statistic between players that play top lane and players that play bottom lane. 

- **Alternate Hypothesis** : There is a significant difference between the distributions of the performance statistic between players that play top lane and players that play bottom lane.

### Making a Performance Metric Column

We added a new column called `perf_metric` which represents the calculated performance metric for each player. We calculated this value using a ratio of proportion of kills, assists, and minion kills to proportion of deaths. 

If the proportion of deaths is 0, then we would only calculate the sum of the proportion of kills, assists, and minion kills. 

**The top 5 columns of the updated dataframe are shown below**:


| gameid                | datacompleteness   |   game | side   | position   |   barons |   opp_barons |   kills |   deaths |   assists |   teamkills |   teamdeaths |   minionkills |   teamminionkills |   teamassists |   prop_kills |   prop_assists |   prop_deaths |   prop_minions |   perf_metric |
|:----------------------|:-------------------|-------:|:-------|:-----------|---------:|-------------:|--------:|---------:|----------:|------------:|-------------:|--------------:|------------------:|--------------:|-------------:|---------------:|--------------:|---------------:|--------------:|
| ESPORTSTMNT01_2690210 | complete           |      1 | Blue   | top        |        0 |            0 |       2 |        3 |         2 |           9 |           19 |           220 |              1360 |            38 |    0.222222  |      0.0526316 |      0.157895 |       0.161765 |       2.76525 |
| ESPORTSTMNT01_2690210 | complete           |      1 | Blue   | bot        |        0 |            0 |       2 |        4 |         2 |           9 |           19 |           208 |              1360 |            38 |    0.222222  |      0.0526316 |      0.210526 |       0.152941 |       2.03203 |
| ESPORTSTMNT01_2690210 | complete           |      1 | Red    | top        |        0 |            0 |       1 |        1 |        12 |          19 |            9 |           221 |              1584 |           124 |    0.0526316 |      0.0967742 |      0.111111 |       0.13952  |       2.60033 |
| ESPORTSTMNT01_2690210 | complete           |      1 | Red    | bot        |        0 |            0 |       8 |        2 |        10 |          19 |            9 |           299 |              1584 |           124 |    0.421053  |      0.0806452 |      0.222222 |       0.188763 |       3.10707 |
| ESPORTSTMNT01_2690219 | complete           |      1 | Blue   | top        |        0 |            0 |       0 |        5 |         2 |           3 |           16 |           241 |              1988 |            14 |    0         |      0.142857  |      0.3125   |       0.121227 |       0.84507 |

### Using a KS-Test

To conduct the test, we are we are picking a Kolmogorov-Smirinov (KS) test using the means of `perf_metric` for each position. We noticed that they were quantitative and had similar means, but their kernel density estimate plots had different shapes, as shown below.

<iframe src="assets/kstest.html" width=800 height=600 frameBorder=0></iframe>

### Running a Permutation Test

We first caluclated our KS test statistic using the `ks_2samp` function from the `scipy.stats` package. We ran the `ks_2samp` function on the `perf_metric` column of our dataframe. 

After running a permutation test of 500 repetitions on the shuffling the values in the `perf_metric` column, we recieved an array of KS statistics. After going ahead and plotting the distribution of the statistics against our original KS test statistic.

The resulting distribution is shown here: 

<iframe src="assets/empdist.html" width=800 height=600 frameBorder=0></iframe>

### Conclusion

Because our p-value was below the significane level, that means that our test was statistically significant and we **reject the null hypothesis**. 

This means that there is likely a significant difference in the performance and contribution to the success of the team between players that play top lane and players that play bottom lane. 

The choice of test to use the KS-Test makes sense because of the plots outputted by the kernel density function graphs being different shapes for both groups, but also the fact that our hypotheses were testing for a difference betweent the two groups, not whether one group "carried" more than the other group. Also, our result is in line with the univariate and bivariate plots we created earlier, and our aggregate which showed some difference in the performance and contribution of both players that play top lane and players that play bot lane.












