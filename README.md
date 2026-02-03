# aws-ml-timeseries-pipeline

## Required Skills
1. Data ingestion & feature engineering
2. Training pipelines
3. Model evaluation & tuning
4. Deployment & inference
5. Monitoring & iteration
6. Cost, reliability, and failure modes

---

## How to Run the Demo
* Local setup
* Docker instructions
* Expected outputs

---

## Overview
Subject system is a mechanical, rotating industrial article operating under multiple regimes and supervised by a control layer. 
ML provides advisory integrity estimates, and some gated control actions.

### Problem Statement 
Mechanical degradation in rotating assets progresses under varying operating regimes and rates. Only indirect health measurements is available to supervisory control systems, limiting early fault detection and increasing reliance on schedule-based maintenance. Use of schedule-driven maintenance provides minimal mitigation of risks associated with catastrophic failures.

### Objectives
* Build visibility into current operating state and asset health.
* Enable more proactive maintenance planning to reduce degradation-related structural failures.
* Preserve safe operation when state of health is uncertain or actively degrading.

### Deliverables
* 

### Success Criteria
* Drift events detected within a defined evaluation window.
* Model confidence correlates with degradation trends under simulated fault conditions.
* Control parameters stable under injected sensor loss and inference latency.
* Availability of alerts and diagnostics through visualization dashboard.

### Planned Project Phases
* Hardware-in-the-loop
* Containerization
* Service boundaries
* Potential cloud targets
* CI/CD outline
* Visualization/UI roadmap

### Subject System Architecture
High-level diagram with:
* Sensors & PLC
* Safety Controller
* Data pipelines
* ML model
* PID tuner
* Cloud and enterprise integration

![System Architecture](docs/diagrams/systemArchitecture.svg)


## Solution Design
* Control strategy
* ML role
* Drift detection approach
* Safety guardrails

### Data
* Manufacturing, multivariate time series
* High-frequency sensor signals (temperature, pressure, speed, vibration) 
* Synthesized sensor loss (dropout) for model training
	* Random sensor blackout windows
	* Permanent sensor failure cases
	* Create a feature based on missing data
* Distribution Drift
	* Regime-specific mean shifts
	* Variable degradation rate
	* Step changes

Reference [Data Specification](data\datasetSpec.md)
#add_TODO: Assumptions and limitations

#### Raw Dataflow

![Global Raw Dataflow](docs/diagrams/flow_end_to_end.svg)
![Raw Data Ingestion](docs/diagrams/flow_raw_ingestion.svg)
![Raw Data Validation](docs/diagrams/flow_raw_validation.svg)
![Processed Data Generation](docs/diagrams/flow_processed_generation_validation.svg)
![Data Error Handling](docs/diagrams/flow_failure_observability.svg)


#### Training Dataset
* 1RegimeStable
* Description: Baseline multivariate time-series dataset representing a rotating industrial asset under stable operating conditions.
* Usage:  Evaluation of system-level behavior including, but not limited to data validation, drift detection, inference gating, and control integration. 
* Purpose: Establish stable baseline independent of operating-regime variability. It is valid for safe determination of impacts to target system by intentionally introducing sensor loss, distribution shift, and inference latency.

* A second dataset, MultiRegimeStable, may be introduced in later phases to evaluate robustness under multiple operating regimes once baseline system behavior is established.

### Model & Inference
* Initial implementations prioritize tree-based models for robustness and interpretability. Sequence models are considered exploratory and non-authoritative.
#TODO: Choose model type * considering XGBoost, LightGBM
* Feature engineering
* Inference cadence
* Latency considerations
  * Delayed inference
  * Out-of-sequence model outputs discarded
* Confidence Gating
	* Prevents use by control layer
	* Outputs below confidence threshold logged only

### Controller Logic
* Control loop flow
* Fail-safe behavior
* Rate limiting of parameter changes
* Fallback modes
* State transitions

### Project Structure
/data
  /processed
  /raw
/docs
  /diagrams
/infra
/notebooks
/src
  /etl
  /features
  /training
  /inference
  /monitoring
LICENSE
README.md
ARCHITECTURE.md
FAILURE_MODES.md
\TODO.md

---

## Release and Publication Considerations
This system is designed to operate in industrial environments where safety, reliability, and predictability are more important than model complexity.

### Safety & Control Boundaries
* Phase release
* Initial boundaries

### Latency & Timing
* ML inference is performed at a slower cadence than the PLC control loop
* Control tuning updates are rate-limited to prevent instability
* Loss of inference automatically freezes parameters at last known safe values

### Drift Detection & Model Validity
* Continuous monitoring of feature distributions
* signal drift detection triggers
  * Feature distribution
  * Sustained decrease in prediction confidence 
  * Statistical regime imbalance
* Alerts and intercessions
* Engineering review

### Failure Isolation
* ML inference, control logic, and observability decoupled
* Failure in the ML layer does not propagate
* Critical functions of subject system remain operational without cloud connectivity

### Observability & Auditability
* Recommended monitoring of ML model
* All tuning changes logged with timestamps, context and model confidence
* Metrics support post-event analysis and regulatory review
* Historical data enables offline model evaluation and improvement

NOTE: While the underlying sensor data originates from a publicly available benchmark, this project modifies the data and operating assumptions to evaluate system-level behavior under sensor loss, drift, and inference latency. The focus is not model accuracy in isolation, but safe integration of ML into a supervisory control context.

### Deployment
* Suitable for optional cloud integration
* Containerized components support repeatable builds and testing
* Designed to integrate with existing platforms

---

## Project Tools and Services

#### Analytics:
* Amazon Athena
  * Purpose:
    * Ad-hoc querying of raw and processed time-series data
    * Ingestion Data Analytics and SQL Query Access
    * Validation, inspection, and offline analysis
  * Justification:
    * Serverless, no cluster management
    * Native support for S3 + Parquet
    * Ideal for exploratory checks and audit queries
  * Alternatives considered:
    * Redshift: Scale of this dataset doesn't require operational complexity
    * EMR: No large-scale distributed processing
* Amazon Glue Data Catalog
	- Purpose
		- Document data storage contract
    - Schema Storage (Catalog): 
	- Justification:
		- notebook integration with Apache Spark
		- Ad-hoc analytics
	- Alternatives considered:
		- Amazon Timestream: real-time analysis not required for this project

#### Compute:
* AWS Lambda
  * Purpose:
    * Lightweight validation tasks
    * Event triggered ingestion quality verification
    * Process data workflows of singular events
  * Justification:
    * Lambda is triggered by events meeting intended usage for this dataset.
    * Cost-effectiveness for intermittent event evaluation
    * Serverless pipeline reduces overhead
  * Alternatives considered:
    * EKS: Orchestration complexity unnecessary for this phase
    * ECS: Overhead is excessive for validation workloads needed by this dataset

#### Containers:
*  Amazon Fargate
		- Purpose:
			- Containerize ETL jobs
			- Repeatable batch processing
			- Optional inference services (future use)
		- Justification:
			- No cluster management
			- Simplicity of use with balanced controls
			- Separation of infra and application code
		- Alternatives considered:
			- EKS: Orchestration complexity unwieldy for requirements at this point
			- Lambda: Runtime limits may be insufficient for ETL 
* Amazon ECR
	- Purpose
		- Container Artifact Storage
		- workflow and lifecycle automation
	- Justification:
		- fully managed container image registry
		- operational simplicity and scaleability for planned architecture 
		- automation via APIs and CLI tools
	- Alternatives considered:
		- EBS/EFS: lack native support for image versioning workflows and IAM-based access control
* Amazon ECS
	- Purpose
		- Domain Validation Execution
	- Justification:
		- Validation logic is stateful, CPU-bound, and deterministic, not event-driven.
		- Containerized validation guarantees:
		- Reproducibility (same image → same validation)
		- Isolation from ingestion and analytics layers
		- Versioned validation logic tied to dataset versions
		- ECS allows validation to scale horizontally without embedding logic into Glue or Lambda.
		- Architecturally clean separation between schema validation (Glue) and domain validation (ECS).
	- Alternatives considered: 
		- AWS Lambda: Poor fit for long-running or memory-heavy validation, difficult versioning and logic reproducability
		- AWS Glue Jobs: Overkill for deterministic rule checks, Ties validation too tightly to Spark
		- EC2: Operational overhead, No benefit for a design-first pipeline


#### Machine Learning:
* Amazon SageMaker
		- Purpose
			- Model training
			- Offline inference
			- Managed ML lifecycle
		- Justification:
			- Separation between training and inference
			- Model versioning and experiment tracking
		- Alternatives considered:
			- Real-time endpoints: ML intended for advisory use instead of live changes to control layer
			- Amazon Forecast: business application oriented
*  Amazon Bedrock
*  Amazon Comprehend
*  AWS Deep Learning AMIs (DLAMI)
*  Amazon Forecast
*  Amazon Fraud Detector
*  Amazon Lex
*  Amazon Kendra
*  Amazon Mechanical Turk
*  Amazon Polly
*  Amazon Q
*  Amazon Rekognition
*  Amazon SageMaker
*  Amazon Textract
*  Amazon Transcribe
*  Amazon Translate
*  Amazon RDS (Relational Database Service)
*  Amazon DynamoDB
*  Amazon Aurora

#### Management and Governance:
*  AWS CloudTrail
	- Purpose
		- Compliance verification and audit logging
		- Model and data access traces
	- Justification:
		- Supports observability and auditability
		- High return, despite no real-time inbound traffic
	- Alternatives considered:
		- Third-party tools (Datadog, Prometheus): Outside AWS, Unnecessary complexity
*  Amazon CloudWatch
	- Purpose
		- Monitoring workflows
    - Centralized logs and metrics for ingestion, validation, and future ML jobs.
		- Alerting on failures, data quality breaches, and pipeline anomalies.
	- Justification
		- Architecture credibility showing operational truth.
		- Audit trails
		- Post-mortem analysis
		- Supports logs, metrics, alarms without additional infrastructure.
		- Enables decoupling of monitoring from execution logic.
	- Alternatives considered
		- CloudTrail: Governance-only (API calls, not runtime behavior)
		- Third-party tools (Datadog, Prometheus): Outside AWS, Unnecessary complexity

#### Networking and Content Delivery: 
*  Amazon VPC 
*  AWS IAM (Identity and Access Management)
*  AWS KMS (Key Management Service)

#### Security, Identity, and Compliance:
* Access Control: AWS IAM
	- Purpose
		- Least-privilege access control
		- Separation of ingestion, training, and monitoring roles
	- Justification:
		- AWS architecture credibility
		- Provide support for auditability
	- Alternatives considered: NONE

#### Storage:
*  Amazon S3
		- Purpose
			- Ingestion Data Destination
      - Data repository
			- Versioning processed datasets
			- Validation outputs and logs
			- Apply lifecycle policies
		- Justification:
			- Durable and immutable when versioned
			- Long-term retention without operational overhead
			- Industry standard for ML pipelines
			- Native integration with Athena, SageMaker
			- enables offline evaluation
		- Alternatives considered:
			- Timestream: Ingestion will be batch instead of real-time telemetry
			- RDS: Time-series data files unsuited for required schema rigidity



## Results

The self-tuning controller was evaluated using high-frequency simulated process data designed to reflect real operating variability, sensor noise, and drift conditions commonly observed in industrial batch systems.

### Quantitative Outcomes
* 

### Control Behavior Improvements
* 

### Operational Impact
* 

---



## Failure Modes & Mitigations

### 1. Sensor Noise or Dropout
**Failure Mode**  
Transient or sustained loss of sensor signal quality.

**Mitigation**
* Signal filtering and validation at ingestion
* ML inference suppressed when data quality thresholds are violated
* Controller reverts to static, validated tuning parameters


### 2. Model Drift Due to Process Changes
**Failure Mode**  
Process behavior changes due to feedstock variation, fouling, or equipment wear.

**Mitigation**
* Drift detection monitors feature distribution shifts
* Adaptive tuning is gated when confidence degrades
* Alerts generated for engineering review before retuning


### 3. Latency Spikes or Inference Delays
**Failure Mode**  
Delayed ML outputs due to compute or network issues.

**Mitigation**
* Control loop operates independently of ML inference timing
* Parameter updates are applied only within safe timing windows
* Stale inference results are discarded automatically


### 4. ML Model Misclassification or Overconfidence
**Failure Mode**  
Model produces confident but incorrect state estimates.

**Mitigation**
* Confidence thresholds required for tuning adjustments
* Rate-limited parameter changes prevent sudden control shifts
* Human override remains available at all times


### 5. Cloud or Network Outage
**Failure Mode**  
Loss of connectivity to cloud monitoring or model services.

**Mitigation**
* Edge controller continues operating with last validated configuration
* No dependency on cloud availability for safe operation
* Cloud layer used for observability and analysis only

---
EOF
