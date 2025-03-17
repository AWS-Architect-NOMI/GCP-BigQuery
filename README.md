# GCP-BigQuery
Real-Time Log Processing with GCP BigQuery
A large-scale e-commerce platform receives millions of requests daily. They want to:

Monitor and analyze logs in real-time.
Detect anomalies and security threats quickly.
Store structured logs in Google BigQuery for future analysis.
However, their current system:
❌ Stores logs in raw format, making queries inefficient.
❌ Lacks real-time insights, causing slow threat detection.
❌ Has manual intervention for data cleanup, increasing human errors.

To solve this, they decide to build an automated log processing pipeline using GCP services.
Main Task
The goal is to build an end-to-end pipeline that:
✅ Collects logs from Cloud Logging.
✅ Streams logs in real-time using Google Cloud Pub/Sub.
✅ Processes logs via Python-based Google Cloud Functions.
✅ Stores cleaned logs in Google BigQuery for analysis.

 Action (A)
🛠 Step 1: Set Up Cloud Logging to Export Logs
📌 Cloud Logging stores logs from different sources (VMs, APIs, GKE, Firewalls).
📌 We will export logs to Pub/Sub for real-time processing.

➡️ Create a Log Sink in GCP
1️⃣ Go to GCP Console → Navigate to Cloud Logging.
2️⃣ Click on "Logs Router" → Create Sink.
3️⃣ Set Sink Destination to Pub/Sub and create a new topic (e.g., log-stream).
4️⃣ Click Create Sink.

⏩ Now, logs will be automatically exported to Google Cloud Pub/Sub.

🛠 Step 2: Create a Pub/Sub Topic and Subscription
📌 Google Cloud Pub/Sub is a real-time messaging service. It will receive logs from Cloud Logging and send them to Cloud Functions.

➡️ Create a Pub/Sub Topic
1️⃣ Open GCP Console → Go to Pub/Sub → Create Topic.
2️⃣ Name it log-stream.

➡️ Create a Subscription
1️⃣ In Pub/Sub, create a new subscription (log-subscription).
2️⃣ Set Delivery Type to Pull (Cloud Functions will pull the data).
3️⃣ Click Create Subscription.

⏩ Now, logs are published in real-time to Pub/Sub.

🛠 Step 3: Create a Google Cloud Function for Processing Logs
📌 Google Cloud Functions (GCF) will:
✅ Read logs from Pub/Sub.
✅ Extract meaningful data (IP, status, request time, errors, etc.).
✅ Format logs into structured JSON.
✅ Insert cleaned logs into BigQuery.

➡️ Write Cloud Function Code (main.py)
python
Copy code
import base64
import json
import datetime
import google.cloud.bigquery as bq

def process_logs(event, context):
    """Triggered from a message on a Pub/Sub topic."""
    
    # Decode Pub/Sub Message
    pubsub_message = base64.b64decode(event['data']).decode('utf-8')
    log_entry = json.loads(pubsub_message)
    
    # Extract relevant fields
    log_data = {
        "timestamp": log_entry.get("timestamp", str(datetime.datetime.utcnow())),
        "log_level": log_entry.get("severity", "INFO"),
        "message": log_entry.get("textPayload", ""),
        "source": log_entry.get("resource", {}).get("type", "unknown"),
        "service_name": log_entry.get("resource", {}).get("labels", {}).get("project_id", "unknown"),
        "http_status": log_entry.get("httpRequest", {}).get("status", None),
        "ip_address": log_entry.get("httpRequest", {}).get("remoteIp", None)
    }
    
    # Insert into BigQuery
    client = bq.Client()
    table_id = "your-project-id.logs_dataset.logs_table"
    errors = client.insert_rows_json(table_id, [log_data])
    
    if errors:
        print(f"BigQuery Errors: {errors}")
    else:
        print("Log successfully inserted into BigQuery")
➡️ Deploy Cloud Function
1️⃣ Open GCP Console → Cloud Functions → Create Function.
2️⃣ Set Trigger Type to Pub/Sub and select log-stream.
3️⃣ Choose Python 3.9 runtime.
4️⃣ Upload the main.py file.
5️⃣ Click Deploy.

⏩ Now, the function will process logs and push structured data to BigQuery.

🛠 Step 4: Create a Google BigQuery Table for Storing Logs
📌 Google BigQuery will store logs in a structured, queryable format.

➡️ Create a Dataset
1️⃣ Open BigQuery Console → Create Dataset (logs_dataset).

➡️ Create a Table (logs_table)
1️⃣ Open the dataset → Click Create Table.
2️⃣ Set schema fields:

timestamp → TIMESTAMP
log_level → STRING
message → STRING
source → STRING
service_name → STRING
http_status → INTEGER
ip_address → STRING
3️⃣ Click Create Table.
⏩ Now, structured logs are stored in BigQuery.

🎯 Business Impact:
✅ Real-time log ingestion with zero latency.
✅ Query logs in seconds using BigQuery.
✅ Detect anomalies instantly via structured log analysis.
✅ Improved security by monitoring suspicious activities (IP blacklisting).
✅ Automated & Serverless pipeline with no manual intervention.

ou can now run SQL queries to analyze logs, for example:

🔹 Find all 500 errors in the last 24 hours:

sql
Copy code
SELECT timestamp, message, http_status, ip_address
FROM logs_dataset.logs_table
WHERE http_status >= 500
AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY);
🔹 Detect suspicious IP addresses:

sql
Copy code
SELECT ip_address, COUNT(*) as request_count
FROM logs_dataset.logs_table
WHERE http_status >= 400
GROUP BY ip_address
ORDER BY request_count DESC
LIMIT 10;
🌟 Conclusion
🚀 We built a real-time log processing pipeline with:
✅ Cloud Logging for log collection.
✅ Pub/Sub for real-time streaming.
✅ Cloud Functions for serverless log processing.
✅ BigQuery for fast log queries & security analysis.

💡 This approach ensures scalability, automation, and real-time insights for any enterprise.

