# Real-Time-CDC-Data-Pipeline-on-AWS


![Architecture Diagram](Real%20-Time%20CDC%20pipeline.png)
 

---

## 🎯 Problem Statement

In retail, it’s crucial to process order changes as they happen — whether a customer cancels, updates their address, or adds a promo code. This project replicates such a scenario, capturing **live updates to customer orders** and pushing them to a data lake where they can be queried, analyzed, or acted upon in real time.

---

## 🧱 Services Used

| AWS Service        | Purpose                                          |
|--------------------|--------------------------------------------------|
| **DynamoDB Streams** | Captures CDC events (INSERT, MODIFY, DELETE)    |
| **EventBridge Pipe** | Filters and routes stream records to Kinesis    |
| **Kinesis Data Streams** | Provides scalable, real-time streaming buffer |
| **Kinesis Firehose** | Buffers, retries, and delivers to destination   |
| **Lambda (Python)**  | Transforms records (flattens JSON, enriches data) |
| **Amazon S3**       | Final data lake storage (query-ready)           |
| **Athena** (optional) | To query processed data                        |

---

## ⚙️ How It Works

1. **Data is inserted/updated** in DynamoDB’s `CustomerOrders` table.
2. **DynamoDB Streams (with NewImage)** emits the full updated record.
3. **EventBridge Pipe** sends filtered records to **Kinesis Data Stream**.
4. **Firehose** buffers the data (1 MB or 60 seconds) and sends to **Lambda**.
5. **Lambda function** transforms and cleans the data.
6. Final output is stored in **S3** as partitioned JSON/Parquet for querying.

---

## 🧪 How to Test

```bash
# Simulate a new order (use Boto3 or AWS Console)
PUT /CustomerOrders {
  "order_id": "123",
  "status": "processing",
  "total": 79.99
}

# Modify the order
UPDATE /CustomerOrders {
  "order_id": "123",
  "status": "cancelled"
}

```
---

## 💡 What I Learned

- Gained hands-on experience with **real-world event-driven pipeline design**
- Understood how **Change Data Capture (CDC)** works using **DynamoDB Streams**
- Learned to **tune Kinesis Firehose buffering** to balance **latency vs cost**
- Designed for **resiliency**, incorporating **failure handling and retry logic** across components


---

## 🔮 Future Enhancements: Apache Hudi Integration

To evolve this real-time ingestion pipeline into a full **lakehouse architecture**, the next enhancement will be the integration of **Apache Hudi** using AWS Glue.

This will allow us to manage the data lifecycle more effectively — supporting upserts, deletes, incremental queries, and schema evolution — while keeping S3 query-optimized and analytics-ready.

### ✅ Planned Enhancements:

- [ ] Build an **AWS Glue PySpark job** to read raw JSON data from S3
- [ ] Write cleaned and deduplicated data into **Apache Hudi tables** on S3 (Copy-On-Write)
- [ ] Use `order_id` as the **primary key** for upserts
- [ ] Enable **partitioning** (e.g., by `order_date` or `status`) to boost Athena performance
- [ ] Register the Hudi table in the **Glue Data Catalog**
- [ ] Allow Athena & QuickSight to query **real-time snapshots** of the order data
- [ ] Implement **schema evolution** to support changes in order structure
- [ ] Leverage **Hudi commit metadata** for **incremental ETL** and CDC-based processing
- [ ] Add logic to handle **deletes** from DynamoDB Streams properly
- [ ] (Optional) Orchestrate the pipeline with **AWS Step Functions** or **EventBridge Triggers**

### 📊 Benefits of Adding Hudi:

| Feature            | Benefit                                                   |
|--------------------|------------------------------------------------------------|
| ✅ Upserts & Deletes | Keeps S3 data lake in sync with latest DynamoDB state     |
| 🧠 Time Travel       | Query historical snapshots of the data                    |
| ⚙️ Incremental Pulls | Efficient downstream Glue/Athena jobs using Hudi commits  |
| 🏗️ Lakehouse Format  | Brings warehouse-like capabilities to your S3 bucket      |
| 🔄 Schema Evolution  | Supports flexible data changes without breaking pipelines |

---

