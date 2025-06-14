# Data Intern Assessment – Event Replay and Recovery

## 📘 Overview
In an event-driven system without a traditional database, recovering missed or incorrectly processed events requires a robust strategy rooted in immutability and replayability. This solution focuses on leveraging archived raw event data to ensure accurate reprocessing and correction of system state.

---

## 🧠 Summary of Approach
My approach is based on **event log replay** using archived data stored in systems like Amazon S3, Google Cloud Storage, or Kafka with long-term retention. The strategy includes:

- Identifying the error window using monitoring dashboards and logs.
- Reprocessing missed events in an isolated environment using deterministic, idempotent logic.
- Comparing recalculated results with the current system state and applying atomic corrections as needed.

This ensures accuracy, auditability, and minimal disruption to the live system.

---

## ⚙️ Tools, Strategies, and Techniques

- **Durable Event Storage**: S3, GCS, Kafka (long-term retention)
- **Batch Reprocessing**: Apache Spark, Flink, or Python-based replayers
- **Isolation**: Dedicated VM or batch cluster for safe processing
- **Validation**: Checksums, hashes, and deterministic logic comparison
- **Transactional Updates**: Ensures consistent live state correction

---

## 🧪 Python Code Sample (Simulated Replay)

```python
import pandas as pd
from datetime import datetime
import hashlib

event_log = pd.read_csv("archived_events.csv", parse_dates=["timestamp"])
failure_start = datetime(2024, 6, 10, 12, 0)
failure_end = datetime(2024, 6, 10, 12, 30)

missed_events = event_log[
    (event_log["timestamp"] >= failure_start) &
    (event_log["timestamp"] <= failure_end)
].copy()

def process_events(events_df):
    results = (
        events_df.groupby("user_id")["value"]
        .sum()
        .reset_index()
        .rename(columns={"value": "recalculated_value"})
    )
    results["checksum"] = results.apply(
        lambda row: hashlib.md5(f"{row.user_id}-{row.recalculated_value}".encode()).hexdigest(),
        axis=1
    )
    return results

recalculated_results = process_events(missed_events)
existing = pd.read_csv("existing_results.csv")
merged = pd.merge(
    recalculated_results,
    existing,
    on="user_id",
    suffixes=("_new", "_old"),
    how="outer"
)

merged["discrepancy"] = merged["recalculated_value"] != merged["value_old"]
corrections_needed = merged[merged["discrepancy"] == True]
corrections_needed[["user_id", "recalculated_value"]].to_csv("corrections_to_apply.csv", index=False)
```

---

## 🔄 Trade-offs and Limitations

- Storage cost for archived events.
- Operational complexity for setup and maintenance.
- Requires idempotent processing logic.
- Batch reprocessing introduces some latency.

---

## 🔧 Enhanced with More Tools
With access to a database or real-time logs:
- Faster querying and correction.
- Easier integration of CDC (Change Data Capture).
- More automated anomaly detection.

---

## 🚀 Scaling to Millions of Events per Hour

This architecture supports scale via:
- Distributed processing (Spark/Flink)
- Partitioned Kafka replay
- Stateless, parallel consumer groups
- Workflow automation using Airflow or Dagster

---

## 📂 Files in This Repository

- `replay_worker.py`: Python script for replay and recalculation
- `archived_events.csv`: Simulated input event logs
- `existing_results.csv`: Simulated live system state
- `README.md`: This file

---

## 👤 Author

**Deva Deekshith Battala**  
Email: bdevadeekshith2024@gmail.com  
GitHub: [Insert your GitHub URL]

