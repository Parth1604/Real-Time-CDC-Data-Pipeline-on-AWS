# Real-Time-CDC-Data-Pipeline-on-AWS


![architecture-diagram](architecture-diagram.png)

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

