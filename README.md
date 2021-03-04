# predicting-next-year-fantasy-points-per-game
Uses stats from 2019-2020 season in an attempt to predict player fantasy points per game averages for the 2020-2021 season using a multiple regression model.

Steps
1. Open the “WR_2019_Stats” and “WR_2020_Stats” files, which are sourced from FFToday.com. Copy both files into one workbook. Then note that player listings for data from FFToday.com follow the form of “#. FirstName LastName” (for example, “1. Davante Adams”). In both files, insert a column at the very left of the spreadsheet. Title this column “Player” and use Excel’s pattern-recognition capabilities to isolate player names such that no number or dot precedes them. If your first two player listings are “1. Davante Adams” and “2. Tyreek Hill” type “Davante Adams” in the cell immediately to the left of “1. Davante Adams” and “Tyreek Hill” in the cell immediately to the left of “2. Tyreek Hill”. Excel will recognize the pattern after several entries and show a prediction for what it thinks you want for all subsequent cells in the column. When this happens, hit control + shift + enter and let it populate the subsequent cells in that column.
1. For both the “WR_2019_Stats” and “WR_2020_Stats” spreadsheets, insert two columns that are the leftmost columns in the respective spreadsheets that do not yet contain data. Title these columns “FPTS” and “FPTS/G”, respective (stands for Fantasy Points and Fantasy Points Per Game, respectively). Calculate FPTS using the following formula: 0.1 points per yard (of any kind) and 6 points per touchdown (of any kind). This likely equates to the following formula: =G2*0.1+H2*6+J2*0.1+K2*6. Calculate FPTS/G by taking the value in FPTS and dividing it by the value of the “Games” column. This likely equates to the following formula: =L2/D2.
1. Open the “WR_2020_NextGenStats” file. Copy this spreadsheet into the same centralized workbook that you put both “WR_2020_Stats” and “WR_2019_Stats” into in the first step. In the first column in the WR_2020_NextGenStats spreadsheet where there is no data (likely to be Column P), type the following formula (this is effectively a less rigid alternative to VLOOKUP that does not require your data to be sorted in a specific way): =INDEX(WR_2019_Stats!$A$1:$M$151, MATCH($A3, WR_2019_Stats!$A:$A, 0), **4**). Note the presence of the dollar signs to anchor certain cells. Copy this formula into the next nine columns to the right of P, incrementing the bolded number (initially 4) each time you move one column to the right. For example, if you started in column P and then copied this formula into the next nine subsequent columns to the right, your formula in Column Y should be =INDEX(WR_2019_Stats!$A$1:$M$151, MATCH(A3, WR_2020_Stats!$A:$A, 0), **13**). Label these columns by copying the labels from cells D1 to L1 in WR_2019_Stats. This aggregates all the 2019 statistics we have for a given player in a single table in our current spreadsheet, which will be very useful when we want to select fields for a regression.
1. Do the same thing you did in the previous step, just this time for the WR_2020_Stats data. This step is necessary because it is how we will connect players and their 2019 season statistics that will be the independent variables in our regression to 2020 performancein terms of fantasy points per game that is the dependent variable in our regression. In the leftmost column not yet containing data (likely Column Z), paste the following formula for all rows with data: =INDEX(WR_2020_Stats!$A$1:$M$151, MATCH(A3, WR_2020_Stats!$A:$A, 0), 13).
1.Throughout the last two steps, you will likely notice a fair amount of “#N/A” appearances in the workbook. This is not as bad as it seems and simply requires a tiny bit of manual data cleaning.
  - Some names that use less-common characters or have multiple ways they could be listed are listed in different ways depending on the website where the data originated (i.e. nextgenstats.nfl.com vs. FFToday.com). Manually adjust names with special features that Excel would otherwise be unable to match. For example, the Next Gen Stats dataset spells Seattle wide receiver D.K. Metcalf’s name as “DK Metcalf” whereas FFToday datasets spell his name as “D.K. Metcalf” instead. Do a quick manual check to update these names to be the same in both places.
  - Other players (e.g. players who retired after the 2019-2020 season, rookies who debuted in the 2020-2021 season, or players with other atypical circumstances or injuries) do not have data for both the 2019-2020 and 2020-2021 seasons. Using the filter feature in Excel (located under “Sort and Filter” in the “Home” tab), delete them from the current view of the data set by filtering on the 2019 and 2020 “FPTS/G” columns and omitting any blank values or values equal to #N/A (we are solely interested in players with data for both seasons in this project).
1.Now we have a really convenient setup where our “WR_2019_NextGenStats” spreadsheet has been conditioned and expanded to have players in rows and every 2019 statistic we have for these players as columns, along with their 2020 fantasy point per game production as a column. Make a copy of this data and paste it in a worksheet titled “Regression Results.” In “Regression Results,” delete all columns besides Player, SEP, TAY%, YDS, TD, Games, and FPTS/G (make sure the FPTS/G column you keep is the one that corresponds to the 2020 season).
1. Create two new columns based on the existing data for each player. Create YDS/G, which is the result of the value in the YDS column for a given player divided by the value in the Games column for that player. Create TD/G, which is the result of the value in the TD column for a given player divided by the value in the Games column for that player. 
1. Pick the fields you want to use in your regression. Feel free to experiment and try this many times with combinations of fields that interest you. To reproduce the results in the repository, use the columns titled SEP, TAY%, YDS/G, and TD/G. Place these columns adjacent to each other, and hide all other columns in the spreadsheet by highlighting them, right-clicking, and selecting “hide.”
1. Using the Data Analysis ToolPak, go to the “Data” tab, select “Data Analysis” at the top right, and select the “Regression” option. Check the box that reads “labels” for good measure. Check “Residuals” to get a table of the residuals (how actual values differ from those predicted by the model). For the “input Y range,” select the cells in the FPTS/G column. For the “input X range,” select the cells in the SEP, TAY%, YDS/G, and TD/G columns. Hit “ok.”
1. Title the new worksheet that is created “Regression Results” and explore your results, taking care to note key metrics like the R-squared coefficient.
