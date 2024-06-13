# Introduction

This project conducts an in-depth analysis on the 2022 public match data for League of Legends which can be found [here](https://oracleselixir.com/tools/downloads). The dataset contains 148992 observations; for the sake of this project we will only use the aggregated team stats and we will be focusing on the following columns

- `result`: The outcome of the game. (`int`)
- `league`: The league the match was played in. (`str`)
- `side`: The side that the team played on. (`str`)
- `teamkills`: The total number of kills from the team. (`int`)
- `teamdeaths`: The total number of deaths from the team. (`int`)
- `doublekills`: The total number of double kills earned from the team. (`int`)
- `triplekills`: The total number of triple kills earned from the team. (`int`)
- `quadrakills`: The total number of quadra kills earned from the team. (`int`)
- `pentakills`: The total number of penta kills earned from the team. (`int`)
- `dragons`: The total number of dragons captured by the team. (`int`)
- `heralds`: The total number of heralds captured by the team. (`int`)
- `barons`: The total number of barons captured by the team. (`int`)
- `goldat15`/`opp_goldat15`: The total gold earned by the team/opponent at the 15 minute mark. (`int`)
- `xpat15`/`opp_xpat15`: The total xp earned by the team/opponent at the 15 minute mark. (`int`)
- `csat15`/`opp_csat15`: The total creep score earned by the team/opponent at the 15 minute mark. (`int`)

In this project, we aim to answer the question: **Does `side` have a significant impact on the outcome of a game?**

We will explore the importance these factors have on the outcome of a game with an emphasis on `'side'`. Additionally, we will explore the differences between matches in different leagues such as the LPL and LCK by comparing metrics that evaluate the action of a game. 

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning Steps

1. Filter the rows corresponding to team stats (`'position'` == 'team') because we are focusing on aggregated team data.
2. Filter the rows corresponding to matches played in regional leagues ('LCK, 'LPL', ...) because we will be comparing stats between different leagues.
3. Removed columns relating to individual players (`'player name'`, `'damage share'`, `'kills'`, ...) that aren't related to the team.
4. Add columns corresponding to differences between team and opposing team (`'killsdif'`, `'towersdif`', ...) to use as features when evaluating the state of a game.
5. Change appropriate columns to `bool` type (`'result'`, `'firstblood`', `'firstdragon`', `'firsttower`', ...).
6. Replace `'side'` with `'red_side'` column (`True` if played on Red side and `False` if played on Blue).
7. Insert `'multikills'` column (sum of `'doublekills'`, `'triplekills'`, `'quadrakills'`, and `'pentakills'`) to use as a feature when evaluating the state of a game.
8. Insert `'objectives'` column (sum of `'dragons'`, `'heralds'`, `'barons'`) to use as a feature when evaluating the state of a game.
9. Combined `'pick1'`, `'pick2`', ..., `'pick5'` into a single column, `'picks'`, corresponding to a list of the champions picked for less cluttered columns.
10. Combined `'ban1'`, `'ban2`', ..., `'ban5'` into a single column, `'bans'`, corresponding to a list of the champions banned for less cluttered columns.
11. Removed irrelevant columns (`'url'`, `'year'`, `'split'`, `'void_grubs'`, ...) to reduce cluttered columns.
12. Filled na values of `'dragons (type unknown)'` with 0.

The head of the final dataframe is shown below

| datacompleteness   | league   | teamname           | teamid                                  |   gamelength | result   |   assists |   teamkills |   teamdeaths |   doublekills |   triplekills |   quadrakills |   pentakills | firstblood   |    kpm |   ckpm | firstdragon   |   dragons |   opp_dragons |   elementaldrakes |   opp_elementaldrakes |   infernals |   mountains |   clouds |   oceans |   chemtechs |   hextechs |   dragons (type unknown) |   elders |   opp_elders | firstherald   |   heralds |   opp_heralds | firstbaron   |   barons |   opp_barons | firsttower   |   towers |   opp_towers | firstmidtower   | firsttothreetowers   |   turretplates |   opp_turretplates |   inhibitors |   opp_inhibitors |   damagetochampions |     dpm |   damagetakenperminute |   damagemitigatedperminute |   wardsplaced |    wpm |   wardskilled |   wcpm |   controlwardsbought |   visionscore |   vspm |   totalgold |   earnedgold |   earned gpm |   goldspent |        gspd |   gpr |   total cs |   minionkills |   monsterkills |   monsterkillsownjungle |   monsterkillsenemyjungle |   cspm |   goldat10 |   xpat10 |   csat10 |   opp_goldat10 |   opp_xpat10 |   opp_csat10 |   golddiffat10 |   xpdiffat10 |   csdiffat10 |   killsat10 |   assistsat10 |   deathsat10 |   opp_killsat10 |   opp_assistsat10 |   opp_deathsat10 |   goldat15 |   xpat15 |   csat15 |   opp_goldat15 |   opp_xpat15 |   opp_csat15 |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 |   assistsat15 |   deathsat15 |   opp_killsat15 |   opp_assistsat15 |   opp_deathsat15 |   killsdif |   eldersdif |   heraldsdif |   baronsdif |   towersdif |   inhibsdif | red_side   |   multikills |   objectives captured | picks                                                      | bans                                                         |
|:-------------------|:---------|:-------------------|:----------------------------------------|-------------:|:---------|----------:|------------:|-------------:|--------------:|--------------:|--------------:|-------------:|:-------------|-------:|-------:|:--------------|----------:|--------------:|------------------:|----------------------:|------------:|------------:|---------:|---------:|------------:|-----------:|-------------------------:|---------:|-------------:|:--------------|----------:|--------------:|:-------------|---------:|-------------:|:-------------|---------:|-------------:|:----------------|:---------------------|---------------:|-------------------:|-------------:|-----------------:|--------------------:|--------:|-----------------------:|---------------------------:|--------------:|-------:|--------------:|-------:|---------------------:|--------------:|-------:|------------:|-------------:|-------------:|------------:|------------:|------:|-----------:|--------------:|---------------:|------------------------:|--------------------------:|-------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|------------:|-------------:|------------:|------------:|------------:|:-----------|-------------:|----------------------:|:-----------------------------------------------------------|:-------------------------------------------------------------|
| partial            | LPL      | Oh My God          | oe:team:f4c4528c6981e104a11ea7548630c23 |         1365 | True     |        35 |          13 |            6 |           nan |           nan |           nan |          nan | False        | 0.5714 | 0.8352 | False         |         2 |             1 |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                        2 |      nan |          nan | False         |       nan |           nan | False        |        1 |            0 | False        |        8 |            3 | False           | False                |            nan |                nan |            1 |                0 |               40086 | 1762.02 |                2263.25 |                        nan |            79 | 3.4725 |            33 | 1.4505 |                   32 |           162 | 7.1209 |       45468 |        30167 |      1326.02 |       36908 | -0.00586225 |   nan |        nan |           nan |            172 |                      98 |                        18 |    nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |          7 |         nan |          nan |           1 |           5 |           1 | False      |            0 |                     3 | ['Jinx', 'Jarvan IV', 'Nautilus', 'Syndra', 'Gwen']        | ['Renekton', 'Lee Sin', 'Caitlyn', 'Jayce', 'Camille']       |
| partial            | LPL      | ThunderTalk Gaming | oe:team:df80f468a3f9a722df056fe9104f052 |         1365 | False    |        11 |           6 |           13 |           nan |           nan |           nan |          nan | True         | 0.2637 | 0.8352 | False         |         1 |             2 |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                        1 |      nan |          nan | False         |       nan |           nan | False        |        0 |            1 | False        |        3 |            8 | False           | False                |            nan |                nan |            0 |                1 |               30417 | 1337.01 |                2541.89 |                        nan |            64 | 2.8132 |            34 | 1.4945 |                   26 |           155 | 6.8132 |       38538 |        23237 |      1021.41 |       37125 |  0.00586225 |   nan |        nan |           nan |            116 |                      94 |                         2 |    nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |         -7 |         nan |          nan |          -1 |          -5 |          -1 | True       |            0 |                     1 | ['Xin Zhao', 'Thresh', 'Aphelios', 'Vex', 'Jax']           | ['Samira', 'Diana', 'Akali', 'LeBlanc', 'Rumble']            |
| partial            | LPL      | Oh My God          | oe:team:f4c4528c6981e104a11ea7548630c23 |         1444 | True     |        40 |          22 |            9 |           nan |           nan |           nan |          nan | True         | 0.9141 | 1.2465 | False         |         2 |             1 |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                        2 |      nan |          nan | False         |       nan |           nan | False        |        1 |            0 | False        |        9 |            2 | False           | False                |            nan |                nan |            1 |                0 |               59746 | 2482.52 |                3026.05 |                        nan |            67 | 2.7839 |            32 | 1.3296 |                   38 |           180 | 7.4792 |       54283 |        38176 |      1586.26 |       50858 |  0.298771   |   nan |        nan |           nan |            178 |                      88 |                        41 |    nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |         13 |         nan |          nan |           1 |           7 |           1 | False      |            0 |                     3 | ['Jinx', 'Xin Zhao', 'Rakan', 'Rumble', 'Corki']           | ['Renekton', 'Caitlyn', 'Thresh', 'Jayce', 'Camille']        |
| partial            | LPL      | ThunderTalk Gaming | oe:team:df80f468a3f9a722df056fe9104f052 |         1444 | False    |        16 |           8 |           22 |           nan |           nan |           nan |          nan | False        | 0.3324 | 1.2465 | False         |         1 |             2 |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                        1 |      nan |          nan | False         |       nan |           nan | False        |        0 |            1 | False        |        2 |            9 | False           | False                |            nan |                nan |            0 |                1 |               35129 | 1459.65 |                3107.16 |                        nan |            82 | 3.4072 |            21 | 0.8726 |                   29 |           159 | 6.6066 |       41155 |        25048 |      1040.78 |       37638 | -0.298771   |   nan |        nan |           nan |            115 |                      88 |                         0 |    nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        -14 |         nan |          nan |          -1 |          -7 |          -1 | True       |            0 |                     1 | ['Lee Sin', 'Leona', 'Ziggs', 'Gangplank', 'Twisted Fate'] | ['Samira', 'Diana', 'Jarvan IV', 'LeBlanc', 'Akali']         |
| partial            | LPL      | FunPlus Phoenix    | oe:team:33d17f3717f58e12a3da80b377221fb |         1893 | True     |        25 |          12 |            8 |           nan |           nan |           nan |          nan | False        | 0.3803 | 0.6339 | False         |         4 |             1 |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                        4 |      nan |          nan | False         |       nan |           nan | False        |        2 |            0 | False        |       10 |            3 | False           | False                |            nan |                nan |            3 |                0 |               54264 | 1719.94 |                2528.49 |                        nan |           100 | 3.1696 |            59 | 1.87   |                   41 |           274 | 8.6846 |       64011 |        43324 |      1373.19 |       58619 |  0.149322   |   nan |        nan |           nan |            264 |                     162 |                        34 |    nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |          4 |         nan |          nan |           2 |           7 |           3 | False      |            0 |                     6 | ['Jinx', 'Viego', 'Thresh', 'Corki', 'Graves']             | ['LeBlanc', 'Twisted Fate', 'Aphelios', 'Nautilus', 'Leona'] |

## Univariate Analysis

This figure plots the distribution of kpm across all games played in regional leagues. We can observe that the the distribution is roughly skewed right with a mean of roughly 0.37.
<iframe
  src="assets/kpm_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure plots the distribution of multikills across all games played in regional leauges. We can observe that the number of multikills seen in a game trends downward with a mean of roughly 0.5.
<iframe
  src="assets/multikills_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure plots the distributions of the various kinds of multikills across all games played in regional leagues. We can observe that it is indeed less common to see larger multikills, and that there were never more than one pentakill in the games observed in 2022 but 408 games included pentakills.
<iframe
  src="assets/all_multikills_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Bivariate Analysis

This figure depicts the distributions of the `'kpm'` of different leagues. We can see that the VCS league seems to have the highest median `'kpm'`whereas the lowest appears to be the LCK.
<iframe
  src="assets/kpm_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure depicts the distributions of the `'multikills'` of different leagues. We can see that the CBLOL and VCS leagues have the highest median observed `'multikills'` of 2 whereas the rest of the leagues of a median observed `'multikills'` of 1.

- Note: All LPL games were missing multikill data, which is why its box plot shows 0.

<iframe
  src="assets/multikills_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure depicts the distributions of the `'objectives'` captured by different leagues. We can observe that the distributions are very similar across all leagues, but the LCK and the PCS have the highest maxes of 10 while the LEC has the lowest max of 6.
<iframe
  src="assets/objectives_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Interesting Aggregates

We can explore the pick rates of each champion after aggregating by `'league'` to observe differences in meta across the various leagues. The table below tabulates the pick rates of each champion across different leagues. We can see that Nautilus was the most picked champion in the CBLOL, LLA, PCS, and VCS with pickrates between 3.9% to 5%, however, his pickrates in the LCK, LCS, and LEC were only between 2.1% to 2.86%. 

| picks        |     CBLOL |       LCK |       LCS |       LEC |       LLA |       PCS |       VCS |
|:-------------|----------:|----------:|----------:|----------:|----------:|----------:|----------:|
| Aatrox       | 0.534979  | 0.406852  | 1.57895   | 0.895522  | 0.695187  | 0.811808  | 0.792079  |
| Ahri         | 3.45679   | 2.91221   | 2.46711   | 2.33831   | 1.92513   | 3.02583   | 3.26733   |
| Akali        | 0.576132  | 1.47752   | 0.855263  | 1.19403   | 0.909091  | 0.664207  | 1.68317   |
| Akshan       | 0.0411523 | 0.107066  | 0.131579  | 0.248756  | 0         | 0         | 0         |
| Alistar      | 0.946502  | 0.471092  | 0.855263  | 0.39801   | 1.28342   | 1.07011   | 1.05611   |
| Amumu        | 0.576132  | 0.578158  | 0.361842  | 0.199005  | 0.427807  | 0.405904  | 0.561056  |
| Anivia       | 0         | 0         | 0.0657895 | 0         | 0         | 0         | 0         |
| Annie        | 0.0411523 | 0         | 0         | 0         | 0         | 0         | 0         |
| Aphelios     | 4.11523   | 4.00428   | 2.73026   | 2.53731   | 4.17112   | 4.46494   | 2.93729   |
| Ashe         | 0.288066  | 0.321199  | 0.328947  | 0.149254  | 0.160428  | 0.369004  | 0.330033  |
| Azir         | 1.19342   | 2.22698   | 2.23684   | 1.29353   | 2.56684   | 1.21771   | 1.22112   |
| Bard         | 0         | 0.0428266 | 0.296053  | 0.248756  | 0.160428  | 0.184502  | 0.0330033 |
| Bel'Veth     | 0         | 0.0856531 | 0         | 0.0995025 | 0.0534759 | 0.0369004 | 0         |
| Blitzcrank   | 0.0823045 | 0.0856531 | 0         | 0.0497512 | 0         | 0.0738007 | 0.0660066 |
| Braum        | 0.90535   | 0.449679  | 0.625     | 0.746269  | 1.17647   | 0.95941   | 0.891089  |
| Caitlyn      | 0.576132  | 0.620985  | 0.559211  | 0.597015  | 0.320856  | 0.147601  | 0.49505   |
| Camille      | 0.864198  | 0.728051  | 0.789474  | 0.597015  | 0.641711  | 1.14391   | 1.65017   |
| Cassiopeia   | 0.123457  | 0.0214133 | 0.0657895 | 0.0995025 | 0         | 0         | 0.0330033 |
| Corki        | 0.823045  | 1.41328   | 1.80921   | 1.39303   | 1.12299   | 0.664207  | 0.792079  |
| Darius       | 0.0411523 | 0         | 0         | 0.149254  | 0         | 0.147601  | 0         |
| Diana        | 0.288066  | 0.385439  | 0.361842  | 0.646766  | 0.374332  | 0.516605  | 0.561056  |
| Dr. Mundo    | 0         | 0         | 0.0328947 | 0         | 0         | 0         | 0         |
| Draven       | 0.493827  | 0.364026  | 0.328947  | 0.79602   | 0.855615  | 0.221402  | 0.363036  |
| Ekko         | 0         | 0         | 0         | 0         | 0         | 0.0369004 | 0         |
| Elise        | 0         | 0         | 0         | 0         | 0         | 0.0369004 | 0         |
| Ezreal       | 1.44033   | 1.30621   | 1.15132   | 1.09453   | 1.22995   | 0.627306  | 0.29703   |
| Fiora        | 0.37037   | 0.278373  | 0.592105  | 0.0995025 | 0.26738   | 0.442804  | 0.429043  |
| Galio        | 0.493827  | 0.792291  | 0.197368  | 0.248756  | 0.26738   | 1.03321   | 1.28713   |
| Gangplank    | 0.740741  | 1.26338   | 1.05263   | 1.44279   | 0.962567  | 1.21771   | 0.660066  |
| Gnar         | 2.92181   | 3.27623   | 2.23684   | 2.53731   | 3.74332   | 3.09963   | 2.83828   |
| Gragas       | 0.781893  | 1.86296   | 1.11842   | 0.79602   | 0.855615  | 1.69742   | 1.05611   |
| Graves       | 1.56379   | 1.24197   | 1.57895   | 1.09453   | 1.01604   | 1.58672   | 1.58416   |
| Gwen         | 2.34568   | 3.08351   | 2.03947   | 2.78607   | 2.29947   | 2.54613   | 3.06931   |
| Hecarim      | 0.823045  | 0.877944  | 0.921053  | 0.597015  | 1.44385   | 1.25461   | 1.15512   |
| Heimerdinger | 0         | 0.0214133 | 0         | 0         | 0         | 0.0369004 | 0         |
| Irelia       | 0.164609  | 0.171306  | 0.263158  | 0.348259  | 0.213904  | 0.258303  | 0.660066  |
| Ivern        | 0         | 0         | 0.0328947 | 0         | 0         | 0.0369004 | 0         |
| Janna        | 0.0411523 | 0.0214133 | 0.131579  | 0.149254  | 0.0534759 | 0         | 0         |
| Jarvan IV    | 1.35802   | 0.963597  | 1.77632   | 2.18905   | 1.22995   | 0.553506  | 0.660066  |
| Jax          | 0.781893  | 0.256959  | 0.328947  | 0.199005  | 0.160428  | 0.885609  | 0.924092  |
| Jayce        | 1.27572   | 1.26338   | 0.592105  | 1.14428   | 1.44385   | 0.885609  | 1.32013   |
| Jhin         | 0.740741  | 1.11349   | 0.690789  | 1.19403   | 0.802139  | 0.442804  | 0.693069  |
| Jinx         | 3.82716   | 3.27623   | 3.78289   | 2.98507   | 2.83422   | 3.80074   | 3.9604    |
| Kai'Sa       | 0.493827  | 0.663812  | 0.526316  | 0.199005  | 1.12299   | 0.442804  | 0.924092  |
| Kalista      | 1.23457   | 1.17773   | 0.592105  | 0.79602   | 1.49733   | 1.32841   | 1.05611   |
| Karma        | 0.740741  | 1.32762   | 0.986842  | 0.447761  | 1.01604   | 0.295203  | 0.49505   |
| Karthus      | 0.0411523 | 0.0214133 | 0.0986842 | 0.0995025 | 0         | 0         | 0.0990099 |
| Kassadin     | 0.0823045 | 0.0428266 | 0         | 0.0497512 | 0         | 0         | 0.0330033 |
| Kayle        | 0.452675  | 0.12848   | 0.131579  | 0.248756  | 0.427807  | 0.221402  | 0.19802   |
| Kayn         | 0         | 0         | 0.0328947 | 0         | 0         | 0         | 0         |
| Kennen       | 0.452675  | 0.235546  | 0.460526  | 0.597015  | 0.26738   | 0.405904  | 0.462046  |
| Kha'Zix      | 0.0411523 | 0         | 0.0328947 | 0         | 0         | 0.0369004 | 0.132013  |
| Kindred      | 0.0411523 | 0.0428266 | 0.0328947 | 0         | 0.106952  | 0.0738007 | 0.0330033 |
| Kled         | 0         | 0         | 0.0328947 | 0.0995025 | 0         | 0.0369004 | 0         |
| Kog'Maw      | 0.123457  | 0.107066  | 0.0986842 | 0.149254  | 0         | 0         | 0.0990099 |
| LeBlanc      | 0.699588  | 1.49893   | 1.25      | 1.34328   | 1.60428   | 0.95941   | 0.759076  |
| Lee Sin      | 3.16872   | 3.68308   | 2.13816   | 1.9403    | 2.45989   | 2.95203   | 3.59736   |
| Leona        | 3.16872   | 2.31263   | 2.13816   | 1.79104   | 1.65775   | 2.87823   | 1.55116   |
| Lillia       | 0         | 0.0428266 | 0.0657895 | 0.0995025 | 0.26738   | 0         | 0.19802   |
| Lissandra    | 1.02881   | 0.792291  | 0.493421  | 0.746269  | 1.17647   | 0.553506  | 0.528053  |
| Lucian       | 0.617284  | 0.899358  | 1.11842   | 1.69154   | 0.962567  | 0.774908  | 1.12211   |
| Lulu         | 0.946502  | 1.07066   | 2.17105   | 1.49254   | 1.01604   | 0.701107  | 0.792079  |
| Lux          | 0.329218  | 0.364026  | 0.427632  | 0.248756  | 0.106952  | 0         | 0.0990099 |
| Malphite     | 0.123457  | 0.0856531 | 0.230263  | 0.149254  | 0.26738   | 0.332103  | 0.264026  |
| Malzahar     | 0         | 0.0214133 | 0.0328947 | 0.0497512 | 0         | 0         | 0.0660066 |
| Maokai       | 0.0411523 | 0         | 0         | 0         | 0         | 0         | 0         |
| Miss Fortune | 0.164609  | 0.0856531 | 0.394737  | 0.0995025 | 0.106952  | 0.110701  | 0.165017  |
| Mordekaiser  | 0.123457  | 0.0642398 | 0.0986842 | 0         | 0.160428  | 0.332103  | 0.0990099 |
| Morgana      | 0.0411523 | 0.12848   | 0         | 0.0995025 | 0         | 0         | 0.264026  |
| Nami         | 0.37037   | 0.792291  | 0.855263  | 1.59204   | 0.802139  | 0.701107  | 0.759076  |
| Nasus        | 0         | 0         | 0         | 0         | 0         | 0.0369004 | 0         |
| Nautilus     | 4.15638   | 2.86938   | 2.86184   | 2.1393    | 3.90374   | 5.05535   | 4.35644   |
| Neeko        | 0         | 0         | 0         | 0         | 0         | 0.0369004 | 0         |
| Nidalee      | 0.0411523 | 0.171306  | 0.0657895 | 0.0497512 | 0         | 0.0738007 | 0.0990099 |
| Nilah        | 0.123457  | 0.149893  | 0.0657895 | 0.199005  | 0.0534759 | 0         | 0.0330033 |
| Nocturne     | 0.288066  | 0.256959  | 0.460526  | 0.149254  | 0.160428  | 0.332103  | 1.12211   |
| Olaf         | 0.699588  | 0.149893  | 0.427632  | 0.199005  | 0.26738   | 0.0738007 | 0.0330033 |
| Orianna      | 0.987654  | 0.513919  | 1.25      | 1.34328   | 1.06952   | 1.40221   | 0.49505   |
| Ornn         | 0.90535   | 1.30621   | 1.84211   | 2.38806   | 2.45989   | 0.516605  | 0.858086  |
| Pantheon     | 0.0823045 | 0.0428266 | 0.0328947 | 0.149254  | 0.106952  | 0.0738007 | 0.363036  |
| Poppy        | 0.864198  | 1.79872   | 1.64474   | 1.69154   | 0.748663  | 1.03321   | 0.924092  |
| Pyke         | 0.164609  | 0.107066  | 0.0657895 | 0.348259  | 0.320856  | 0         | 0.264026  |
| Qiyana       | 0         | 0.0214133 | 0.0657895 | 0.0995025 | 0         | 0.0738007 | 0.0660066 |
| Quinn        | 0.0411523 | 0         | 0         | 0         | 0         | 0         | 0         |
| Rakan        | 1.52263   | 1.64882   | 1.31579   | 2.28856   | 1.01604   | 1.58672   | 1.45215   |
| Rek'Sai      | 0         | 0.171306  | 0.0986842 | 0         | 0.160428  | 0.110701  | 0.231023  |
| Rell         | 0.37037   | 0.214133  | 0.0986842 | 0.149254  | 0.160428  | 0.147601  | 0.29703   |
| Renata Glasc | 1.56379   | 1.56317   | 1.18421   | 1.59204   | 2.19251   | 1.66052   | 2.07921   |
| Renekton     | 0.699588  | 1.02784   | 1.15132   | 1.49254   | 1.01604   | 1.36531   | 1.35314   |
| Rengar       | 0         | 0         | 0.0328947 | 0         | 0         | 0         | 0         |
| Riven        | 0         | 0         | 0.0328947 | 0         | 0         | 0         | 0.132013  |
| Rumble       | 0.699588  | 0.0214133 | 0.131579  | 0.0497512 | 0.213904  | 0.110701  | 0.0990099 |
| Ryze         | 1.35802   | 1.86296   | 1.11842   | 0.945274  | 1.5508    | 1.77122   | 1.88119   |
| Samira       | 0.0411523 | 0.214133  | 0         | 0.149254  | 0         | 0.184502  | 0.165017  |
| Sejuani      | 1.06996   | 1.37045   | 1.15132   | 1.34328   | 1.44385   | 1.77122   | 0.561056  |
| Senna        | 0.617284  | 0.877944  | 0.953947  | 1.04478   | 0.748663  | 0.590406  | 0.825083  |
| Seraphine    | 0.740741  | 0.449679  | 0.756579  | 0.298507  | 0.588235  | 0.369004  | 0.561056  |
| Sett         | 0.329218  | 0.278373  | 0.296053  | 0.0995025 | 0.320856  | 0.295203  | 0.39604   |
| Shen         | 0         | 0.0214133 | 0.0986842 | 0         | 0.0534759 | 0.0369004 | 0.0990099 |
| Shyvana      | 0.164609  | 0.0642398 | 0.0986842 | 0.298507  | 0.106952  | 0.0738007 | 0.165017  |
| Singed       | 0         | 0.107066  | 0         | 0         | 0         | 0.0369004 | 0.0330033 |
| Sion         | 0.37037   | 0.278373  | 0.493421  | 0.497512  | 0.802139  | 0.295203  | 0.165017  |
| Sivir        | 0.864198  | 0.770878  | 1.90789   | 1.89055   | 1.17647   | 1.62362   | 1.84818   |
| Skarner      | 0         | 0.299786  | 0.0657895 | 0.149254  | 0         | 0.0369004 | 0         |
| Sona         | 0         | 0.0428266 | 0.164474  | 0.0497512 | 0         | 0         | 0         |
| Soraka       | 0.0411523 | 0.0214133 | 0.164474  | 0.348259  | 0.0534759 | 0         | 0         |
| Swain        | 0.576132  | 0.385439  | 0.460526  | 0.447761  | 0.320856  | 0.479705  | 0.363036  |
| Sylas        | 1.39918   | 1.34904   | 1.21711   | 2.18905   | 1.65775   | 1.91882   | 1.55116   |
| Syndra       | 0.658436  | 0.492505  | 0.625     | 0.248756  | 0.320856  | 0.811808  | 0.693069  |
| Tahm Kench   | 1.27572   | 2.24839   | 1.84211   | 1.69154   | 2.45989   | 2.02952   | 2.21122   |
| Taliyah      | 1.35802   | 0.792291  | 0.822368  | 1.39303   | 1.44385   | 1.25461   | 1.32013   |
| Talon        | 0.0411523 | 0.0428266 | 0         | 0         | 0         | 0.0369004 | 0.0990099 |
| Taric        | 0.0411523 | 0.0214133 | 0.0657895 | 0.199005  | 0.106952  | 0         | 0         |
| Thresh       | 0.740741  | 0.770878  | 1.34868   | 1.04478   | 0.962567  | 0.258303  | 0.594059  |
| Tristana     | 0.37037   | 0.0856531 | 0.0328947 | 0.0497512 | 0.213904  | 0.295203  | 0.330033  |
| Trundle      | 1.68724   | 1.39186   | 2.00658   | 2.78607   | 1.97861   | 1.32841   | 0.561056  |
| Tryndamere   | 0.781893  | 0.920771  | 0.822368  | 0.199005  | 0.374332  | 0.922509  | 0.660066  |
| Twisted Fate | 0.823045  | 0.685225  | 0.723684  | 0.945274  | 0.534759  | 0.258303  | 0.462046  |
| Twitch       | 0.37037   | 0.214133  | 0.296053  | 0.646766  | 0.0534759 | 0.0738007 | 0.231023  |
| Udyr         | 0.0411523 | 0.0214133 | 0.361842  | 0         | 0.160428  | 0         | 0.0990099 |
| Urgot        | 0         | 0.0214133 | 0.0657895 | 0         | 0.0534759 | 0.0369004 | 0         |
| Varus        | 0.205761  | 0.256959  | 0.230263  | 0.348259  | 0.534759  | 0.110701  | 0.165017  |
| Vayne        | 0         | 0.0642398 | 0.0986842 | 0.0995025 | 0.160428  | 0.0369004 | 0.165017  |
| Veigar       | 0.164609  | 0.235546  | 0.197368  | 0.149254  | 0.26738   | 0.479705  | 0.429043  |
| Vel'Koz      | 0.0411523 | 0         | 0         | 0         | 0.0534759 | 0         | 0         |
| Vex          | 1.893     | 0.706638  | 0.460526  | 0.79602   | 0.588235  | 0.664207  | 0.957096  |
| Vi           | 0.411523  | 0.835118  | 0.559211  | 0.696517  | 1.39037   | 0.885609  | 0.660066  |
| Viego        | 3.7037    | 2.71949   | 2.96053   | 2.18905   | 2.6738    | 3.83764   | 3.16832   |
| Viktor       | 1.68724   | 1.606     | 2.30263   | 1.64179   | 1.65775   | 1.18081   | 1.55116   |
| Vladimir     | 0.0411523 | 0.0214133 | 0.0986842 | 0         | 0.0534759 | 0.0369004 | 0.0330033 |
| Volibear     | 1.27572   | 0.685225  | 1.15132   | 1.19403   | 1.49733   | 1.10701   | 1.22112   |
| Wukong       | 2.6749    | 2.37687   | 2.03947   | 2.53731   | 2.19251   | 2.43542   | 2.77228   |
| Xayah        | 0.699588  | 1.17773   | 0.526316  | 0.646766  | 0.695187  | 1.62362   | 1.38614   |
| Xerath       | 0.0411523 | 0         | 0         | 0         | 0         | 0         | 0         |
| Xin Zhao     | 2.79835   | 2.7409    | 2.36842   | 2.48756   | 1.76471   | 1.91882   | 1.51815   |
| Yasuo        | 0.123457  | 0.171306  | 0.361842  | 0.547264  | 0.160428  | 0.0369004 | 0.330033  |
| Yone         | 0.452675  | 0.214133  | 0.493421  | 0.348259  | 0.481283  | 0.332103  | 0.264026  |
| Yorick       | 0.0411523 | 0         | 0         | 0         | 0         | 0         | 0         |
| Yuumi        | 1.39918   | 1.64882   | 1.51316   | 1.54229   | 0.695187  | 1.14391   | 0.924092  |
| Zac          | 0         | 0.0428266 | 0.0328947 | 0.0497512 | 0.0534759 | 0         | 0         |
| Zed          | 0.0411523 | 0         | 0.0657895 | 0         | 0         | 0.0738007 | 0         |
| Zeri         | 2.42798   | 2.22698   | 3.38816   | 2.63682   | 2.35294   | 2.76753   | 3.0363    |
| Ziggs        | 0         | 0.171306  | 0.0328947 | 0.0497512 | 0.106952  | 0.147601  | 0.0660066 |
| Zilean       | 0.164609  | 0.214133  | 0.559211  | 0.547264  | 0.0534759 | 0.258303  | 0.0660066 |
| Zoe          | 0.576132  | 0.385439  | 0.328947  | 0.547264  | 0.534759  | 0.701107  | 0.0330033 |
| Zyra         | 0         | 0         | 0.0328947 | 0.0497512 | 0         | 0         | 0         |

# Assessment of Missingness

Assessing each column before cleaning, we believe one column that could be NMAR is the `'url'` column. The values in the `'url'` column are all lpl.qq.com or matchhistory.na.leagueoflegends.com urls, so it could be that matches recorded on other urls were not kept in the dataset. 

## Missingness Hypothesis Test 1
To assess the missingness of `'doublekills'` on `'league'` we run the following hypothesis test:

- Null: The distribution of `'league'` when `'doublekills'` is missing is the same as when `'doublekills'` is not missing.
- Alternative: The distribution of `'league'` is different between when `'doublekills'` is missing and not missing.
- Test Statistic: TVD between the categorical distributions of `'league'`.

We reject the null hypothesis with a p-value of 0.0. Thus, we can conclude that `'doublekills'` is MAR on `'league'`. The empirical tvds from our permutation testing is depicted below.

<iframe
  src="assets/doublekills_on_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

At first this result seems very bizzare, but it can be explained by the fact that all of the observations where `'doublekills'` (among similar columns) were missing came from LPL games. 

## Missingness Hypothesis Test 2
To assess the missingness of `'doublekills'` on `'dpm'` we run the following hypothesis test:

- Null: The distribution of `'dpm'` when `'doublekills'` is missing as the same as when `'doublekills'` is not missing.
- Alternative: The distribution of '`dpm'` is different between when `'doublekills'` is missing and not missing.
- Test Statistic: Difference of means between the quantitative distributions of `'dpm'`.

We fail to reject the null hypothesis with a p-value of 0.474. Thus, we cannot conclude that `'doublekills'` is MAR on `'dpm'`. The empirical difference of means from our permutation testing is depicted below. 

<iframe
  src="assets/doublekills_on_dpm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Hypothesis Testing

## Hypothesis Test 1

### Set Up
- Null: The mean kpm (kills per minute) between the LPL league and LCK league is the same.
- Alternative: The mean kpm (kills per minute) of the LPL league is higher than the LCK league.
- Test Statistic: Difference of Means.
- Significant Level: 5%.

### Results
- P-value: 0.0

<iframe
  src="assets/kpm_lck_lpl.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Conclusions

With a p-value of 0, we have evidence to reject the null hypothesis. The mean `'kpm'` of the LPL is likely to be higher than the mean `'kpm'` of the LCK. We can use this to imply that LPL games have more action than LCK games because there are more kills per minute. 

## Hypothesis Test 2

### Set Up
- Null: The mean result (winrate) between Red side and Blue side is the same.
- Alternative: The mean result (winrate) between Red side and Blue side is not the same.
- Test Statistic: Difference of Means.

### Results
- P-value: 0.0

<iframe
  src="assets/red_vs_blue.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Conclusions

With a p-value of 0, we have evidence to reject the null hypothesis. The mean winrate of Red side is likely not to be the same as the mean winrate of Blue side. This indicates that `'side'` has a significant impact on the outcome of a game.

# Framing a Prediction Problem

## Question

Can we predict if a team will win/lose the game?

- Binary classification problem
- We want to predict the `'result'` column
- We will use accuracy as the metric for its simplicity and we are not concerned with false positive and false negative tradeoffs.

# Baseline Model

Using information available 15 minutes into the game, we utilize the following features to create a basic `DecisionTreeClassifier()`:
- `'red_side'`: nominal feature, found by encoding `'side'` to `'True'` if `'side' == 'Red'` and `'False'` if `'side' == Blue'`.
- `'golddifat15'`: quantitative feature, found by subtracting `'goldat15'` by `'opp_goldat15'`.
- `'xpdifat15'`: quantitative feature, found by subtracting `'xpat15'` by `'opp_xpat15'`.
- `'csdifat15'`: quantitative feature, found by subtracting `'csat15'` by `'opp_csat15'`.
- `'killsat15'`: quantitative feature, original column.
- `'assistsat15'`: quantitative feature, original column.
- `'deathsat15'`: quantitative feature, original column.

In total, we used 1 nominal and 6 quantitative features. This model achieved train accuracy of `0.747` and test accuracy of `0.755`. We do not believe that this model is good yet because its accuracy is fairly low and can be improved.

# Final Model

Expanding to using all of the available information throughout the game, we utilize the following features to create a `RandomForestClassifier()`:
- `'red_side'`: we chose to keep this feature because it showed to be a strong predictor in an earlier hypothesis test.
- `'killsdif'`: we chose to include this feature because having more or less kills than your opponent should be a good indicator of the result of the game. 
- `'objectives captured'`: we chose to include this feature because having more or less objectives captured should be a decent indicator of the result of a game.
- `'earned gpm'`: we chose to include this feature because a team earning more or less than average gold per minute should be a good indicator of the result of a game.

For this final model, we chose to use a `RandomForestClassifier()`, which is a model that trains an ensemble of `DecisionTreeClassifier()'s` and bases its prediction on a vote cast by each `DecisionTreeClassifier()`. `RandomForestCliassifier()`'s are more complex than `DeccisionTreeClassifier()`s, but they can be more robust to unseen data.

To fine tune the hyperparameters, we used a `GridSearchCV()` to find the combination of `max_depth`, `criterion`, and `min_samples_split` with the best validation score. We would test a range for each hyperparameter and adjust the range depending on the result until the hyperparameters converged. 

Our resulting best hyperparameters were `criterion='gini'`, `max_depth=3`, and `min_samples_split=48`.

Our final model achieved a train accuracy of `0.978` test accuracy of `0.96`. It performs considerably better than our baseline model because it has achieved much higher accuracy without overfitting to the training data. 

# Fairness Analysis

We will evaluate our final model's F1-score between two groups:
- Group 1: Red side teams
- Group 2: Blue side teams

## Setup
We will conduct the following hypothesis test:
- Null: The model is fair. Its F1-score is the same for teams playing Red side and Blue side.
- Alternative: The model is unfair. Its F1-score is higher for teams playing on Red side over Blue side.

## Results
- P-value: 0.558

<iframe
  src="assets/red_blue_f1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Conclusions
With a p-value of 0.558, we fail to reject the null hypothesis. We cannot say that our model unfairly favors predicting one side over the other.

# Overall

Throughout this project, we were able to achieve a deep understanding of the dataset and come across various findings:
- Games across different regional leagues are quite different, with some leagues demeonstrating more action in their games over others.
- Regional leagues develop their own metas with champion pick rates varying across each region
- The missingness of the data largely stems from incomplete data reported by LPL games.
- Side indeed has a significant impact on determining the outcome of the game.
- Using features based on information provided at early stages of a game can create a decent model to predict `'result'`.
- Using features based on information provided after the entire game can create a strong model to predict `'result'`.
- Despite our final model incorporating `'side'` as a feature, it does not evaluate predictions unfairly between sides.