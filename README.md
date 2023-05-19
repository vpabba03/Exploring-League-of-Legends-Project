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



|   kills |   assists |   minionkills |   deaths |
|--------:|----------:|--------------:|---------:|
| 4.2588  |   5.37153 |       256.718 |  2.54763 |
| 4       |   5       |       256     |  2       |
| 2.79474 |   5.02498 |       232.843 |  2.95333 |
| 2       |   4       |       230     |  3       |











