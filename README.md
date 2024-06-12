# 2022 LOL Analysis

## Introduction

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

We will explore the importance these factors have on the outcome of a game with an emphasis on `side`. Additionally, we will explore the differences between matches in different leagues such as the LPL and LCK by comparing metrics that evaluate the action of a game. 

## Data Cleaning and EDA

### Data Cleaning Steps

1. Filter the rows corresponding to team stats (`'position'` == 'team') because we are focusing on aggregated team data.
2. Filter the rows corresponding to matches played in regional leagues ('LCK, 'LPL', ...) because we will be comparing stats between different leagues.
3. Removed columns relating to individual players (`'player name'`, `'damage share'`, `'kills'`, ...) that aren't related to the team.
4. Add columns corresponding to differences between team and opposing team (`'killsdif'`, `'towersdif`', ...) to use as features when evaluating the state of a game.
5. Change appropriate columns to `bool` type (`'result'`, `'firstblood`', `'firstdragon`', `'firsttower`', ...).
6. Replace `'side'` with `'red_side'` column (`True` if on played on Red side and `False` if played on Blue).
7. Insert `'multikills'` column (sum of `'doublekills'`, `'triplekills'`, `'quadrakills'`, and `'pentakills'`) to use as a feature when evaluating the state of a game.
8. Insert `'objectives'` column (sum of `'dragons'`, `'heralds'`, `'baronds'`) to use as a feature when evaluating the state of a game.
9. Combine `'pick1'`, `'pick2`', ..., `'pick5'` into a single column `'picks'` corresponding to a list of the champions picked for less cluttered columns.
10. Combine `'ban1'`, `'ban2`', ..., `'ban5'` into a single column `'bans'` corresponding to a list of the champions banned for less cluttered columns.
11. Filtered rows that had complete data (`'datacompleteness'` == 'complete') because some observations with 'partial' or 'ignore' completeness were missing key data points that we could not reasonably infer/impute.
12. Removed irrelevant columns (`'url'`, `'datacompleteness'`, `'year'`, `'split'`, `'void_grubs'`, ...) to reduce cluttered columns.
13. Filled na values of `'dragons (type unknown)'` with 0.

The head of the final dataframe is shown below

| league   | teamname     | teamid                                  |   gamelength | result   |   assists |   teamkills |   teamdeaths |   doublekills |   triplekills |   quadrakills |   pentakills | firstblood   |    kpm |   ckpm | firstdragon   |   dragons |   opp_dragons |   elementaldrakes |   opp_elementaldrakes |   infernals |   mountains |   clouds |   oceans |   chemtechs |   hextechs |   dragons (type unknown) |   elders |   opp_elders | firstherald   |   heralds |   opp_heralds | firstbaron   |   barons |   opp_barons | firsttower   |   towers |   opp_towers | firstmidtower   | firsttothreetowers   |   turretplates |   opp_turretplates |   inhibitors |   opp_inhibitors |   damagetochampions |     dpm |   damagetakenperminute |   damagemitigatedperminute |   wardsplaced |    wpm |   wardskilled |   wcpm |   controlwardsbought |   visionscore |   vspm |   totalgold |   earnedgold |   earned gpm |   goldspent |        gspd |   gpr |   total cs |   minionkills |   monsterkills |   monsterkillsownjungle |   monsterkillsenemyjungle |    cspm |   goldat10 |   xpat10 |   csat10 |   opp_goldat10 |   opp_xpat10 |   opp_csat10 |   golddiffat10 |   xpdiffat10 |   csdiffat10 |   killsat10 |   assistsat10 |   deathsat10 |   opp_killsat10 |   opp_assistsat10 |   opp_deathsat10 |   goldat15 |   xpat15 |   csat15 |   opp_goldat15 |   opp_xpat15 |   opp_csat15 |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 |   assistsat15 |   deathsat15 |   opp_killsat15 |   opp_assistsat15 |   opp_deathsat15 |   killsdif |   eldersdif |   heraldsdif |   baronsdif |   towersdif |   inhibsdif | red_side   |   multikills |   objectives captured | picks                                                  | bans                                                         |
|:---------|:-------------|:----------------------------------------|-------------:|:---------|----------:|------------:|-------------:|--------------:|--------------:|--------------:|-------------:|:-------------|-------:|-------:|:--------------|----------:|--------------:|------------------:|----------------------:|------------:|------------:|---------:|---------:|------------:|-----------:|-------------------------:|---------:|-------------:|:--------------|----------:|--------------:|:-------------|---------:|-------------:|:-------------|---------:|-------------:|:----------------|:---------------------|---------------:|-------------------:|-------------:|-----------------:|--------------------:|--------:|-----------------------:|---------------------------:|--------------:|-------:|--------------:|-------:|---------------------:|--------------:|-------:|------------:|-------------:|-------------:|------------:|------------:|------:|-----------:|--------------:|---------------:|------------------------:|--------------------------:|--------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|------------:|-------------:|------------:|------------:|------------:|:-----------|-------------:|----------------------:|:-------------------------------------------------------|:-------------------------------------------------------------|
| LCK      | DRX          | oe:team:101f8589e58c724c1dcd5a9c1555277 |         2195 | False    |         9 |           5 |           14 |             0 |             0 |             0 |            0 | True         | 0.1367 | 0.5194 | False         |         1 |             4 |                 1 |                     4 |           0 |           0 |        0 |        1 |           0 |          0 |                        0 |        0 |            0 | True          |         2 |             0 | False        |        0 |            2 | True         |        8 |            9 | True            | True                 |             10 |                  0 |            0 |                2 |               50038 | 1367.78 |                2227.52 |                    2178.86 |           127 | 3.4715 |            57 | 1.5581 |                   55 |           305 | 8.3371 |       63747 |        39978 |      1092.79 |       60525 |  0.00190185 |  2.24 |        nan |           945 |            287 |                     nan |                       nan | 33.6765 |      15121 |    18570 |      330 |          14840 |        18166 |          324 |            281 |          404 |            6 |           0 |             0 |            0 |               0 |                 0 |                0 |      27198 |    30325 |      511 |          22441 |        28785 |          510 |           4757 |         1540 |            1 |           4 |             7 |            1 |               1 |                 1 |                4 |         -9 |           0 |            2 |          -2 |          -1 |          -2 | False      |            0 |                     3 | ['Aphelios', 'Sona', 'Viego', 'Graves', 'Ryze']        | ['Diana', 'Caitlyn', 'Twisted Fate', 'LeBlanc', 'Viktor']    |
| LCK      | Liiv SANDBOX | oe:team:c75f1f337fc5867914749d438a4871d |         2195 | True     |        39 |          14 |            5 |             2 |             0 |             0 |            0 | False        | 0.3827 | 0.5194 | True          |         4 |             1 |                 4 |                     1 |           0 |           1 |        0 |        2 |           0 |          1 |                        0 |        0 |            0 | False         |         0 |             2 | True         |        2 |            0 | False        |        9 |            8 | False           | False                |              0 |                 10 |            2 |                0 |               59071 | 1614.7  |                2236.78 |                    1898.73 |           114 | 3.1162 |            64 | 1.7494 |                   43 |           292 | 7.9818 |       67669 |        43900 |      1200    |       60410 | -0.00190185 | -2.24 |        nan |           986 |            204 |                     nan |                       nan | 32.5285 |      14840 |    18166 |      324 |          15121 |        18570 |          330 |           -281 |         -404 |           -6 |           0 |             0 |            0 |               0 |                 0 |                0 |      22441 |    28785 |      510 |          27198 |        30325 |          511 |          -4757 |        -1540 |           -1 |           1 |             1 |            4 |               4 |                 7 |                1 |          9 |           0 |           -2 |           2 |           1 |           2 | True       |            2 |                     6 | ['Yuumi', 'Xin Zhao', 'Jhin', 'Syndra', 'Tryndamere']  | ['Renekton', 'Lee Sin', 'Leona', 'Jayce', 'Akali']           |
| LCK      | DRX          | oe:team:101f8589e58c724c1dcd5a9c1555277 |         2070 | False    |        21 |           7 |           15 |             0 |             0 |             0 |            0 | False        | 0.2029 | 0.6377 | False         |         0 |             4 |                 0 |                     4 |           0 |           0 |        0 |        0 |           0 |          0 |                        0 |        0 |            0 | True          |         1 |             1 | False        |        1 |            1 | True         |        3 |            9 | False           | False                |              3 |                  1 |            0 |                1 |               66774 | 1935.48 |                2318.49 |                    2678.09 |           117 | 3.3913 |            59 | 1.7101 |                   60 |           262 | 7.5942 |       60674 |        38182 |      1106.72 |       60660 |  0.00914149 | -1.31 |        nan |           994 |            186 |                     nan |                       nan | 34.2029 |      15495 |    17872 |      318 |          16695 |        19149 |          333 |          -1200 |        -1277 |          -15 |           2 |             5 |            4 |               4 |                 5 |                2 |      23612 |    29371 |      528 |          24657 |        30106 |          546 |          -1045 |         -735 |          -18 |           2 |             5 |            4 |               4 |                 5 |                2 |         -8 |           0 |            0 |           0 |          -6 |          -1 | False      |            0 |                     2 | ['Aphelios', 'Jarvan IV', 'Thresh', 'Ryze', 'Graves']  | ['Diana', 'Caitlyn', 'Yuumi', 'Samira', 'Syndra']            |
| LCK      | Liiv SANDBOX | oe:team:c75f1f337fc5867914749d438a4871d |         2070 | True     |        31 |          15 |            7 |             2 |             0 |             0 |            0 | True         | 0.4348 | 0.6377 | True          |         4 |             0 |                 4 |                     0 |           0 |           0 |        0 |        2 |           1 |          1 |                        0 |        0 |            0 | False         |         1 |             1 | True         |        1 |            1 | False        |        9 |            3 | True            | True                 |              1 |                  3 |            1 |                0 |               57616 | 1670.03 |                2938.81 |                    2685.86 |           104 | 3.0145 |            60 | 1.7391 |                   45 |           253 | 7.3333 |       67152 |        44660 |      1294.49 |       60108 | -0.00914149 |  1.31 |        nan |          1051 |            227 |                     nan |                       nan | 37.0435 |      16695 |    19149 |      333 |          15495 |        17872 |          318 |           1200 |         1277 |           15 |           4 |             5 |            2 |               2 |                 5 |                4 |      24657 |    30106 |      546 |          23612 |        29371 |          528 |           1045 |          735 |           18 |           4 |             5 |            2 |               2 |                 5 |                4 |          8 |           0 |            0 |           0 |           6 |           1 | True       |            2 |                     6 | ['Leona', 'Xin Zhao', 'Akali', 'Ezreal', 'Tryndamere'] | ['Renekton', 'Lee Sin', 'Twisted Fate', 'Viktor', 'LeBlanc'] |
| LCK      | T1           | oe:team:ce499dea30cfce118f4fe85da0227e8 |         2233 | True     |        26 |          12 |            7 |             1 |             1 |             0 |            0 | True         | 0.3224 | 0.5105 | True          |         5 |             0 |                 4 |                     0 |           1 |           0 |        0 |        2 |           0 |          1 |                        0 |        1 |            0 | True          |         2 |             0 | True         |        1 |            0 | False        |        7 |            2 | True            | True                 |              2 |                  1 |            1 |                0 |               74327 | 1997.14 |                2789.72 |                    2453.12 |           122 | 3.2781 |            83 | 2.2302 |                   50 |           332 | 8.9207 |       66455 |        42304 |      1136.69 |       59950 |  0.0386823  |  1    |        nan |           970 |            236 |                     nan |                       nan | 32.4048 |      15662 |    18130 |      324 |          15510 |        19078 |          337 |            152 |         -948 |          -13 |           1 |             1 |            1 |               1 |                 1 |                1 |      24357 |    29835 |      518 |          23048 |        30005 |          533 |           1309 |         -170 |          -15 |           3 |             2 |            1 |               1 |                 1 |                3 |          5 |           1 |            2 |           1 |           5 |           1 | False      |            2 |                     8 | ['Karma', 'Ezreal', 'Jarvan IV', 'Gragas', 'Zoe']      | ['Lee Sin', 'Ryze', 'Viktor', 'LeBlanc', 'Graves']           |

### Univariate Analysis

This figure plots the distribution of kpm across all games played in regional leagues. We can observe that the the distribution is roughly skewed right with a mean of about 0.37.
<iframe
  src="assets/kpm_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure plots the distribution of multikills across all games played in regional leauges. We can observe that the number of multikills seen in a game trends downward with a mean around 2.
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

### Bivariate Analysis

This figure depicts the distributions of the `'kpm'` of different leagues. We can see that the VCS league seems to have the highest median `'kpm'`whereas the lowest appears to be the LCK.
<iframe
  src="assets/kpm_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure depicts the distributions of the `'multikills'` of different leagues. We can see that the CBLOL and VCS leagues have the highest median observed `'multikills'` of 2 whereas the rest of the leagues of a median observed `'multikills'` of 1.
<iframe
  src="assets/multikills_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure depicts the distributions of the `'objectives'` captured by different leagues. We can observe that the distributions are very similar across all leagues, but the LCK and the PCS have the highest maxes of 10 while the LEC has the lowest max of 6.
<iframe
  src="assets/multikills_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

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

### Assessment of Missingness

Assessing each column before cleaning, we believe one column that could be NMAR is the `'url'` column. The values in the `'url'` column are all lpl.qq.com or matchhistory.na.leagueoflegends.com urls, so it could be that matches recorded on other urls were not kept in the dataset. 

#### Hypothesis Test 1
To assess the missingness of `'doublekills'` on `'league'` we run the following hypothesis test:

- H0: The distribution of `'league'` when `'doublekills'` is missing is the same as when `'doublekills'` is not missing.
- H1: The distribution of `'league'` is different between when `'doublekills'` is missing and not missing.
- Test Statistic: TVD between the categorical distributions of `'league'`.

We reject the null hypothesis with a p-value of 0.0. Thus, we can conclude that `'doublekills'` is MAR on `'league'`. The empirical tvds from our permutation testing is depicted below.

<iframe
  src="assets/doublekills_on_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

At first this result seems very bizzare, but it can be explained by the fact that all of the observations where `'doublekills'` (among similar columns) were missing came from LPL games. 

#### Hypothesis Test 2
To assess the missingness of `'doublekills'` on `'dpm'` we run the following hypothesis test:

- H0: The distribution of `'dpm'` when `'doublekills'` is missing as the same as when `'doublekills'` is not missing.
- H1: The distribution of '`dpm'` is different between when `'doublekills'` is missing and not missing.
- Test Statistic: Difference of means between the quantitative distributions of `'dpm'`.

We fail to reject the null hypothesis with a p-value of 0.474. Thus, we cannot conclude that `'doublekills'` is MAR on `'dpm'`. The empirical difference of means from our permutation testing is depicted below. 

<iframe
  src="assets/doublekills_on_dpm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

