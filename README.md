# Wallet-Risk-Scoring-From-Scratch

## 📝 Brief Explanation

# 📦 1. Data Collection Method
- We collected wallet-level data from the Compound V2 subgraph hosted by The Graph. This was done using GraphQL queries to retrieve detailed information for each wallet address, including:

- Wallet’s supply and borrow balances across tokens

- Number of times the wallet has been liquidated

- Number of times the wallet acted as a liquidator

- GraphQL Endpoint Used: https://api.thegraph.com/subgraphs/name/graphprotocol/compound-v2

# 🎯 2. Feature Selection Rationale
The selected features are directly aligned with wallet risk in the context of lending/borrowing on Compound:


| Feature            | Description                                                                 |
| ------------------ | --------------------------------------------------------------------------- |
| `total_supplied`   | Sum of assets the wallet has supplied                                       |
| `total_borrowed`   | Sum of borrowed assets                                                      |
| `net_position`     | `total_supplied - total_borrowed` — a proxy for solvency                    |
| `utilization_rate` | Ratio of `borrowed / supplied` — high utilization suggests high leverage    |
| `countLiquidated`  | Number of times the wallet was liquidated — strong indicator of poor risk   |
| `countLiquidator`  | Number of times the wallet *performed* a liquidation — proxy for savvy user |


# 📊 3. Scoring Method
Each wallet is given a score between 0 and 1000, with higher values indicating lower risk.

for i, row in df.iterrows():
    total_supplied = row["total_supplied"]
    total_borrowed = row["total_borrowed"]
    net_position = row["net_position"]
    num_liquidations = row["num_liquidations"]
    num_times_liquidator = row["num_times_liquidator"]
    
    # Calculate utilization rate
    utilization_rate = total_borrowed / total_supplied if total_supplied > 0 else 0

    # Scoring
    score = 1000

    if utilization_rate > 0.8:
        score -= 300

    score -= 200 * num_liquidations

    if net_position < 0:
        score -= 100

    score += 100 * num_times_liquidator

    # Clamp score between 0 and 1000
    score = max(0, min(1000, score))

    # Assign to DataFrame
    df.loc[i, "score"] = score




# ✅ 4. Justification of Risk Indicators


| Risk Indicator         | Justification                                                                |
| ---------------------- | ---------------------------------------------------------------------------- |
| `utilization_rate`     | High leverage (borrowed/supplied) indicates fragility to price volatility    |
| `num_liquidations`     | Shows a history of default or over-leveraging                                |
| `net_position`         | A negative balance suggests more debt than assets — a key risk signal        |
| `num_times_liquidator` | Acting as a liquidator shows engagement with the protocol and risk knowledge |

