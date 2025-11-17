# Database Schema
Following is an initial draft of the database schema. It is subject to change as per the AI Agent's requirements or data availability limitations.

## 1. Players Table
| Column Name                         | Type       | Description                                                    |
| ----------------------------------- | ---------- | -------------------------------------------------------------- |
| `player_id`                         | INT (PK)   | Unique ID (from FPL API)                                       |
| `name`                              | STRING     | Player full name                                               |
| `team_id`                           | INT (FK)   | Link to Teams table                                            |
| `position`                          | STRING     | Position (GK, DEF, MID, FWD)                                   |

## 2. Player Gameweek Stats Table
| Column Name                         | Type       | Description                                                    |
| ----------------------------------- | ---------- | -------------------------------------------------------------- |
| `player_id`                         | INT        | Unique ID (from FPL API) - composite primary key i/ii          |
| `gw_id`                             | INT        | GW number - composite primary key ii/ii                        |
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
| `expected_goals`                    | FLOAT      | Expected Goals                                                 |
| `expected_assists`                  | FLOAT      | Expected Assists                                               |
| `expected_goal_involvements`        | FLOAT      | Expected Goal Involvements                                     |
| `expected_goals_conceded`           | FLOAT      | Expected Goals Conceded                                        |
| `expected_minutes`                  | FLOAT      | Proxy → rolling avg minutes / later replaced with FFS lineup % |
| `expected_points`                   | FLOAT      | Expected points                                                |

## 3. Teams Table
| Column Name     | Type   | Description                    |
| --------------- | ------ | ------------------------------ |
| `team_id`       | INT    | Unique team ID                 |
| `team_name`     | STRING | Team full name                 |
| `strength_home` | INT    | Relative team strength at home |
| `strength_away` | INT    | Relative team strength away    |

## 4. Fixtures Table
| Column Name       | Type     | Description                     |
| ----------------- | -------- | ------------------------------- |
| `fixture_id`      | INT      | Unique fixture ID               |
| `gw_id`           | INT      | GW number                       |
| `home_team_id`    | INT (FK) | Home team                       |
| `away_team_id`    | INT (FK) | Away team                       |
| `kickoff_time`    | DATETIME | Scheduled start time            |
| `difficulty_home` | INT      | Fixture difficulty (API rating) |
| `difficulty_away` | INT      | Fixture difficulty (API rating) |

## 5. Gameweeks Table
| Column Name     | Type       | Description                                     |
| --------------- | ---------- | ----------------------------------------------- |
| `gw_id`         | INT        | GW number                                       |
| `deadline_time` | DATETIME   | Official GW deadline                            |

## 6. Agent Chip Usage Table
| Column Name      | Type       | Description                                                                           |
| ---------------- | ---------- | ------------------------------------------------------------------------------------- |
| `simulation_id`  | INT        | Unique ID for each MCTS rollout                                                       |
| `gw_id`          | INT        | GW number where the chip was used                                                     |
| `chip_name`      | STRING     | One of: `wildcard`, `free_hit`, `bench_boost`, `triple_captain`, `assistant_manager`  |
### PRIMARY KEY: 
(`simulation_id`, `chip_name`)

## 7. Storage Plan
* Raw Data
  * Keep JSON dumps in ```/data/raw/``` (for reproducibility + debugging)
  * Format: JSON (direct API dumps)
  * ⚠️ Ignored in Git (```.gitignore```)
  * This ensures we always have a frozen copy of original API responses.
* Database Engine
  * We start with SQLite (lightweight, file-based, no setup cost)
* Processed Data
  * All cleaned + transformed data written into SQL tables (```players```, ```player_gameweek_stats```, ```teams```, ```fixtures```, ```gameweeks```, ```lineup_likelihood```).
  * Easy to run SQL queries

## 8. Data Ingestion Workflow
1. Fetch Phase
    * Call FPL API → save raw JSON
2. Transform Phase
    * Parse raw → structured tables (Players, Fixtures, Teams, etc.)
3. Enrichment Phase
    * Compute ```expected_minutes``` proxy
    * Add lineup likelihood flags
4. Store Phase
    * Save as Parquet for experiments
    * Write to SQLite

## 9. Testing and Validation
* Ensure all ```player_id``` map to valid ```team_id```.
* Fixtures align correctly with GW deadlines.
* ```expected_minutes``` values: ```0 ≤ x ≤ 90```.
* No nulls in key fields (```player_id```, ```gw_id```, ```fixture_id```).