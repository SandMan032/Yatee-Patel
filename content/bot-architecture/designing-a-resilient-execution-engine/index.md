---
title: "Designing a Resilient Execution Engine"
date: 2026-07-18
draft: false
description: "A tour of the moving parts behind a low-latency order-execution engine — and how each layer fails gracefully."
summary: "How the execution engine is structured into ingestion, strategy, risk and order-routing layers — and the design choices that keep it alive when a broker socket drops mid-session."
tags: ["architecture", "go", "systems-design", "trading"]
categories: ["Bot Architecture"]
---

{{< alert icon="pencil" >}}
**Demo / placeholder post.** The prose here is illustrative filler used to show off
the blog's formatting capabilities — code blocks, diagrams, callouts and images.
Real deep-dives will replace it.
{{< /alert >}}

{{< lead >}}
An execution engine has one job: turn a decision into a filled order before the
market moves. Everything else is about surviving the moment that job gets hard.
{{< /lead >}}

## The shape of the system

At a high level the engine is four cooperating stages connected by bounded
channels. Market data flows in from the left; orders flow out to the broker on
the right. Each stage is independently testable and can back-pressure the one
before it.

{{< mermaid >}}
flowchart LR
    A[Market Data Feed] -->|ticks| B(Ingestion &amp; Normalisation)
    B -->|events| C{Strategy Core}
    C -->|intents| D[Risk Gate]
    D -->|approved orders| E[Order Router]
    E -->|FIX / REST| F((Broker))
    F -.fills / rejects.-> G[Reconciler]
    G -.position updates.-> C
{{< /mermaid >}}

The dashed edges are the ones people forget. A fill is not confirmed until the
**reconciler** has seen it — the strategy core never assumes an order worked just
because it was sent.

## Bounded channels, not unbounded ambition

Every hop uses a buffered channel with an explicit capacity. When a downstream
stage stalls, the buffer fills, and the upstream stage learns about it
immediately instead of silently queuing gigabytes of stale ticks.

```go
// A bounded pipe between two stages. If the consumer stalls, Send blocks
// and the producer can shed load deliberately instead of exploding memory.
type Pipe[T any] struct {
	ch chan T
}

func NewPipe[T any](capacity int) *Pipe[T] {
	return &Pipe[T]{ch: make(chan T, capacity)}
}

func (p *Pipe[T]) Send(ctx context.Context, v T) error {
	select {
	case p.ch <- v:
		return nil
	case <-ctx.Done():
		return ctx.Err() // caller decides: drop, retry, or halt
	}
}
```

{{< alert icon="lightbulb" cardColor="#1e3a5f" iconColor="#38bdd2" textColor="#e2e8f0" >}}
**Rule of thumb:** if a queue in a trading system is unbounded, it is a memory
leak with a countdown timer. Give every buffer a number and a policy for what
happens when it is full.
{{< /alert >}}

## Failure is a first-class input

The interesting engineering is in what happens when the broker socket drops
mid-session. The router treats a disconnect as just another event:

| Failure | Detection | Response |
| --- | --- | --- |
| Socket drop | Heartbeat gap > 2s | Freeze new orders, attempt reconnect |
| Order rejected | Broker NAK | Surface to strategy, do **not** retry blindly |
| Duplicate fill | Reconciler mismatch | Quarantine + alert, halt the strategy |
| Clock skew | NTP drift check | Widen timeout windows, log for review |

The guiding principle: **a stalled engine is safe; a confused engine is
expensive.** When in doubt, the system stops trading and pages a human.

### Reconnect, carefully

```go
func (r *Router) reconnect(ctx context.Context) error {
	backoff := 250 * time.Millisecond
	for attempt := 1; attempt <= maxAttempts; attempt++ {
		if err := r.dial(ctx); err == nil {
			r.resyncOpenOrders() // reconcile before trusting anything
			return nil
		}
		select {
		case <-time.After(backoff):
			backoff = min(backoff*2, 8*time.Second)
		case <-ctx.Done():
			return ctx.Err()
		}
	}
	return ErrBrokerUnreachable
}
```

## What the latency budget actually looks like

The chart below is a placeholder visual, but it stands in for the kind of
per-stage latency breakdown I track. The tail — not the median — is what decides
whether a strategy is viable.

![Per-stage latency budget across the execution pipeline](sample-latency.png "Placeholder: p50 vs p99 latency by pipeline stage")

## Where this goes next

Upcoming posts in this section will unpack each stage in detail:

- **Ingestion** — normalising a messy exchange feed into a clean event stream.
- **Risk Gate** — the pre-trade checks that can never be skipped.
- **Reconciler** — the boring component that saves you at 09:16 on expiry day.

{{< button href="/learning-log/" target="_self" >}}
Read the Learning Log →
{{< /button >}}
