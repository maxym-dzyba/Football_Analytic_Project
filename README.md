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

## Overpay Analysis (2021–2030)

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
