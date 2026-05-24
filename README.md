# Football_Analytic_Project
SQL analysis of football transfer dataset (successful transfers)

Dataset link: https://www.kaggle.com/datasets/davidcariboo/player-scores/data

## Main question:
Do football clubs overpay for transfer and what drives it?

## Analytic process

### Overpay ratio and total overpay by country
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

