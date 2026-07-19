---
title: "Algorithmic Trading Bot"
description: "A live algo-trading bot for Indian equity derivatives — execution, risk and market-data engineering in Go."
summary: "A live algorithmic trading bot for Indian equity derivatives. Low-latency data ingestion, a strict pre-trade risk gate, and resilient order routing — built in Go."
weight: 10
---

My flagship project: a live algorithmic trading bot for Indian equity
derivatives. The interesting engineering isn't the strategy — it's the
low-latency data ingestion, the pre-trade risk gate, and the order routing that
has to stay correct when the market moves against you.

The posts below document how each piece is designed and built.

{{< alert "circle-info" >}}
**The source is private for now.** This bot runs live against real capital, so the
repository stays closed while it's in production. Happy to walk through specific parts
of the design — feel free to [reach out](/about/).
{{< /alert >}}
