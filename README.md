# Football_Analytic_Project
SQL analysis of football transfer dataset (successful transfers)

Dataset link: https://www.kaggle.com/datasets/davidcariboo/player-scores/data

## Main question:
Do football clubs overpay for transfer and what drives it?

## Data
- Table transfers - info about transfers
- Table appearences - info about goals, assists, red/yellow cards, minutes played for every player
- Table players - info about all players (date of birth, age, position etc)
- Table clubs - info about clubs (average age, foreign numbers)

## Analytic process

### Overpay ratio and total overpay by country
The analysis is split in 3 different periods to consider the market value. Transfers before 2009 not included due a small amount of transfers. Transfers in the future included. The percentage of overpay is the main, but also showed total overpay in EUR.

Top 10 total overpay ratio:
   
    SELECT
    to_club_name,
    ROUND(SUM(transfer_fee) / SUM(market_value_in_eur), 2) AS overpay_ratio,
    SUM(transfer_fee - market_value_in_eur) AS total_overpay,
    COUNT(*) AS transfers_count
    FROM transfers
    WHERE transfer_fee > 0
    AND transfer_fee != ''
    AND market_value_in_eur != ''
    GROUP BY to_club_name
    HAVING COUNT(*) > 5
    ORDER BY overpay_ratio DESC;
<img width="877" height="361" alt="image" src="https://github.com/user-attachments/assets/8c8c2cd8-6533-4646-ae4a-c36c3703f0bd" />
Top 10 overpay ration in period 2021 - 2030 (transfers in the future included)
<img width="934" height="363" alt="image" src="https://github.com/user-attachments/assets/92e4d501-b47e-4031-bde1-674f7547cef8" />
Top 10 overpay ration in period 2015 - 2021
<img width="935" height="362" alt="image" src="https://github.com/user-attachments/assets/9cdab77f-b348-4e2b-a90f-79958ea4acab" />
Top 10 overpay ration in period 2009 - 2015
<img width="934" height="365" alt="image" src="https://github.com/user-attachments/assets/dbff64b3-8e7e-4463-98f1-b5daac8d72cd" />
