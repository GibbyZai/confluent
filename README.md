***

# Confluent Cloud Hands-on Lab

End-to-end lab for getting started with Confluent Cloud using a simple e-commerce use case (`orders`, `payments`, `customers`). Suitable as a GitHub `README.md` or workshop guide.


## **Table of Contents**
1. Prerequisites
2. Lab Overview
3. Create a Confluent Cloud Account
4. Create Initial Environment and Cluster
5. Create a Topic and Use the UI to Produce/Consume
6. Set Up Dev and Prod Environments and Clusters
7. Design and Create the Topic Set
8. Validate Partitions and Replication
9. Generate API Keys
10. Run a JSON Producer (Orders)
11. Run Consumers and Observe Consumer Group Behavior
12. Introduce a Schema Change
13. (Optional) Use the Confluent CLI


## Prerequisites
- Modern web browser
- Terminal (bash, zsh, or PowerShell)
- Python 3.8+ installed
- Ability to install Python packages
- (Optional) Confluent CLI installed and configured


## Lab Overview
In this lab you will:

- Create a **free Confluent Cloud account** (or use a shared workshop org).
- Create an **initial environment and cluster**.
- Create a **topic** and **produce/consume** data using the Confluent Cloud UI.
- Create separate **dev** and **prod** environments and clusters.
- Design and create a **topic set**: `orders`, `payments`, `customers`.
- Validate **partitions** and **replication factors** using the UI.
- Generate **API keys** for a cluster.
- Run a **Python producer** that sends JSON order events.
- Run **Python consumers** in the same group and observe **consumer group rebalancing**.
- Introduce a **schema change** (new field) and see how the consumer handles it.


## Step 1 – Create a Confluent Cloud Account
1. Navigate to:
`https://confluent.cloud`
2. Click **Sign up**.
3. Create an account using:
    - Work email and password, or
    - Google/GitHub SSO.
4. Complete email verification if prompted.
5. After login, you should land on the **Confluent Cloud Home** or **Environments** page.

> If you are using a **shared workshop org**, simply log in with the credentials provided by the instructor and skip the signup flow.
---
## Step 2 – Create Initial Environment and Cluster
### 2.1 Create an environment

1. In the top navigation bar, click the current environment name (for example: `default`).
2. Click **+ Add environment**.
3. Enter a name, for example: `workshop`.
4. Click **Create**.
### 2.2 Create a basic Kafka cluster

1. Ensure **Environment** is set to `workshop`.
2. Click **+ Add cluster**.
3. Choose a **Cluster type**:
    - For this lab, use **Basic** or **Essentials**, depending on availability.
4. Select a **Cloud provider** (AWS/Azure/GCP) and **Region** close to you.
5. Enter a cluster name, for example: `workshop-cluster`.
6. Click **Launch cluster** (or **Create cluster**).
7. Wait for the cluster status to become **Running**.

---
## Step 3 – Create a Topic and Use the UI to Produce/Consume
### 3.1 Create a topic

1. In Confluent Cloud, verify:
    - **Environment**: `workshop`
    - **Cluster**: `workshop-cluster`
2. In the left sidebar, click **Topics**.
3. Click **+ Add topic**.
4. Configure the topic:
    - **Topic name**: `workshop-events`
    - **Partitions**: `3`
    - **Replication factor**: keep the default (typically `3` in multi-AZ clusters)
5. Click **Create with defaults**.
### 3.2 Produce messages using the UI

1. Open the **workshop-events** topic.
2. Go to the **Messages** (or **Data**) tab.
3. Click **Produce a new message**.
4. Provide:
    - **Key**: `key-1`
    - **Value**:
        ```json
        {"type": "workshop", "message": "hello from the Confluent Cloud UI"}
        ```
5. Click **Produce**.
### 3.3 Consume messages using the UI

1. In the same **Messages** tab, set the offset selector to **Consume from beginning** (or similar).
2. Click **Start** (if required).
3. Confirm that the message you produced appears in the results.

---
## Step 4 – Set Up Dev and Prod Environments and Clusters
### 4.1 Create the `dev` environment and cluster

1. Click the environment selector and choose **+ Add environment**.
2. Name the environment: `dev`.
3. Click **Create**.
4. With **Environment** set to `dev`, click **+ Add cluster**.
5. Configure the cluster:
    - Name: `dev-cluster`
    - Cluster type: **Basic** / **Essentials**
    - Cloud/region: choose any suitable combination
6. Click **Launch cluster** and wait for **Running** status.
### 4.2 Create the `prod` environment and cluster

1. Click the environment selector and choose **+ Add environment**.
2. Name the environment: `prod`.
3. Click **Create**.
4. With **Environment** set to `prod`, click **+ Add cluster**.
5. Configure the cluster:
    - Name: `prod-cluster`
    - Cluster type: can match `dev-cluster` for the lab
6. Click **Launch cluster** and wait for **Running** status.

At this point you should have:

- `dev` → `dev-cluster`
- `prod` → `prod-cluster`

---
## Step 5 – Design and Create the Topic Set

We will use a minimal e-commerce domain:

- `orders`
- `payments`
- `customers`
### 5.1 Topic design (simple)

For each topic:

- **Partitions**: `3`
- **Replication factor**: default for the cluster (commonly `3`)
### 5.2 Create topics in `dev`

1. Switch to:
    - **Environment**: `dev`
    - **Cluster**: `dev-cluster`
2. Go to **Topics** → **+ Add topic**.
3. Create the `orders` topic:
    - **Topic name**: `orders`
    - **Partitions**: `3`
    - **Replication factor**: default
    - Click **Create**.
4. Repeat the same process for:
    - `payments`
    - `customers`
### 5.3 Create topics in `prod`

1. Switch to:
    - **Environment**: `prod`
    - **Cluster**: `prod-cluster`
2. Create the same topics, with identical partition and replication settings:
    - `orders`
    - `payments`
    - `customers`

---
## Step 6 – Validate Partitions and Replication
### 6.1 Validate topics in `dev`

1. In `dev` → `dev-cluster`, open **Topics**.
2. Click the `orders` topic.
3. On the **Overview** / **Configuration** tab verify:
    - **Number of partitions**: `3`
    - **Replication factor**: default value (commonly `3`)
4. Navigate to the **Partitions** tab to inspect:
    - Partition IDs (`0`, `1`, `2`)
    - Leaders and replicas for each partition.
5. Repeat for `payments` and `customers`.
### 6.2 Validate topics in `prod`

Repeat the same validation steps for `prod` → `prod-cluster`.

---
## Step 7 – Generate API Keys

You will now create API keys for the `dev-cluster` so that local clients can connect.
### 7.1 Create an API key and secret

1. Switch to:
    - **Environment**: `dev`
    - **Cluster**: `dev-cluster`
2. In the left sidebar or cluster overview, click **API keys**.
3. Click **Create key** (or **+ Add key**).
4. When asked for scope, select:
    - **Granular access** for this Kafka cluster (recommended).
5. Click **Next** / **Create**.
6. Copy and store the following values safely:
    - **API Key** (for example: `RK7...`)
    - **API Secret** (only shown once)
7. (Optional) Add a description like: `dev-workshop-client`.
### 7.2 Retrieve the bootstrap server

1. Navigate to the **Cluster settings** or **Clients** page for `dev-cluster`.
2. Copy the **Bootstrap server** value, for example:
`pkc-xxxxx.ap-southeast-1.aws.confluent.cloud:9092`

You will use the following environment variables in the code samples:

- `BOOTSTRAP_SERVERS`
- `CLOUD_API_KEY`
- `CLOUD_API_SECRET`
- `TOPIC_ORDERS` (topic name)

---
## Step 8 – Run a JSON Producer (Orders)

This section uses **Python** and the `confluent-kafka` library.
### 8.1 Install the Python client

```bash
python -m venv venv
source venv/bin/activate      # Windows PowerShell: .\venv\Scripts\Activate.ps1

pip install confluent-kafka
```
### 8.2 Set environment variables

> Replace the placeholder values with your actual Confluent Cloud values.
**macOS/Linux:**

```bash
export BOOTSTRAP_SERVERS="pkc-xxxxx.region.provider.confluent.cloud:9092"
export CLOUD_API_KEY="YOUR_API_KEY"
export CLOUD_API_SECRET="YOUR_API_SECRET"
export TOPIC_ORDERS="orders"
```

**Windows PowerShell:**

```powershell
$env:BOOTSTRAP_SERVERS="pkc-xxxxx.region.provider.confluent.cloud:9092"
$env:CLOUD_API_KEY="YOUR_API_KEY"
$env:CLOUD_API_SECRET="YOUR_API_SECRET"
$env:TOPIC_ORDERS="orders"
```
### 8.3 Create `producer_orders.py`

```python
import os
import json
from confluent_kafka import Producer

conf = {
    "bootstrap.servers": os.environ["BOOTSTRAP_SERVERS"],
    "security.protocol": "SASL_SSL",
    "sasl.mechanisms": "PLAIN",
    "sasl.username": os.environ["CLOUD_API_KEY"],
    "sasl.password": os.environ["CLOUD_API_SECRET"],
}

producer = Producer(conf)
topic = os.environ.get("TOPIC_ORDERS", "orders")


def delivery_report(err, msg):
    if err is not None:
        print(f"Delivery failed for record {msg.key()}: {err}")
    else:
        print(
            f"Record produced to {msg.topic()} partition [{msg.partition()}] "
            f"@ offset {msg.offset()}"
        )


orders = [
    {"order_id": 1, "customer_id": 100, "amount": 49.99, "status": "NEW"},
    {"order_id": 2, "customer_id": 101, "amount": 19.99, "status": "NEW"},
    {"order_id": 3, "customer_id": 100, "amount": 5.99, "status": "NEW"},
]

for order in orders:
    key = str(order["order_id"])
    value = json.dumps(order)
    producer.produce(
        topic=topic,
        key=key,
        value=value,
        on_delivery=delivery_report,
    )

producer.flush()
print("All messages sent.")
```
### 8.4 Run the producer

```bash
python producer_orders.py
```

You should see delivery reports and offsets printed to the console.

---
## Step 9 – Run Consumers and Observe Consumer Group Behavior

You will now create a consumer application and run multiple instances to observe consumer group rebalancing.
### 9.1 Create `consumer_orders.py`

```python
import os
from confluent_kafka import Consumer, KafkaException

conf = {
    "bootstrap.servers": os.environ["BOOTSTRAP_SERVERS"],
    "security.protocol": "SASL_SSL",
    "sasl.mechanisms": "PLAIN",
    "sasl.username": os.environ["CLOUD_API_KEY"],
    "sasl.password": os.environ["CLOUD_API_SECRET"],
    "group.id": "orders-consumer-group",
    "auto.offset.reset": "earliest",
}

topic = os.environ.get("TOPIC_ORDERS", "orders")

consumer = Consumer(conf)
consumer.subscribe([topic])

print(f"Consuming from topic '{topic}' as group 'orders-consumer-group'...")

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            raise KafkaException(msg.error())
        print(
            f"Consumed record from partition {msg.partition()}, offset {msg.offset()}: "
            f"key={msg.key()}, value={msg.value().decode('utf-8')}"
        )
except KeyboardInterrupt:
    print("Stopping consumer...")
finally:
    consumer.close()
```
### 9.2 Run the first consumer

In **Terminal 1**:

```bash
python consumer_orders.py
```

You should see all previously produced messages.
### 9.3 Run a second consumer instance (same group)

In **Terminal 2** (same directory and environment variables):

```bash
python consumer_orders.py
```

Expected behavior:

- Kafka will **rebalance** the consumer group.
- Each instance will handle a subset of partitions.
- New messages are processed by exactly **one** consumer instance within the group.

To see this clearly, re-run:

```bash
python producer_orders.py
```

while both consumers are running.

---
## Step 10 – Introduce a Schema Change

Next, you will add a new field (`discount_code`) to the orders produced, and observe that the consumer continues to function.
### 10.1 Update the producer to include a new field

Edit `producer_orders.py` and replace the `orders` list with:

```python
orders = [
    {
        "order_id": 4,
        "customer_id": 102,
        "amount": 59.99,
        "status": "NEW",
        "discount_code": "NEWCUSTOMER10",
    },
    {
        "order_id": 5,
        "customer_id": 103,
        "amount": 29.99,
        "status": "NEW",
        "discount_code": "SUMMER2025",
    },
]
```

Run the producer again:

```bash
python producer_orders.py
```
### 10.2 Observe the consumer output

While the consumer(s) from Step 9 are running, you should see:

- New messages with the `discount_code` field included in the JSON payload.
- Consumers still functioning without errors because they treat the value as an opaque string.
### 10.3 (Optional) Parse JSON and handle missing fields

To make schema evolution more explicit, you can parse JSON in the consumer and use optional fields.

Update `consumer_orders.py`:

```python
import os
import json
from confluent_kafka import Consumer, KafkaException

conf = {
    "bootstrap.servers": os.environ["BOOTSTRAP_SERVERS"],
    "security.protocol": "SASL_SSL",
    "sasl.mechanisms": "PLAIN",
    "sasl.username": os.environ["CLOUD_API_KEY"],
    "sasl.password": os.environ["CLOUD_API_SECRET"],
    "group.id": "orders-consumer-group",
    "auto.offset.reset": "earliest",
}

topic = os.environ.get("TOPIC_ORDERS", "orders")

consumer = Consumer(conf)
consumer.subscribe([topic])

print(f"Consuming from topic '{topic}' as group 'orders-consumer-group'...")

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            raise KafkaException(msg.error())

        payload = msg.value().decode("utf-8")
        data = json.loads(payload)

        order_id = data["order_id"]
        amount = data["amount"]
        discount_code = data.get("discount_code", None)

        print(
            f"Order {order_id}: amount={amount}, discount_code={discount_code}, raw={payload}"
        )
except KeyboardInterrupt:
    print("Stopping consumer...")
finally:
    consumer.close()
```

Re-run the consumer(s):

```bash
python consumer_orders.py
```

You should see:

- Older messages: `discount_code=None`
- Newer messages: `discount_code` populated

This demonstrates a basic **forward-compatible** consumer pattern using optional fields.

---
## Step 11 – (Optional) Use the Confluent CLI

If you have the **Confluent CLI** configured for Confluent Cloud, you can also produce and consume via CLI.
### 11.1 Log in and select the environment and cluster

```bash
confluent login

confluent environment use dev
confluent kafka cluster list   # find your dev-cluster ID
confluent kafka cluster use <your-dev-cluster-id>
```
### 11.2 Produce messages

```bash
confluent kafka topic produce orders
```

Type a few lines (each line is a message value), then terminate input with:

- `Ctrl+D` (Linux/macOS), or
- `Ctrl+Z` followed by Enter (Windows PowerShell).
### 11.3 Consume messages

```bash
confluent kafka topic consume orders --from-beginning
```

You should see all messages from the beginning of the topic.

---

You now have a complete, GitHub-ready hands-on lab that walks through:

- Confluent Cloud account setup
- Dev/prod environments and clusters
- Topic design
- UI and CLI message production/consumption
- Programmatic producer/consumer
- Consumer groups and basic schema evolution behavior
