# Football_Analytic_Project
SQL analysis of football transfer dataset (successful transfers)

Dataset link: https://www.kaggle.com/datasets/davidcariboo/player-scores/data

## Main question:
Do football clubs overpay for transfer and what drives it? (For last 5 years)

## Data
- Table transfers - info about transfers
- Table appearences - info about goals, assists, red/yellow cards, minutes played for every player
- Table players - info about all players (date of birth, age, position etc)
- Table clubs - info about clubs (average age, foreign numbers)

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

### Player effeciency ratio
Effeciency of each player includes sum of goals and assists divided on total minutes played for last 3 years before date of transfer. To make the analyze project easier later, the new table 'player_performance' was cerated
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
### Average market_value of players
Split all players in 4 category:
- 1 - low perfomance players
- 2 - below average perfomance players
- 3 - above average perfomance players
- 4 - top perfomance players
```
WITH quartile_table AS (
	SELECT *,
    NTILE(4) OVER (ORDER BY effeciency) AS quartile
FROM player_performance
WHERE minutes >= 450
)
SELECT
quartile,
ROUND((AVG(market_value_in_eur)),2) AS avg_quartile
FROM quartile_table
GROUP BY quartile
```
The following screenshot shows average market price of each group (grouped by perfomance ratio) on the moment of transfer date

<img width="375" height="176" alt="image" src="https://github.com/user-attachments/assets/cc264bd0-3b97-469d-9305-348772e04267" />

### Overpay for transfers
For every transfer calculate difference and percentage between expected_valued and transfer fee (*continuing of previous SQL query*)

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
