# Tracing for Beginners: Following a Request Through Your Django + React App

When our app feels slow, the first question is always the same: *where exactly is the time going?* A slow page could be blamed on the database, a third-party API, a bloated response payload, or slow React rendering. Without the right tool, we're just guessing.

That's the problem **tracing** solves.

---

## What Is Tracing?

Tracing is the practice of tracking a single request as it travels through our entire system — from the moment the user triggers an action in React, through the Django backend, into the database, and back again.

Think of it like a receipt for a request. Instead of just knowing "this endpoint took 900ms," tracing tells us:

- Django received the request → 2ms
- Auth middleware ran → 5ms
- Database query executed → 600ms ← *here's the problem*
- Serializer formatted the response → 3ms
- Response sent back → 1ms

Each step in that journey is called a **span**. A collection of spans for one request is called a **trace**. Together, they give us a visual timeline of exactly where time was spent and in what order.

---

## Core Concepts

### Spans

A span is the fundamental building block of a trace. Every unit of work we want to measure — a function call, a database query, an outgoing HTTP request — can be represented as a span. Each span records a name, a start time, an end time, a duration, and optional metadata like `user_id` or `rows_returned`. When we look at a trace, we're really just looking at a collection of spans laid out on a timeline.

### Traces

A trace is the complete picture — the full collection of all spans for one end-to-end request. Every span in a trace shares a common **Trace ID**, a unique identifier generated when the first span is created and carried through every subsequent operation. This is how everything connects. When Django receives a request and queries the database, both the request span and the query span carry the same Trace ID. Later, we can look up that ID in our tracing dashboard and see everything that happened during that request in one place.

### Context Propagation

Our React frontend and Django backend are separate processes. Context propagation is how a trace crosses that boundary. When React makes an HTTP request to Django, it attaches the current Trace ID as an HTTP header (following the W3C Trace Context standard). Django reads that header and continues the trace rather than starting a new one. The result is one unified trace that covers both frontend and backend.

---

## Distributed Tracing vs Local Profiling

These two approaches complement each other but answer different questions.

**Local profiling** is best when we already suspect a specific operation is slow. We zoom in on that one thing — a query, a function — and measure it precisely.

**Distributed tracing** is best when we don't yet know *where* the slowness lives. It shows us the entire journey of a request so we can identify which segment is the bottleneck. Once tracing tells us "it's the database query," we switch to profiling to understand why that query is slow.

Tracing gives us the map. Profiling gives us the magnifying glass.

---

## The Difference Between Logging, Profiling, and Tracing

These three often get conflated, so it's worth being precise:

**Logging** records *what happened* — discrete events like "user logged in" or "payment failed." Great for debugging errors, but doesn't tell us much about timing or sequence.

**Profiling** measures *how long a specific operation took*. It zooms in on one thing at a time.

**Tracing** shows us *the full journey of a request* — every operation, in order, with durations and relationships. It zooms out to reveal causality.

In a mature observability setup, we use all three together. Tracing tells us *where* the problem is. Profiling tells us *why* that operation is slow. Logging tells us *what was happening* at that moment in time.

---

## The Takeaway

Tracing is fundamentally about **visibility**. The core ideas to walk away with:

- Every operation is a **span** with a name, start time, end time, and duration
- A **trace** is the complete collection of spans for one request, tied together by a shared Trace ID
- **Context propagation** carries that Trace ID across service boundaries via HTTP headers, linking frontend and backend into one unified trace
- Tracing is most useful when we don't yet know where the problem is — it narrows things down so profiling can zoom in

Once we internalize these ideas, the specific tools — Jaeger, Zipkin, OpenTelemetry, Datadog — are just implementation details. The mental model is what matters.
