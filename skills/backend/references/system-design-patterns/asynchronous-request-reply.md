# Asynchronous Request Reply

## Asynchronous Request Reply Standards

- Use this pattern when an operation takes too long for a synchronous HTTP request
- The client must receive an immediate acknowledgment with a way to check the result later
- The response must include a status URL or correlation ID for polling

---

## Instructions

### Step 1 - Identification

> Understand whether async request-reply is needed

- Identify whether the operation takes longer than a few seconds
- Identify whether the operation involves external systems, heavy computation, or batch processing
- Identify whether the client can tolerate eventual consistency

---

### Step 2 - Evaluate

> Determine if async request-reply is the right fit

- Best for: Long-running operations (report generation, video processing, bulk imports)
- Best for: Operations that involve calling external services with unpredictable latency
- NOT for: Operations that complete in under a second
- NOT for: Real-time requirements where eventual consistency is unacceptable

---

### Step 3 - Output

> Provide the async pattern design

- Define the request flow (accept → acknowledge → process → notify)
- Define the status check mechanism (polling endpoint, webhook, or both)
- Define the response shapes for each stage

---

## Edge Cases

### 1. Polling vs Webhook

> Choosing between client polling and server push

**Rule of Thumb:**

- Polling is simpler to implement; webhooks are more efficient for long operations
- Support both when possible

**Practical Trade-off:**

- Polling creates unnecessary load if the operation is slow
- Webhooks require the client to expose an endpoint

---

## Bad Example

### 1. Synchronous Endpoint for Long-Running Task

```typescript
app.post("/reports/generate", async (req, res) => {
  const report = await generateLargeReport(req.body); // takes 5 minutes
  res.json(report); // client timeout
});
```

---

## Good Example

### 1. Async Request Reply With Polling

```typescript
// Accept the request
app.post("/reports", async (req, res) => {
  const jobId = await queue.enqueue("generate-report", req.body);
  res.status(202).json({
    jobId,
    status: "pending",
    statusUrl: `/reports/${jobId}/status`,
  });
});

// Check status
app.get("/reports/:jobId/status", async (req, res) => {
  const job = await jobStore.get(req.params.jobId);
  if (job.status === "completed") {
    return res.json({ status: "completed", downloadUrl: `/reports/${job.id}/download` });
  }
  res.json({ status: job.status, progress: job.progress });
});
```
