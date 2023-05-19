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

Generally, it is a hot topic among League of Legends players on which lane is the most important, so our tests will be to determine which lane does the most for their team in the overall match. We based the performance metric off the popular Kills-Deaths-Assists (KDA) ratio, but instead include minions kills in the calculation because players that kill more minions tend to also improve the success of their team and perform better.

## Data Cleaning and Exploratory Data Analysis

### Importing Relevant Packages
- **pandas** : pack












