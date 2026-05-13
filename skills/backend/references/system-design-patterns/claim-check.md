# Claim Check

## Claim Check Standards

- Use this pattern when messages or payloads are too large to send through a messaging system
- Store the payload externally and send only a reference (claim check) through the message queue
- The consumer retrieves the full payload using the claim check

---

## Instructions

### Step 1 - Identification

> Understand whether claim check is needed

- Identify whether messages exceed the messaging system's size limits
- Identify whether large payloads are causing performance issues in the message broker
- Identify whether the full payload is always needed by the consumer

---

### Step 2 - Evaluate

> Determine if Claim Check is the right fit

- Best for: Large file processing, batch data transfers, video/image processing pipelines
- Best for: When the message broker has strict size limits
- NOT for: Small messages that fit comfortably within broker limits
- NOT for: When the consumer does not need the full payload

---

### Step 3 - Output

> Provide the claim check design

- Define the external storage for payloads (S3, Blob Storage, shared file system)
- Define the claim check schema (storage URL, payload metadata, size)
- Define the retrieval mechanism for the consumer

---

## Edge Cases

### 1. Payload Cleanup

> Stored payloads must be cleaned up after consumption

**Rule of Thumb:**

- Define a retention policy or cleanup mechanism after the consumer confirms processing
- Use TTL on the storage if the broker supports acknowledgment

**Practical Trade-off:**

- Manual cleanup adds operational overhead
- Storage costs accumulate without cleanup

---

## Bad Example

### 1. Sending 50MB Payload Through Message Queue

```typescript
// Sending a full report as the message body
await queue.publish("report-processed", {
  body: massiveReportBuffer, // 50MB — exceeds broker limits, blocks other messages
});
```

---

## Good Example

### 1. Claim Check With External Storage

```typescript
// Producer — store payload, send reference
async function publishReport(report: Buffer) {
  const storageKey = `reports/${uuid()}.json`;
  await storage.put(storageKey, report, { ttl: 86400 }); // 24h TTL

  await queue.publish("report-ready", {
    storageKey,
    size: report.length,
    generatedAt: new Date().toISOString(),
  });
}

// Consumer — retrieve payload using claim check
async function handleReportReady(message: ReportReadyEvent) {
  const report = await storage.get(message.storageKey);
  await processReport(report);
  await storage.delete(message.storageKey); // cleanup
}
```
