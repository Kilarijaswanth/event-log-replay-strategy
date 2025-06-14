# Data Intern Assessment – Event Replay and Recovery

## 🌇 Scenario
In an event-driven system where historical event data isn't stored in a traditional database, recovering from missed or incorrectly processed events requires a specific strategy focused on the immutability and replay ability of events themselves.

---

## ✅ Explanation of Approach

My approach centres on the principle of event log replay using archived event data.

### 🔁 How to Recover and Back-Calculate:

1. **Event Archiving (Crucial Assumption)**: 
   All events emitted onto the event bus are durably archived in their raw, immutable form—stored in cloud object storage (e.g., S3, GCS), distributed file systems (e.g., HDFS), or message queues with high retention (e.g., Kafka). 
   - Best practices: Store in compressed, partitioned format (e.g., Parquet by date/hour) with event metadata.

2. **Identify the Error Window**: 
   Use monitoring tools and logs to detect the precise timeframe where failures occurred.

3. **Isolated Re-processing Environment**: 
   Spin up a batch cluster, VM, or even a local script to safely reprocess historical events.

4. **Event Log Retrieval**: 
   Use AWS Athena, Kafka offset replay, or GCS Dataflow to extract logs for the failure window.

5. **Batch Re-calculation**: 
   Feed the logs into the isolated system, applying original worker logic.
   - Ensure idempotency and disable side effects (e.g., notifications) during replay.

6. **Comparison and Correction**: 
   Compare recalculated vs. current live state.
   - Use checksums or hashes for comparison.
   - Patch only mismatched data.

7. **Atomic Update**: 
   Apply corrections using transactional writes to prevent partial updates.

---

## ⚙️ Tools, Strategies, or Techniques:

- Durable Event Storage (e.g., S3, GCS, Kafka with long retention)
- Replay from known offset/timestamp
- Temporary in-memory store (e.g., Python dictionary)
- Batch frameworks: Spark, Flink, Python/Go scripts
- Idempotent logic and safe retry handling
- Versioned Calculations (for backward-compatible logic)
- Validation/Reconciliation (before-after comparisons)

---

## ✅ Ensuring Accuracy and Consistency:

- Immutable Source: Archived event logs are the source of truth
- Deterministic Logic: Same input always yields the same output
- Isolation: Re-processing doesn't affect live system
- Thorough Testing: Validate against known results first
- Transactional Updates: All-or-nothing corrections

---

## 🐍 Python Code for Event Log Replay

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

## 💡 Another Simplified Approach (Prototype Style)

### Assumptions:

- Events are time-stamped
- Each event has a unique ID
- Access to flat logs or replay system
- Stateless worker logic
- Bug or downtime caused data loss

### Tools:

- Kafka / File Logs / AWS Kinesis
- Python with Pandas
- Hashing/Versioning
- Unit Tests

### Python Code:

```python
import pandas as pd
from datetime import datetime

event_log = pd.read_csv("events_log.csv", parse_dates=["timestamp"])
failure_start = datetime(2024, 6, 10, 12, 0)
failure_end = datetime(2024, 6, 10, 12, 30)

missed_events = event_log[(event_log["timestamp"] >= failure_start) & 
                          (event_log["timestamp"] <= failure_end)]

recalculated_metrics = missed_events.groupby("user_id")["value"].sum().reset_index()
print("Recalculated Metrics During Failure:")
print(recalculated_metrics)
```

---

## 🧾 Comparison Table

| Criteria              | Your Approach                                | Initial Approach (ChatGPT)            |
|----------------------|-----------------------------------------------|---------------------------------------|
| Source of Recovery   | Archived logs in S3, Kafka, HDFS              | Log file like CSV or replayable stream|
| Replay Strategy      | Time- or offset-based using tools             | Basic filtering in Pandas             |
| Isolation            | Isolated environment (cluster/VM)             | Local script                          |
| Accuracy Mechanisms  | Idempotency, hashing, versioning              | Post-hoc checks                       |
| Scalability          | Spark/Flink/Dataflow                          | Low volume                            |
| Consistency Handling | Transactional updates and diffs               | Overwrites                            |
| Production Readiness | Enterprise-grade                              | Prototype/demo                        |
| Tools Used           | Kafka, Spark, Athena, Flink                   | Python, Pandas                        |

---

## ✅ Summary and Discussion

**Summary of Approach and Rationale:**  
My approach is centered on the concept of **event log replay**, leveraging durable, immutable archives of event data as the ultimate source of truth. It ensures safe recovery by isolating replay, applying deterministic logic, and enabling comparison before applying corrections.

**Trade-offs and Limitations:**  
- Increased storage cost for logs  
- Operational complexity  
- Assumes idempotency  
- Slight latency due to batch recalculation

**With Access to More Tools:**  
I’d use SQL/CDC tools, integrate automated workflows (e.g., Airflow), and leverage real-time logs for faster recovery and rollback.

**Scalability:**  
Can scale using distributed data systems (Spark/Flink), partitioned event replay (Kafka), and orchestration (Airflow, Dagster) to support millions of events/hour.

---

## 📂 Repository Contents

- `replay_worker.py`: Python replay script
- `archived_events.csv`: Sample logs
- `existing_results.csv`: Simulated live data
- `README.md`: Documentation

---

## 👤 Author

**Deva Deekshith Battala**  
Email: bdevadeekshith2024@gmail.com  
GitHub: [Insert your GitHub URL]  
