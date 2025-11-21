---
layout: single
title: "What is a feature store and why do we use it?"
date: 2025-11-21 13:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-11-21-what-is-a-feature-store
---

# What is a feature store and why do we use it?

Why do we even need a feature store for our model pipeline? Why can't we just have a pipeline where we keep everything in object storage and call it at runtime? Why even have a feature store?

## Example problem

Let's say that we're doing a simple classification task for the classic [Titanic dataset from Kaggle](https://www.kaggle.com/competitions/titanic/data). Part of that task involves generating features for your model. This task is pretty simple if you're doing a one-off training regime, but once you start wanting to productionize your model, an inevitable tool that you'll start wanting to use is a **feature store**.

## What is a feature store?

A feature store is a centralized data system that manages, stores, and serves the input features used in machine learning models, ensuring consistency and reusability across both training and production environments.

By separating feature engineering and management from the rest of the ML pipeline, it allows data scientists and engineers to easily access validated, up-to-date features without duplicating work or risking inconsistencies.

Some examples of common feature store solutions include Google Vertex AI Feature Store, Feast, AWS SageMaker Feature Store, and Databricks Feature Store. These tools provide built-in capabilities for feature versioning, sharing, and serving, helping organizations quickly operationalize ML projects and maintain robust, scalable pipelines.

## What would our pipeline look like without a feature store?

Let's say you didn't want to use a feature store but you wanted to start doing ML engineering in the cloud anyways. What would that look like? One possible way is something like this:

Files stored in Cloud Storage (GCS) as the primary interface between steps. Logic for processing data is often duplicated between the training phase and the inference phase.

### [Naive approach] System Components

- Storage: Google Cloud Storage (GCS) buckets (stores CSV/Parquet files).
- Compute: Vertex AI Training Jobs (or local Python scripts).
- Model Registry: Vertex AI Model Registry (stores the trained .bst or .pkl file).
- Serving Endpoint: Vertex AI Endpoint (hosts the model).
- Client/UI: Streamlit App.

### [Naive approach] The Workflow (Pipeline Design)

#### Data Ingestion & Processing

You run a script (preprocess.py) that downloads raw data, cleans it (fills missing ages, encodes titles), and saves a file processed_train.csv to GCS. Artifact: A static file in a bucket.

#### Model Training

- The training script (train.py) downloads processed_train.csv from GCS.
- It trains the XGBoost model and saves the model artifact to the Model Registry.

#### Inference (Serving) - The "Danger Zone"

- A user on the Streamlit app enters raw data (e.g., "Age: 22, Title: Mr").
- CRITICAL: The Streamlit app (or the serving container) must contain a copy of the logic from preprocess.py to convert "Mr" to the exact integer 1 that the model expects, and fill missing values exactly as the training set did.
- The app sends this processed vector to the Endpoint for a prediction.

### Where does this naive approach go wrong?

#### Training-Serving Skew (The #1 Silent Killer)

You calculate Family_Size = SibSp + Parch + 1 in your training script. When you build your Streamlit UI, you have to re-write that same logic in Python inside the app. If you update one and forget the other, your model fails silently because the inputs don't mean the same thing.

#### Point-in-Time Correctness (Time Travel)

Your CSV in the bucket just has the latest data. If you want to retrain a model to see how it would have performed last month, you can't easily "rewind" the data in a CSV to what it looked like on Nov 1st.

#### Online Serving Latency

To serve predictions, your app would need to load the CSV or query a slow database to find passenger traits.

### What benefits would another approach ideally have?

An alternative approach would ideally resolve the problems of the naive approach:

#### Solving Problem 1: Training-Serving Skew (The #1 Silent Killer)

Ideally, you compute the feature once, store it, and then both the training job and the Streamlit app ask the Feature Store for "Passenger 123's Family_Size". That way, they're guaranteed to get the exact same definition.

#### Solving Problem 2: Point-in-Time Correctness (Time Travel)

Ideally, it stores a history of values. You can ask: "Give me the training data as it existed on Nov 1st." This prevents data leakage (training on future information).

### Solving Problem 3: Online Serving Latency

Ideally, it has an Online Store (low-latency, like Redis/Bigtable) specifically for serving. When your Streamlit app asks for features, it gets them in milliseconds.

## How does a feature store work?

At a high level, a feature store sits between raw data sources and model consumers. Teams register feature definitions in a central registry (names, owners, data types, freshness SLAs, and transformation code).

### How do we add data to a feature store?

Ingestion jobs (batch and/or streaming) compute features from raw events and operational tables. The platform writes immutable, time-stamped feature values to an offline store for training. It also maintains lineage and event times to guarantee point-in-time correctness ("data was correct up to this date") and so that any training set can be traced back to the source data and transformation code used to generate it.

The feature store also manages adding the latest values from the online store to an online store with very low-latency lookups (e.g., Redis).

### How do we serve the data?

#### Offline serving

When models are trained, they can request "as-of" datasets (e.g,. all the data up to 2025-10-05") and the feature store assembles that dataset by getting the state of the data up to that point from the offline store.

#### Online serving

For inference, we need data immediately. For this, the feature store serves the data from the online store.

## Benefits of a feature store

The primary benefit is consistency: compute a feature once, use it everywhere. This eliminates training–serving skew, accelerates iteration by enabling reuse across teams, and makes features discoverable through a catalog with ownership and documentation. Reproducibility improves because every value is versioned and time‑stamped, so you can rebuild yesterday’s training set exactly and audit how a model was produced.

Operationally, a feature store reduces cost and complexity by centralizing pipelines, handling backfills and schema evolution, and separating batch analytics from low‑latency serving through dedicated offline/online stores.

Performance also improves via pre‑materialization and caching, while governance improves through RBAC, data contracts, and PII policies applied consistently across training and serving. The result is faster model delivery with fewer incidents and clearer accountability, across both training and serving.

## What exactly is a feature store under the hood? How would you create one yourself?

A feature store is essentially two specialized databases working together, plus some orchestration logic. It's not magic, it's just databases optimized for specific ML use cases.

### The Two-Database Architecture

#### 1. Offline Store (Training Database)

- **Technology**: Usually a data warehouse like BigQuery, Snowflake, or Hive
- **Purpose**: Stores historical feature data for training
- **Optimized for**: High-throughput batch reads, time-travel queries
- **Storage format**: Columnar (Parquet, ORC) for analytical queries

#### 2. Online Store (Serving Database)

- **Technology**: Key-value stores like Redis, DynamoDB, Cassandra, or Bigtable
- **Purpose**: Serves features for real-time inference
- **Optimized for**: Sub-millisecond lookups by entity ID
- **Storage format**: Row-based for fast single-record retrieval

#### The "Secret Sauce" Components

Beyond just databases, feature stores add:

- **Sync Logic**: Automatically copies data from offline to online databases
- **Time-Travel Engine**: Reconstructs "what the data looked like on date X"
- **Schema Registry**: Tracks feature definitions and versions
- **Transformation Engine**: Applies consistent feature engineering logic

### Creating a feature store from scratch

A simple implementation of a feature store would look something like this:

```python
# Simplified Feature Store Architecture
class DIYFeatureStore:
    def __init__(self):
        # Offline Store: PostgreSQL with time-series tables
        self.offline_db = PostgreSQLConnection()
        
        # Online Store: Redis for fast lookups
        self.online_cache = RedisConnection()
        
        # Metadata Store: Track feature definitions
        self.registry = FeatureRegistry()
```

For the offline store, it would look something like this:

```sql
-- Historical features table (PostgreSQL)
CREATE TABLE passenger_features (
    entity_id VARCHAR(50),           -- PassengerId
    feature_timestamp TIMESTAMP,     -- When this data was valid
    ingestion_timestamp TIMESTAMP,   -- When we learned about it
    age DOUBLE PRECISION,
    fare DOUBLE PRECISION,
    family_size INTEGER,
    -- ... other features
    PRIMARY KEY (entity_id, feature_timestamp)
);

-- Index for time-travel queries
CREATE INDEX idx_time_travel ON passenger_features (feature_timestamp, entity_id);
```

For the online Redis store, it would look something like this:

```json
# Redis key structure for fast lookups
# Key: "passenger:892:latest"
# Value: JSON with all features
{
    "age": 22.0,
    "fare": 7.25,
    "family_size": 2,
    "last_updated": "2024-11-21T10:30:00Z"
}
```

For the core feature store logic, it would look something like this, with functionalities for (1) ingesting features, (2) getting training data for offline queries (e.g., ML training), and (3) getting online features, for inference.

```python
class DIYFeatureStore:
    def ingest_features(self, df, entity_col, timestamp_col):
        """Ingest features into both stores"""
        # 1. Store in offline store (PostgreSQL)
        df.to_sql('passenger_features', self.offline_db, if_exists='append')
        
        # 2. Update online store (Redis) with latest values
        for _, row in df.iterrows():
            entity_id = row[entity_col]
            features = row.drop([entity_col, timestamp_col]).to_dict()
            
            redis_key = f"passenger:{entity_id}:latest"
            self.online_cache.set(redis_key, json.dumps(features))
    
    def get_training_data(self, entity_ids, as_of_time):
        """Time-travel query for training"""
        query = """
        SELECT DISTINCT ON (entity_id) *
        FROM passenger_features 
        WHERE entity_id = ANY(%s) 
          AND feature_timestamp <= %s
        ORDER BY entity_id, feature_timestamp DESC
        """
        return pd.read_sql(query, self.offline_db, 
                          params=[entity_ids, as_of_time])
    
    def get_online_features(self, entity_ids, feature_names):
        """Fast lookup for serving"""
        results = []
        for entity_id in entity_ids:
            redis_key = f"passenger:{entity_id}:latest"
            features = json.loads(self.online_cache.get(redis_key) or '{}')
            results.append({k: features.get(k) for k in feature_names})
        return results
```

## Revised architecture after including a feature store

We can redesign our architecture to now use a feature store. This new design introduces a single central "source of truth" for all feature data. The feature store handles serving data to both the model trainer and the inference app, making sure that both are consistent.

### [Revised approach] System Components

- Storage: Google Cloud Storage (GCS) buckets (stores CSV/Parquet files).
- **Feature Store**: Vertex AI Feature Store.
- **Offline Store**: High-capacity storage (BigQuery-backed) for historical training data.
- **Online Store**: Low-latency storage (Bigtable/Redis-backed) for real-time serving.
- Compute: Vertex AI Training Jobs (or local Python scripts).
- Model Registry: Vertex AI Model Registry (stores the trained .bst or .pkl file).
- Serving Endpoint: Vertex AI Endpoint (hosts the model).
- Client/UI: Streamlit App.

### [Revised approach] The Workflow (Pipeline Design)

#### [Revised approach] Feature Engineering & Ingestion

- You run preprocess.py to calculate features (Age, Family_Size).
- Instead of just saving a CSV, you ingest these values into the Feature Store, tagged with a timestamp and an Entity ID (PassengerId).
- The Feature Store automatically syncs this data to both the Offline and Online storage layers.

#### [Revised approach] Model Training (Batch Fetch)

- The training script (train.py) does not read a CSV. It sends a query to the Offline Feature Store: "Give me the features for these 800 passengers as they looked on Jan 1st."
- The Feature Store reconstructs the dataset for that point in time.
- The model is trained and saved.

#### [Revised approach] Inference (Online Lookup)

- Scenario A (Known Passenger): The Streamlit app only needs the PassengerId. It asks the Online Feature Store: "Give me the latest features for Passenger 892." It gets the pre-computed, trusted values in milliseconds and sends them to the model.
- Scenario B (New/Hypothetical Passenger): You can still manually pass values, but typically in mature systems, you would ingest the new data into the Feature Store first, or use the Feature Store's transformation layer (if available) to ensure the logic is identical.

### What are the benefits of this revised approach?

### Zero Skew

The exact value used to train the model (e.g., Family_Size=4) is the exact value returned by the Online Store for serving.

### Latency

The Online Store is optimized for millisecond retrieval, unlike reading from GCS or querying a standard SQL database.

### Time Travel

You can re-train the model on the state of data from 6 months ago without needing to keep old CSV files around; the Offline Store manages the timeline for you.

## What would this look like in GCP?

To set up our feature store in GCP, we would do something like this:

### Setting up and creating the feature store

```python
from google.cloud import aiplatform

# Initialize SDK
aiplatform.init(project="titanic-ml-project", location="us-central1")

# 1. Create the Feature Store
# This is the container for all your data
titanic_fs = aiplatform.Featurestore.create(
    featurestore_id="titanic_featurestore",
    online_store_fixed_node_count=1,  # Low cost for demo
    project="titanic-ml-project",
    location="us-central1",
    sync=True  # Wait for creation (can take ~10 mins)
)

# 2. Create an Entity Type
# "Passenger" is the entity we are tracking
passenger_entity = titanic_fs.create_entity_type(
    entity_type_id="passenger",
    description="Titanic passenger features",
    sync=True
)

# 3. Register Features
# Define the schema of what you are storing
passenger_entity.batch_create_features(
    feature_configs={
        "age": {"value_type": "DOUBLE"},
        "fare": {"value_type": "DOUBLE"},
        "family_size": {"value_type": "INT64"},
        "pclass": {"value_type": "INT64"},
        "has_cabin": {"value_type": "BOOL"},
    },
    sync=True
)
```

### Ingestion (Moving data from GCS to Feature Store)

You need a .csv in GCS with columns matching your features, plus `entity_id` (PassengerId) and `feature_timestamp` (when this data was known).

```python
# 4. Ingest Data from GCS
# Your CSV must have a column for the entity ID and a timestamp column
passenger_entity.ingest_from_gcs(
    feature_ids=["age", "fare", "family_size", "pclass", "has_cabin"],
    feature_time="timestamp",  # Name of the timestamp column in your CSV
    entity_id_field="PassengerId", # Name of the ID column in your CSV
    gcs_source_uris=["gs://titanic-ml-data-YOUR_PROJECT_ID/data/processed/features.csv"],
    sync=True
)
```

### Serving

For serving, at runtime (e.g., in a Streamlit UI), we'd do something like this:

```python
# 5. Online Serving (Low Latency)
# Retrieve features for a specific passenger (e.g., ID 892)
feature_values = passenger_entity.read(
    entity_ids=["892"],
    feature_ids=["age", "fare", "family_size", "pclass", "has_cabin"]
)

print(feature_values) 
# Output: Pandas DataFrame with the latest values for Passenger 892
```

## Summary

A feature store is not a single database. Instead, it's a set of services that standardize how features are defined, computed, versioned, and served to both training and production. It may not be apparent at first why one needs to use a feature store, but in production, problems like training–serving skew, reproducibility, and latency come up, problems that feature stores are designed to solve.
