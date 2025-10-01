# Database Schema
Following is an initial draft of the database schema. It is subject to change as per the AI Agent's requirements or data availability limitations.

## 1. Players Table
| Column Name                         | Type       | Description                                                    |
| ----------------------------------- | ---------- | -------------------------------------------------------------- |
| `player_id`                         | INT        | Unique ID (from FPL API)                                       |
| `name`                              | STRING     | Player full name                                               |
| `team_id`                           | INT (FK)   | Link to Teams table                                            |
| `gw_id`                             | INT        | GW number                                                      |
| `position`                          | STRING     | Position (GK, DEF, MID, FWD)                                   |
| `price`                             | FLOAT      | Current price in £m                                            |
| `ownership`                         | FLOAT      | % ownership across managers                                    |
| `form`                              | FLOAT      | Form rating (from API)                                         |
| `name`                              | STRING     | Player full name                                               |
| `team_id`                           | INT (FK)   | Link to Teams table                                            |
| `position`                          | STRING     | Position (GK, DEF, MID, FWD)                                   |
| `price`                             | FLOAT      | Current price in £m                                            |
| `ownership`                         | FLOAT      | % ownership across managers                                    |
| `form`                              | FLOAT      | Form rating (from API)                                         |
| `bps`                               | INT        | Bonus points system value                                      |
| `bonus`                             | INT        | Bonus points scored                                            |
| `injury_status`                     | STRING     | Status (fit, injured, doubtful, suspended)                     |
| `minutes_played`                    | INT        | Minutes played                                                 |
| `goals_scored`                      | INT        | Goals scored                                                   |
| `assists`                           | INT        | Assists                                                        |
| `clean_sheets`                      | INT        | Clean sheets                                                   |
| `goals_conceded`                    | INT        | Goals conceded                                                 |
| `own_goals`                         | INT        | Own goals scored                                               |
| `penalties_saved`                   | INT        | Penalties saved                                                |
| `penalties_missed`                  | INT        | Penalties missed                                               |
| `yellow_cards`                      | INT        | Yellow cards                                                   |
| `red_cards`                         | INT        | Red cards                                                      |
| `saves`                             | INT        | Saves                                                          |
| `influence`                         | INT        | Influence                                                      |
| `creativity`                        | INT        | Creativity                                                     |
| `threat`                            | INT        | Threat                                                         |
| `ict_index`                         | INT        | ICT Index                                                      |
| `clearances_blocks_interceptions`   | INT        | Clearances, blocks and interceptions                           |
| `recoveries`                        | INT        | Recoveries                                                     |
| `tackles`                           | INT        | Tackles                                                        |
| `defensive_contribution`            | INT        | Defensive Contribution                                         |
| `expected_goals`                    | INT        | Expected Goals                                                 |
| `expected_assists`                  | INT        | Expected Assists                                               |
| `expected_goal_involvements`        | INT        | Expected Goal Involvements                                     |
| `expected_goals_conceded`           | INT        | Expected Goals Conceded                                        |
| `points_history`                    | JSON/ARRAY | List of past GW points                                         |
| `expected_minutes`                  | FLOAT      | Proxy → rolling avg minutes / later replaced with FFS lineup % |

## 2. Teams Table
| Column Name     | Type   | Description                    |
| --------------- | ------ | ------------------------------ |
| `team_id`       | INT    | Unique team ID                 |
| `team_name`     | STRING | Team full name                 |
| `strength_home` | INT    | Relative team strength at home |
| `strength_away` | INT    | Relative team strength away    |
| `fixture_ids`   | ARRAY  | Links to fixtures              |

## 3. Fixtures Table
| Column Name       | Type     | Description                     |
| ----------------- | -------- | ------------------------------- |
| `fixture_id`      | INT      | Unique fixture ID               |
| `gw_id`           | INT      | GW number                       |
| `home_team_id`    | INT (FK) | Home team                       |
| `away_team_id`    | INT (FK) | Away team                       |
| `kickoff_time`    | DATETIME | Scheduled start time            |
| `difficulty_home` | INT      | Fixture difficulty (API rating) |
| `difficulty_away` | INT      | Fixture difficulty (API rating) |

## 4. Gameweeks Table
| Column Name     | Type       | Description                                     |
| --------------- | ---------- | ----------------------------------------------- |
| `gw_id`         | INT        | GW number                                       |
| `deadline_time` | DATETIME   | Official GW deadline                            |
| `chip_usage`    | JSON/ARRAY | Aggregate chip usage (optional, future feature) |


## 5. Lineup Likelihood Table
| Column Name        | Type     | Description                                                                       |
| ------------------ | -------- | --------------------------------------------------------------------------------- |
| `player_id`        | INT (FK) | Player ID                                                                         |
| `gw_id`            | INT (FK) | Gameweek                                                                          |
| `expected_minutes` | FLOAT    | Predicted minutes (proxy: rolling avg + injury flag; later → FFS API probability) |

## 6. Storage Plan
* Raw Data
  * Keep JSON dumps in ```/data/raw/``` (for reproducibility + debugging)
  * Format: JSON (direct API dumps)
  * ⚠️ Ignored in Git (```.gitignore```)
  * This ensures we always have a frozen copy of original API responses.
* Database Engine
  * Start with SQLite (lightweight, file-based, no setup cost)
* Processed Data
  * All cleaned + transformed data written into SQL tables (```players```, ```teams```, ```fixtures```, ```gameweeks```, ```lineup_likelihood```).
  * Easy to run SQL queries

## 7. Data Ingestion Workflow
1. Fetch Phase
    * Call FPL API → save raw JSON
2. Transform Phase
    * Parse raw → structured tables (Players, Fixtures, Teams, etc.)
3. Enrichment Phase
    * Compute ```expected_minutes``` proxy
    * Add lineup likelihood flags
4. Store Phase
    * Save as Parquet for experiments
    * (Optional) Write to SQLite

## 8. Testing and Validation
* Ensure all ```player_id``` map to valid ```team_id```.
* Fixtures align correctly with GW deadlines.
* ```expected_minutes``` values: ```0 ≤ x ≤ 90```.
* No nulls in key fields (```player_id```, ```gw_id```, ```fixture_id```).