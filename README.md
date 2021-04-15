# predicting-next-year-fantasy-points-per-game
Uses stats from 2019-2020 season in an attempt to predict player fantasy points per game averages for the 2020-2021 season using a multiple regression model.
## Data Sources
1. 2019 general statistics: [WR_2019_Stats](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/WR_2019_Stats.xlsx) | [FFToday Website](https://fftoday.com/stats/playerstats.php?Season=2019&GameWeek=&PosID=30&LeagueID=)
2. 2019 Next Gen Stats:  [WR_2019_NextGenStats](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/WR_2019_NextGenStats.xlsx) | [NFL NextGenStats Website](https://nextgenstats.nfl.com/stats/receiving/2019/REG/all#yards)
3. 2020 general statistics: [WR_2020_Stats](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/WR_2020_Stats.xlsx) | [FFToday Website](https://fftoday.com/stats/playerstats.php?Season=2020&GameWeek=&PosID=30&LeagueID=)
## The Linear Regression Model
Fantasy football is a popular game where participants draft a team of players as if they were the General Manager of an NFL football team and compete against their friends to see who can score more fantasy points in a given week. With more than 40 million players throughout the nation and lots of pride and money at stake, people make a living out of offering advice and attempting to project player performance in subsequent seasons as competitors are eager to give themselves any competitive advantage when constructing at team. This leads to the following business question: using solely stats from the 2019-2020 season, how accurately can I predict wide receivers’ production in the 2020-2021 season?

For the sake of this analysis, “production” will defined in terms of fantasy points per game using standard scoring settings. Fantasy points are picked because who wins and who loses a fantasy matchup is decided by how many fantasy points a team scores. Fantasy points are then weighted on a per-game basis to account for the prevalence of injuries in football under the premise that players should be judged on the proficiency of their performance in games they play rather than their ability to stay healthy over an entire season.

With the benchmark statistic identified, I then proceeded to identifying several commonly available 2019-2020 season statistics for players. However, it was important to apply two ways of thinking in determining what statistics to use in a regression. First, there is value in using one statistic to account for a given categorical area that you want to be include in the model to avoid strong correlations between model variables (a precondition for linear regression). Many statistics are related to one another, and this would show up in the form of an extremely high p-value if both were used in a regression. For example, the number of targets a wide receiver received (i.e. how many times their team tried to throw them the ball) and how many receptions the wide receiver had (i.e. how many times the wide receiver actually caught the ball) are understandably related. This meant that it only made sense to include on volume-based metric detailing the involvement of a given wide receiver in the offense. After experimenting with different fields, the Next Gen Stat “TAY%,” which stands for Total Air Yards Percentage and details the percentage of all quarterback throws weighted for how many yards they were that were intended to be caught by the given wide receiver. This stat consistently had a higher coefficient in the regression outcome while having an acceptably low p-value (0.047 in the final regression), suggesting it was an effective indicator of the opportunities that a wide receiver got to accumulate yards and score touchdowns (and thus accrue fantasy points). 

In the context of fantasy points directly depending on yards and touchdowns (0.1 * yards + 6 * touchdowns), I also rather intuitively wanted to feature average yards per game and average touchdowns per game from the 2019 season into the model. In the context of using one statistic to account for different categorical areas in the model to avoid strong correlations between model variables, I wanted to validate this approach and ensure that there were no strong correlations between TAY%, average yards per game, and average touchdowns per game. I verified that these two stats are not strongly correlated to each other (R-squared coefficient of just 0.37) and pose no issue of correlation to TAY% (R-squared coefficients of 0.53 and 0.20, respectively). Thus, my linear regression model so far is built on total air yards percentage, average yards per game, and average touchdowns per game.

Lastly, I wanted to include a statistic in the model that captured how good a wide receiver was at being open (i.e. evading defenders and leaving the quarterback lots of room to throw him or her a pass). The Next Gen Stat SEP, which is the average separation (i.e. distance) that a wide receiver has from the nearest defender when thrown the ball, was used in this regard to incorporate an underlying stat into the model that wasn’t solely based on results (like average yards per game and average touchdowns per game) or team offense strength (total air yards percentage might be lower for a wide receiver who has other great wide receivers playing alongside him or her). This statistic rounded out the model.

## Interpreting the Result

![alt text](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/Projected_vs._Actual_Fantasy_Points_Per_Game.png)
When 2019-2020 separation, total air yards percentage, average yards per game, and average touchdowns per game were regressed together to 2020-2021 average fantasy points per game, the resulting equation for predicted 2020-2021 average fantasy points per game was as follows:

Projected 2020-2021 Average Fantasy Points per Game = -4.84667 + 2.08078 * SEP + 0.11383 * TAY% + 0.04566 * YDS/G + 4.61338 * TD/G. 

These coefficients can be thought of as a weight of how important each variable is in projecting 2020-2021 average fantasy points per game, however they need to be interpreted with a grain of salt. Recall that fantasy points are calculated in accordance with the formula Fantasy Points = 0.1 * Yards + 6 * Touchdowns. Since one touchdown is by definition worth 60 times more points than one yard, seeing the coefficient of 4.61338 for TD/G in relation to the coefficient of 0.04566 would somewhat counterintuitively actually suggest that average yards per game is a more important predictor than average touchdowns per game when taken proportionally in this context. A good way to interpret the coefficients is to frame it as a one unit increase in my variable would be projected to translate to an increase in the 2020-2021 average fantasy points per game projection equal to the coefficient. For example, if I was able to on average achieve one more unit of separation from defenders over the course of the 2019-2020 season, my projected fantasy points per game for the next season would increase by 2.08078. The standard error can be thought of as the variability (similar to standard deviation) associated with the given coefficient in the same row. Higher standard errors correspond to greater uncertainty. It is worth noting that, relative to the magnitude of the coefficients, our standard errors tend to be quite large, which is less than ideal but not something that derails this model.

The F-Significant test statistic of less than 0.01 (on the order of ten to the negative 7) is a very good sign. This suggests that the probability that none of targeted air yards percentage, average yards per game, average touchdowns per game, and average separation are significant is close to zero, which is a vote of confidence for the model.
The p-values, which give a snapshot of the probability of observing this outcome by chance, were mostly acceptable although on the higher end of acceptable. They were 0.11336 for the intercept and 0.015615, 0.04679, 0.07187, and 0.04365, respectively, for each of the aforementioned variables. Most of these p-values are below the common thresholds of 0.05. This suggests that the variables in the experiment are likely not problematically correlated to one another, a necessary step for giving this model validity.

One additional step worth taking is to take a look at the residual plot. Residuals are defined as the actual value in real life minus the projected value from the model. In short, when a plot of projected values from the model and their associated residuals is produced, a random scatter of points should be observed (no clear pattern should be present). Our model passes this test as shown in the plot below.

![alt text](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/Residuals.png)

Lastly, let’s dive into our model’s R-squared coefficient. On the surface, an R-squared coefficient of 0.402 might sound rather abysmal. It suggests that only about 40% of the variation in 2020-2021 average fantasy points per game can be explained by  the 2019 stats separation, total air yards percentage, average yards per game, and average touchdowns per game. You could argue that flipping a coin would have similar predictive power. However, taken in the context of how truly random year-to-year fantasy point production is in life, the R-squared coefficient of 0.402 is nothing to sneeze at. For example, a straight-up regression of 2019-2020 average fantasy points per game to 2020-2021 average fantasy points per game only has an R-squared coefficient of 0.347, so this model increases our predictive power by just over six percentage points—a nice accomplishment suggesting that this model does indeed add some value.

While stats like the R-squared coefficient are fantastic at providing quantitative descriptions of the data, a visual representation is also useful for capturing the whole story. When the 2020-2021 fantasy points per game projections of the model are compared to players’ actual 2020-2021 season fantasy points per game for the same players, a clear positive association is observed, which is a good sign. Moreover, three likely outliers jump out from the data. Top wide receiver Michael Thomas was projected to average 12.79 fantasy points per game in the 2020-2021 season, but instead found himself playing at less than 100 percent and yielding significantly subpar results relative to this projection. On the flip side, wide receivers Davante Adams and Tyreek Hill improved significantly on solid-but-not-spectacular 2019-2020 performances to light up the league as the two top wide receivers in the game for the 2020-2021 season, averaging 17.53 and 16.13 fantasy points per game in the actual 2020-2021 season respectively. Overall, this model appears to be better than current existing statistics at projecting fantasy points one season in advance using the prior season’s data. While there is definitely room for improvement and it would neat to include additional stats like player age, team offense rank, quarterback rating, team offensive line rank, and team wins record, this model is a promising start that will help me and others to win games in upcoming fantasy seasons.

![alt text](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/Projected_vs._Actual_Fantasy_Points_Per_Game.png)

## Steps for replicating analysis:
1. Open the “WR_2019_Stats” and “WR_2020_Stats” files, which are sourced from FFToday.com. Copy both files into one workbook. Then note that player listings for data from FFToday.com follow the form of “#. FirstName LastName” (for example, “1. Davante Adams”). In both files, insert a column at the very left of the spreadsheet. Title this column “Player” and use Excel’s pattern-recognition capabilities to isolate player names such that no number or dot precedes them. If your first two player listings are “1. Davante Adams” and “2. Tyreek Hill” type “Davante Adams” in the cell immediately to the left of “1. Davante Adams” and “Tyreek Hill” in the cell immediately to the left of “2. Tyreek Hill”. Excel will recognize the pattern after several entries and show a prediction for what it thinks you want for all subsequent cells in the column. When this happens, hit control + shift + enter and let it populate the subsequent cells in that column.
1. For both the “WR_2019_Stats” and “WR_2020_Stats” spreadsheets, insert two columns that are the leftmost columns in the respective spreadsheets that do not yet contain data. Title these columns “FPTS” and “FPTS/G”, respective (stands for Fantasy Points and Fantasy Points Per Game, respectively). Calculate FPTS using the following formula: 0.1 points per yard (of any kind) and 6 points per touchdown (of any kind). This likely equates to the following formula: =G2*0.1+H2*6+J2*0.1+K2*6. Calculate FPTS/G by taking the value in FPTS and dividing it by the value of the “Games” column. This likely equates to the following formula: =L2/D2.
1. Open the “WR_2020_NextGenStats” file. Copy this spreadsheet into the same centralized workbook that you put both “WR_2020_Stats” and “WR_2019_Stats” into in the first step. In the first column in the WR_2020_NextGenStats spreadsheet where there is no data (likely to be Column P), type the following formula (this is effectively a less rigid alternative to VLOOKUP that does not require your data to be sorted in a specific way): =INDEX(WR_2019_Stats!$A$1:$M$151, MATCH($A3, WR_2019_Stats!$A:$A, 0), **4**). Note the presence of the dollar signs to anchor certain cells. Copy this formula into the next nine columns to the right of P, incrementing the bolded number (initially 4) each time you move one column to the right. For example, if you started in column P and then copied this formula into the next nine subsequent columns to the right, your formula in Column Y should be =INDEX(WR_2019_Stats!$A$1:$M$151, MATCH(A3, WR_2020_Stats!$A:$A, 0), **13**). Label these columns by copying the labels from cells D1 to L1 in WR_2019_Stats. This aggregates all the 2019 statistics we have for a given player in a single table in our current spreadsheet, which will be very useful when we want to select fields for a regression.
1. Do the same thing you did in the previous step, just this time for the WR_2020_Stats data. This step is necessary because it is how we will connect players and their 2019 season statistics that will be the independent variables in our regression to 2020 performancein terms of fantasy points per game that is the dependent variable in our regression. In the leftmost column not yet containing data (likely Column Z), paste the following formula for all rows with data: =INDEX(WR_2020_Stats!$A$1:$M$151, MATCH(A3, WR_2020_Stats!$A:$A, 0), 13).
1. Throughout the last two steps, you will likely notice a fair amount of “#N/A” appearances in the workbook. This is not as bad as it seems and simply requires a tiny bit of manual data cleaning. Some names that use less-common characters or have multiple ways they could be listed are listed in different ways depending on the website where the data originated (i.e. nextgenstats.nfl.com vs. FFToday.com). Manually adjust names with special features that Excel would otherwise be unable to match. For example, the Next Gen Stats dataset spells Seattle wide receiver D.K. Metcalf’s name as “DK Metcalf” whereas FFToday datasets spell his name as “D.K. Metcalf” instead. Do a quick manual check to update these names to be the same in both places. Other players (e.g. players who retired after the 2019-2020 season, rookies who debuted in the 2020-2021 season, or players with other atypical circumstances or injuries) do not have data for both the 2019-2020 and 2020-2021 seasons. Using the filter feature in Excel (located under “Sort and Filter” in the “Home” tab), delete them from the current view of the data set by filtering on the 2019 and 2020 “FPTS/G” columns and omitting any blank values or values equal to #N/A (we are solely interested in players with data for both seasons in this project).

1. Now we have a really convenient setup where our “WR_2019_NextGenStats” spreadsheet has been conditioned and expanded to have players in rows and every 2019 statistic we have for these players as columns, along with their 2020 fantasy point per game production as a column. Make a copy of this data and paste it in a worksheet titled “Regression Results.” In “Regression Results,” delete all columns besides Player, SEP, TAY%, YDS, TD, Games, and FPTS/G (make sure the FPTS/G column you keep is the one that corresponds to the 2020 season).
1. Create two new columns based on the existing data for each player. Create YDS/G, which is the result of the value in the YDS column for a given player divided by the value in the Games column for that player. Create TD/G, which is the result of the value in the TD column for a given player divided by the value in the Games column for that player. 
1. Pick the fields you want to use in your regression. Feel free to experiment and try this many times with combinations of fields that interest you. To reproduce the results in the repository, use the columns titled SEP, TAY%, YDS/G, and TD/G. Place these columns adjacent to each other, and hide all other columns in the spreadsheet by highlighting them, right-clicking, and selecting “hide.”
1. Using the Data Analysis ToolPak, go to the “Data” tab, select “Data Analysis” at the top right, and select the “Regression” option. Check the box that reads “labels” for good measure. Check “Residuals” to get a table of the residuals (how actual values differ from those predicted by the model). For the “input Y range,” select the cells in the FPTS/G column. For the “input X range,” select the cells in the SEP, TAY%, YDS/G, and TD/G columns. Hit “ok.”
1. Title the new worksheet that is created “Regression Results” and explore your results, taking care to note key metrics like the R-squared coefficient. If you wish to replicate the residuals plot, make a scatter plot using the "Predicted FPTS/G" and "Residuals" columns of the residuals data below the initial regression results.
1. If interested in replicating the Projected vs. Actual Fantasy Points Per Game Plot, make a scatterplot of 2019 FPTS/G and 2020 FPTS/G.

## Updated Python Re-Analysis
# Link to Python Code
[Google Colaboratory Notebook](https://colab.research.google.com/drive/1MYalMjVm9kwQC4geE5MOU37lSAk90OPV?usp=sharing)
# Updated Regression Results
![alt text](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/Updated%20Python%20Regression%20Results.PNG)
# Updated Plotly Visualizations
# Predicted 2020 Fantasy Points per Game versus Actual 2020 Fantasy Points per Game
[Interactive visualization downloadable in HTML form](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/predicted_versus_actual_fantasy_points_2020) | [Static screenshot](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/predicted_versus_actual_fantasy_points_2020_static.PNG)
![alt text](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/predicted_versus_actual_fantasy_points_2020_static.PNG)
This visual is largely the same as the initial rendition of the Predicted 2020 FPTS/G versus Actual 2020 FPTS/G graph created in Excel with the modification that this model has been updated to also include a stat called plus minus (more on this below). Relative to current models, this model has an R-squared coefficient of 0.412, suggesting that about 41.2% of the variation in Actual 2020 FPTS/G is accounted for by the model's prediction for Predicted 2020 FPTS/G. We can see the mix of players who outperform the model (Actual 2020 FPTS/G exceeds Predicted 2020 FPTS/G), who underperforms the model (vice versa), and what players especially fit the model (the model correctly projected Emmanuel Sanders' actual 2020 FPTS/G within 0.0008 fantasy points per game).

# Residuals versus Predicted 2020 Fantasy Points per Game
[Interactive visualization downloadable in HTML form](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/residuals_versus_predicted_fantasy_points_2020) | [Static screenshot](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/residuals_versus_predicted_fantasy_points_2020_static.PNG)
![alt text](https://github.com/tberkery/predicting-next-year-fantasy-points-per-game/blob/main/residuals_versus_predicted_fantasy_points_2020_static.PNG)
This residual plot is exactly what we would hope it would look like. It makes good sense that a relatively similar amount of players outperform and underperform the model. There is also no clear-cut pattern to the points in the scatterplot, suggesting that the linear model is an appropriate fit for this data.

One really neat feature of the interactive HTML versions of both visuals is that you can hover over specific data points and see the player that each data point corresponds to.

# Advantages of Python and Model Adjustments
While Excel is a convenient and powerful tool, I was blown away by the increased functionality and ease of building a model in Python. For Excel, I found myself doing significant copying and pasting of custom formulas that could not be generalized to many different columns. This also frequently entailed reinventing the wheel when cleaning my data (e.g. removing dots and hyphens from player names) and performing many repetitive tasks by hand (e.g. creating scaterplots, writing VLOOKUPS, etc.) In contrast, with pandas dataframes in Pandas dataframes in Python, I was able to use built-in functions like split, upper, replace, and merge without having to write these formulaically from scratch or writing the same formula/code many times. 

In addition, outside of simplifying repetitive tasks, I appreciated how Python makes it much easier to scale and adapt your work. With Python, I found it very easy to vary the variables in my model. Since testing a combination of input variables is as simple as adding or removing an additional column of a dataframe into the input for the SKLearn model, I was able to efficiently try additional combinations of variables in the linear regression model and see the coefficients output on the model and the impact on the R-squared. This led me to add a stat called plus-minus, which measures how many more or less yards than expected a player averages per reception, to the model, resulting in a one percentage point increase in the R-squared coefficient.

Python versus Excel is a stylistic dilemma. Python has the con of being the higher fixed cost option but the pro of having lower long-run variable cost. In contrast, Excel is the lower fixed cost option with high variable costs that do not decrease significantly in the long-run. Stylistically, I'm a big fan of the former and investing time upfront for payoff in the future and plan to use Python for future linear regression analysis.
