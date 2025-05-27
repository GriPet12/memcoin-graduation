# memcoin-graduation

This model aims to predict the probability that a newly launched memecoin on the Pump Fun platform will "graduate"—defined as reaching a liquidity threshold of at least 85 SOL—based solely on data available within the first 100 blocks after token minting.

### Objective

The task is a binary classification problem evaluated using log loss. The target variable is has_graduated, indicating whether the token meets the graduation criterion.

### Data Sources

- Train/Test metadata (`train.csv`, `test_unlabeled.csv`)
- Early transaction logs (`chunk*.csv`, first 50 blocks used)
- On-chain token attributes (`token_info_onchain_divers.csv)`)
- Additional token metadata from Dune (`dune_token_info.csv`)

### Preprocessing & Feature Engineering

- Transactions were filtered to include only those within 50 blocks of the mint slot (`slot_min + 50`).
- Aggregated transactional features were extracted:
  - Total number of transactions per token
  - Number of unique signing wallets
  - Quote/base coin amounts (sum and mean)
  - Buy and sell counts
  - Buy-sell ratio

Token-level features such as `decimals` and `bundle_size` were merged in.
After preprocessing, 12 engineered features were selected for modeling. Missing values were filled with 0, and feature names were sanitized for consistency.

### Model Architecture

- The model uses LightGBM, a gradient boosting framework optimized for performance on large-scale datasets:
- Stratified 3-fold cross-validation ensures balance of the rare positive class across splits.
- Hyperparameters:
  - `n_estimators=500`
  - `learning_rate=0.05`
  - `objective='binary'`
- Early stopping was applied with patience of 50 rounds.

### Results

- Overall Out-of-Fold Log Loss: `0.0594`
- Each fold consistently achieved low log loss, indicating stable performance.
- Predictions for the test set are averaged across all folds.

## [memcoin-server-repository](https://github.com/GriPet12/memcoin_server)

The Memcoin prediction model is deployed and accessible via the following endpoint:
[`https://memcoin-server.onrender.com/predict`](https://memcoin-server.onrender.com/predict)

### How to Use

Send a `POST` request to the `/predict` endpoint with a JSON payload containing the following fields:

| Feature                  | Description                                                   |
| ------------------------ | ------------------------------------------------------------- |
| `tx_idx_count`           | Total number of transactions in the token's history           |
| `signing_wallet_nunique` | Number of unique wallets that signed transactions             |
| `quote_coin_amount_sum`  | Total traded amount in quote currency (e.g., USDT)            |
| `quote_coin_amount_mean` | Average traded amount per transaction in quote currency       |
| `base_coin_amount_sum`   | Total traded amount in base currency (e.g., token itself)     |
| `base_coin_amount_mean`  | Average traded amount per transaction in base currency        |
| `buy_count`              | Number of buy transactions                                    |
| `sell_count`             | Number of sell transactions                                   |
| `buy_sell_ratio`         | Ratio between buy and sell transactions                       |
| `decimals`               | Number of decimal places the token uses                       |
| `bundle_size`            | Number of transactions bundled together in a typical sequence |


### Response

he API responds with a JSON object containing the predicted probability that the token has the characteristic `has_graduated = 1`. For example:
```
{
  "probability": 0.0005
}
```
This value represents the model's confidence that the token will reach a "graduated" status based on its transaction behavior and structure.
