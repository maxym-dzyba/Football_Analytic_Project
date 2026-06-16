# Football_Analytic_Project
SQL analysis of football transfer dataset (successful transfers)

Dataset link: https://www.kaggle.com/datasets/davidcariboo/player-scores/data

## Main question:
Do clubs overpay for high-performing players efficiently, or waste money on low-performing ones? (For last 5 years)

## Data
- Table transfers - info about transfers
- Table appearences - info about goals, assists, red/yellow cards, minutes played for every player

## Overpay Analysis (2021–2026)

### Overpay ratio and total overpay by country
The percentage of overpay is the main, but also showed total overpay in EUR.

    SELECT
    to_club_name,
    ROUND(SUM(transfer_fee) / SUM(market_value_in_eur), 2) AS overpay_ratio,
    SUM(transfer_fee - market_value_in_eur) AS total_overpay,
    COUNT(*) AS transfers_count
    FROM transfers
    WHERE transfer_fee > 0
    AND transfer_fee != ''
    AND market_value_in_eur != ''
    AND transfer_date >= '2021-01-01'
    GROUP BY to_club_name
    HAVING COUNT(*) > 5
    ORDER BY overpay_ratio DESC;
    
The chart shows which clubs consistently overpay relative to market value.
<img width="934" height="363" alt="image" src="https://github.com/user-attachments/assets/92e4d501-b47e-4031-bde1-674f7547cef8" />

Additional time-period analysis was performed

Insights:
- High ratio often comes from few transfers
- Big clubs overpay more in total due to volume
- Market value is used as a proxy for expected player value, which may not fully capture contextual factors such as injuries, potential, or tactical fit

### Player efficiency ratio
Efficiency of each player includes sum of goals and assists divided on total minutes played for last 3 years before date of transfer. To make the analyze project easier later, the new table 'player_performance' was cerated
```
CREATE TABLE player_performance AS
WITH players_from_2021 AS (
    SELECT *
    FROM transfers
    WHERE transfer_date >= '2021-01-01' AND transfer_date <= '2026-05-30'
      AND transfer_fee != ''
      AND transfer_fee != 0
      AND market_value_in_eur != ''
      AND market_value_in_eur != 0
),
player_performance_2021 AS (
    SELECT
    t.player_id,
    t.transfer_date,
    t.market_value_in_eur,
    t.transfer_fee,
    SUM(a.goals + a.assists) AS ga,
    SUM(a.minutes_played) AS minutes
    FROM players_from_2021 t JOIN appearances a ON t.player_id = a.player_id
    WHERE a.date < t.transfer_date
    AND a.date >= date(t.transfer_date, '-1095 days')
    GROUP BY t.player_id, t.transfer_date
)
SELECT *,
	ROUND((ga * 90.0 / minutes), 2) AS effeciency
FROM player_performance_2021
```

<img width="1339" height="689" alt="image" src="https://github.com/user-attachments/assets/de127a50-0a2d-4669-8b26-fc110169fa71" />

Insights:
- The most efficient players are underpaid
- Clubs overpay even for non-efficient players

### Average market value of players
Split all players in 4 category:
- 1 - low perfomence players
- 2 - below average perfomence players
- 3 - above average perfomence players
- 4 - top perfomence players
```
WITH quartile_table AS (
	SELECT *,
    NTILE(4) OVER (ORDER BY effeciency) AS quartile
FROM player_performance
WHERE minutes >= 450
)
avg_market_val_per_quartile AS (
SELECT
quartile,
ROUND((AVG(market_value_in_eur)),2) AS avg_quartile
FROM quartile_table
GROUP BY quartile
),
```
The following screenshot shows average market price of each group (grouped by perfomence ratio) on the moment of transfer date

<img width="375" height="176" alt="image" src="https://github.com/user-attachments/assets/cc264bd0-3b97-469d-9305-348772e04267" />

### Overpay for transfers
For every transfer calculate difference and percentage between expected value and transfer fee (*continuing of previous SQL query*)

```
players_with_avg_marker_val AS(
SELECT
q.player_id,
q.transfer_date,
q.transfer_fee,
avm.avg_quartile AS expected_value
FROM quartile_table q JOIN avg_market_val_per_quartile avm ON q.quartile = avm.quartile
)
SELECT *,
	ROUND((transfer_fee - expected_value), 0) AS overpay,
	ROUND(((transfer_fee - CAST(expected_value AS FLOAT)) / CAST(expected_value AS FLOAT)) * 100, 2) AS overpay_pct
FROM players_with_avg_marker_val
```

### Total club overpay
*continuing of previous SQL query*

```
SELECT
	ot.player_id,
	ot.transfer_date,
	t.to_club_name,
	ot.transfer_fee,
	ot.expected_value,
	ot.overpay,
	ot.overpay_pct
FROM overpay_table ot JOIN transfers t ON ot.player_id = t.player_id AND ot.transfer_date = t.transfer_date
)
SELECT
	to_club_name,
	ROUND((AVG(overpay_pct)), 2) AS club_overpay_ratio
FROM club_join_table 
GROUP BY to_club_name
ORDER BY club_overpay_ratio DESC
```

On following screenshot showed total club overpay for last 5 years, starts from 01.01.2021 includes compare of efficiency of player

<img width="540" height="525" alt="image" src="https://github.com/user-attachments/assets/e96783c4-8f00-497b-a35e-b0ca062048c9" />

<img width="949" height="665" alt="image" src="https://github.com/user-attachments/assets/ca322911-40a6-4862-b8f9-705fac1697dc" />


Insights:
- Almost all big and succsesfull clubs overpay for effective players. Less popular clubs usually buy players for market value or lower of it.

### Top most expensive overpays per coutnry

*continuing of previous SQL query*
```
SELECT *
FROM (
    SELECT
        to_club_name,
        player_id,
        transfer_date,
        transfer_fee,
        overpay,
        ROW_NUMBER() OVER(PARTITION BY to_club_name ORDER BY overpay DESC) AS rn
    FROM club_join
) t
WHERE rn <= 3
ORDER BY to_club_name, overpay DESC
)
```

Insights:
- Most clubs top 1 most overpaid transfers take the largest part of total overpay.
- There are clubs, that overpay for players which are in above average perfomence players or top perfomence players groups. This is working strategy

### Transfer Investment Efficiency Index

*continuing of previous SQL query*
```
club_join AS(
SELECT ot.*,
t.to_club_name,
CASE 
	WHEN (quartile IN(4,3)) AND overpay > 0 THEN 'good overpay'
	WHEN (quartile IN(1,2)) AND overpay > 0 THEN 'bad overpay'
	ELSE 'neutral'
END AS overpay_quality
FROM overpay_table ot JOIN transfers t ON ot.player_id = t.player_id AND ot.transfer_date = t.transfer_date
)
SELECT
    to_club_name,
    SUM(CASE WHEN overpay_quality = 'good overpay' THEN overpay ELSE 0 END) AS smart_overpay,
    SUM(CASE WHEN overpay_quality = 'bad overpay' THEN overpay ELSE 0 END) AS waste_overpay,
ROUND((SUM(CASE WHEN overpay_quality = 'good overpay' THEN overpay ELSE 0 END)
        -
SUM(CASE WHEN overpay_quality = 'bad overpay' THEN overpay ELSE 0 END)) * 1.0 / COUNT(DISTINCT player_id), 2) AS efficiency_score,
SUM(overpay) AS total_overpay
FROM club_join
GROUP BY to_club_name
ORDER BY effecience_Score  DESC
```

<img width="1169" height="527" alt="image" src="https://github.com/user-attachments/assets/73be799d-4e69-48da-b2a0-6da10bb0b7d7" />

<img width="1339" height="669" alt="image" src="https://github.com/user-attachments/assets/b01997bc-fcbe-4f6c-866b-8e866a313bb7" />


Insights:
- The biggest clubs have the biggest smart overpay, which give them quality players. But at the same time they have the biggest wasted overpay
- Transfer overpay is not the proof of inefficiency or efficiency of players
- Most clubs do not systematically overpay

## Conclusions
- Clubs do not fail because of average decisions - they fail because of a few expensive mistakes that has a big impact of total overpay of club
- Market value often doesn't show the real price for transfer
- Larger clubs show higher total overpayment mostly due to higher transfer volume and higher spending capacity, not necessarily worse decision quality
- Market value doesn't show a real price for player

## Reccomendations
- Focus on reducing downside risk in high-cost transfers, rather than avoiding low-performing player categories entirely
- Clubs should optimize scouting for value gap (performance vs price), especially in mid-tier markets
- Squad efficiency is better improved through balanced portfolio building rather than targeting only top-performing players
