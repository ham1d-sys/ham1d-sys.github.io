---
layout: post
title: Catching a batch-failure gap in my ticket-triage pipeline
---

# Catching a batch-failure gap in my ticket-triage pipeline

Caught a gap in my ticket-triage pipeline in an AI code review, before it ever hit an outage. Logging it because the fix is boring but underrated.

## The gap

The pipeline classifies support tickets with an LLM, one API call per ticket. It ran end-to-end fine. But nothing distinguished "this ticket failed" from "the API is down or not responding, so every remaining call will fail too."

Without that, an outage means the batch keeps retrying ticket after ticket, each one doomed, until the whole queue has crawled through. You only find out at the end.

## The fix

A **fail-fast rule**, a stripped-down cousin of the *circuit breaker* pattern:

- Every ticket that completes, triaged or refused by the model, counts as a **success**.
- Only **transient API failures** (timeouts, connection errors) count against a threshold.
- If the last **N** calls in a row all fail that way, the batch **aborts** and logs why.

## One deliberate choice

A model refusal isn't a failure here. It's a client-side outcome, not a sign the API is down.

## Known limit

An abort currently drops the results for tickets already triaged.

---

Repo: [ticket-triage-pipeline](https://github.com/ham1d-sys/ticket-triage-pipeline)
