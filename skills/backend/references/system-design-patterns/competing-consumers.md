# Competing Consumers

## Competing Consumers Standards

- Multiple consumers read from the same queue, but each message is processed by exactly one consumer
- Consumers must be idempotent — messages may be delivered more than once
- The system must handle consumer failures gracefully (message re-delivery)

---

## Instructions

### Step 1 - Identification

> Understand whether competing consumers are appropriate

- Identify whether there is a high volume of messages that need parallel processing
- Identify whether processing a single message is independent of other messages
- Identify whether throughput is a bottleneck

---

### Step 2 - Evaluate

> Determine if Competing Consumers is the right fit

- Best for: High-throughput task queues, background job processing, event processing at scale
- Best for: When horizontal scaling of consumers is needed
- NOT for: When messages must be processed in strict order
- NOT for: When a single consumer is sufficient for the throughput

---

### Step 3 - Output

> Provide the competing consumers design

- Define the message source (queue, topic)
- Define the consumer scaling strategy (auto-scale based on queue depth)
- Define the message acknowledgment mechanism
- Define the dead-letter handling for poison messages

---

## Edge Cases

### 1. Message Ordering

> Competing consumers process messages in parallel, breaking order

**Rule of Thumb:**

- If ordering matters within a subset, use partitioned queues (e.g., by entity ID)
- Messages for the same entity go to the same partition/consumer

**Practical Trade-off:**

- Partitioning preserves order per entity but limits parallelism within that partition

---

### 2. Poison Messages

> A message that consistently causes consumer failure

**Rule of Thumb:**

- Implement a retry limit with dead-letter queue
- After N retries, move the message to a dead-letter queue for investigation
- Never block the main queue with poison messages

**Practical Trade-off:**

- Some messages may be lost to the dead-letter queue
- Better to lose one message than block all processing

---

## Bad Example

### 1. Single Consumer for High-Volume Queue

```typescript
// One consumer processing 10,000 messages/minute
queue.subscribe("orders", async (message) => {
  await processOrder(message); // bottleneck — can only process 1,000/min
});
```

---

## Good Example

### 1. Competing Consumers With Dead Letter Queue

```typescript
const consumerPool = createConsumerPool({
  queue: "orders",
  concurrency: 10,
  maxRetries: 3,
  deadLetterQueue: "orders-dlq",
});

consumerPool.subscribe(async (message) => {
  await processOrder(message);
  message.ack();
});
```

### 2. Partitioned for Ordering

```typescript
// Messages for the same order ID always go to the same partition
const producer = createProducer({
  queue: "orders",
  partitionKey: (message) => message.orderId,
});
```
