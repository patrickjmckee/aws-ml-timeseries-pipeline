// /data/datasetSpec.md
// Document condition and designed modifications for raw data and schema for all data


# Required Modifications for 1RegimeStable
### A. Sensor Dropout
#### Purpose:
Simulate telemetry loss, wiring faults, comms interruptions.
#### Specification
* Random blackout windows:
* duration: configurable (e.g., 5–50 cycles)
* probability per unit
* Permanent sensor failure:
* sensor flatlines or becomes NaN after a given cycle
* Missingness is not silently imputed
#### Design Requirement
Missingness becomes a feature, not just a preprocessing problem.
### B. Distribution Drift
#### Purpose:
Simulate wear-rate changes, operating environment shifts, maintenance effects.
#### Specification
* Regime-specific mean shifts applied to subsets of sensors
* Variable degradation slope per unit
* Step changes (simulated maintenance or operating point change)
#### Design Requirement
Drift is detectable but ambiguous (not trivially labeled)
### C. Label Handling (RUL)
#### Purpose:
Prevent leakage and unrealistic clairvoyance.
#### Specification
* RUL removed from inference-time inputs
* RUL retained only for:
* offline evaluation
* visualization
* audit comparison
### D. Time Semantics
#### Purpose:
Support latency, ordering, and stale inference handling.
#### Specification
* Explicit cycle index retained
* Optional wall-clock timestamp added (synthetic)
### E. Dataset Identity
#### Purpose:
* Create project neutral, but more descriptive dataset and file names
* Neutralize “NASA C-MAPSS” conspicuousness without deception.
#### Specification
* Dataset name abstracted (1RegimeStable)
* Operating assumptions framed as “rotating industrial asset”
* Documentation explicitly states benchmark origin + modifications
## Raw Data Schema
### Design Scope 
Dataset 1RegimeStable_raw, pre-modification, except for timestamps and IDs.
### Design Intent
* Immutable
* Lossless
* Traceable to original benchmark
### Design Schema
|  Field Name  |  Type  |  Description  |
|  ----------  |  -----------  |  ----------------  |
|  asset_id  |  int  |  Unique unit identifier  |
|  cycle  |  int  |  Monotonic operating cycle  |
|  op_setting_1  |  float  |  Operating setting  |
|  op_setting_2  |  float  |  Operating setting  |
|  op_setting_3  |  float  |  Operating setting  |
|  sensor_01 … sensor_21  |  float  |  Raw sensor measurements  |
|  source_dataset  |  string  |  "1RegimeStable"  |
|  ingestion_timestamp  |  timestamp  |  When ingested into system  |
### Design Notes
* No relabeling of failures
* No synthetic fault injection except sensor loss
* No drift injection
* No derived features
* No control loop simulation
* No RUL leakage
* No optimization against RUL leaderboard metrics
## Processed Data Schema
### Design Scope 
* 1RegimeStable_processed
* This is post-rename, pre-modification, except for timestamps and IDs.
### Design Intent
* Safe for ML consumption
* Explicitly encodes uncertainty
* Supports validation, monitoring, and gating

### Keys
| Field           | Type      |
| --------------- | --------- |
| asset_id        | int       |
| cycle           | int       |
| event_timestamp | timestamp |

### Sensor Signals (post-modification)
| Field                 | Type    | Notes                     |
| --------------------- | ------- | ------------------------- |
| sensor_01 … sensor_21 | float   | May contain NaN           |
| sensor_missing_count  | int     | Number of missing sensors |
| sensor_missing_ratio  | float   | Missing / total           |
| sensor_blackout_flag  | boolean | Active blackout window    |

### Drift & Regime Indicators
| Field                 | Type    | Notes                      |
| --------------------- | ------- | -------------------------- |
| drift_score           | float   | Statistical drift metric   |
| drift_flag            | boolean | Thresholded                |
| degradation_slope_est | float   | Estimated wear rate        |
| regime_id             | string  | Stable / shifted (derived) |

### Model Interaction Fields
| Field                 | Type    | Notes                                |
| --------------------- | ------- | ------------------------------------ |
| health_index          | float   | Normalized health estimate           |
| failure_risk_score    | float   | Probability-like                     |
| prediction_confidence | float   | Used for gating                      |
| inference_valid       | boolean | Passed confidence + freshness checks |
| inference_latency_ms  | int     | Used for stale detection             |

### Control Safety Fields
| Field              | Type    | Notes       |
| ------------------ | ------- | ----------- |
| tuning_allowed     | boolean | Final gate  |
| fallback_active    | boolean | Static mode |
| last_safe_state_id | string  | Audit trail |

### Evaluation Specific
| Field    | Type | Notes               |
| -------- | ---- | ------------------- |
| true_rul | int  | Not used online |

