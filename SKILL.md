---
name: inference-engineering
description: >-
  Make architecture, performance, and production decisions for serving generative AI models. Use
  when selecting models or hardware, diagnosing latency or throughput, optimizing LLM or
  multimodal inference, benchmarking, scaling GPU services, or estimating serving cost.
---

# Inference Engineering

Use this skill to turn an AI product's quality, latency, reliability, and cost requirements into
a measured inference system. It is an independent operational synthesis informed by Philip
Kiely's *Inference Engineering* (Baseten Books, 2026), and is not affiliated with or endorsed by
the author or publisher. Vendor features, model releases, hardware availability, and software
behavior change quickly; verify current details against official documentation before making
implementation decisions.

## Start with the workload

Before choosing an engine or optimization, establish:

- **Product contract:** required output quality, modalities, context/input sizes, response
  shape, streaming needs, and failure behavior.
- **Traffic shape:** request rate, concurrency, burstiness, arrival pattern, input/output length
  distributions, and cacheable prefix frequency.
- **Service objectives:** end-to-end P50/P95/P99 latency, time to first token (TTFT),
  inter-token latency (ITL), throughput, availability, and recovery targets.
- **Constraints:** budget, regions, data residency, privacy, license, hardware access, and
  operational capacity.
- **Baseline:** representative quality evaluation plus end-to-end performance and cost under
  realistic load.

Do not optimize a proxy metric in isolation. Define the unit of work and include client, queue,
network, preprocessing, model, and postprocessing time where applicable. For agentic flows,
measure the whole task including tool calls and retries.

## Operating workflow

1. **Characterize the workload.** Separate interactive online traffic from offline/batch work;
   identify latency-sensitive and throughput-oriented paths. Record real input/output
   distributions and concurrency.
2. **Choose candidate models.** Shortlist by task-specific quality, modality, context, serving
   constraints, license, and operational fit. Use public benchmarks only to narrow candidates;
   evaluate on representative product data and failure cases.
3. **Set budgets.** Translate end-to-end objectives into budgets for queueing, network,
   preprocessing, prefill, decode, and postprocessing. Include headroom for bursts and degraded
   conditions.
4. **Fit model to hardware.** Estimate weight memory, runtime/activation overhead, KV-cache
   demand, and usable headroom. Check measured memory bandwidth, compute, interconnect, and
   topology; a nominal accelerator spec is not a workload benchmark.
5. **Establish a reproducible baseline.** Fix model revision, engine/version, hardware,
   precision, configuration, prompt mix, output limits, and load shape. Warm up, run enough
   samples, and report percentiles and throughput at the same time.
6. **Locate the bottleneck.** Use end-to-end measurements first, then system and kernel
   profiling. Distinguish queueing, prefill, decode, memory capacity/bandwidth, compute,
   communication, and host overhead.
7. **Change one variable at a time.** Apply the least complex optimization that addresses the
   measured bottleneck. Recheck quality, tail latency, throughput, memory, and cost together.
8. **Validate production behavior.** Test overload, cold starts, scale events, failures,
   deploys, cancellation, streaming, and recovery. Instrument the same metrics used in the
   service objectives.

## Metrics and diagnosis

- **TTFT:** time from a clearly stated start event (for example, client send or server
  acceptance) to first generated token. Keep that boundary consistent when comparing runs; it is
  sensitive to queueing, prompt length, prefill, and startup effects.
- **ITL:** time between generated tokens; reflects decode responsiveness. Report per-request
  distributions, not only aggregate token rate.
- **End-to-end latency:** client-observed completion time; include network and application work.
- **Throughput:** completed requests or tokens per unit time at a stated workload and
  latency/quality envelope. State whether tokens/sec is per user or system-wide.
- **Percentiles:** report P50 plus tail objectives such as P95/P99; averages can conceal slow
  requests.
- **Goodput:** work completed while meeting the relevant latency and quality objectives. Use
  this to compare configurations that trade speed against quality or tail behavior.

Interpret symptoms as hypotheses, then verify with traces/profiles:

| Symptom | Likely areas to measure |
| --- | --- |
| High TTFT, normal ITL | Queue wait, long prompts, prefill compute, cold start, preprocessing |
| Normal TTFT, poor ITL | Decode bandwidth, batch contention, KV-cache pressure, kernel efficiency |
| Both worsen as concurrency rises | Queueing, memory capacity, batch policy, admission control, saturation |
| GPU utilization low, latency high | CPU/tokenization, data transfer, synchronization, network, scheduling |
| Throughput rises but tail latency violates SLO | Batching/concurrency policy, queue limits, workload isolation |

These are diagnostic leads, not conclusions. Confirm the active bottleneck before changing
configuration.

## Model and hardware decisions

- Prefer the smallest/least costly model that passes the product's quality evaluation with
  acceptable margin. Consider prompting, retrieval, fine-tuning, or distillation only when
  evaluation identifies a specific gap and the lifecycle cost is justified.
- Separate online and offline deployments when their latency, batching, or availability
  objectives conflict and measured traffic justifies the operational overhead.
- Fit the model and runtime state into accelerator memory with explicit headroom for KV cache,
  activations, fragmentation, and concurrency. Weight fit alone is insufficient.
- For autoregressive LLMs, prefill and decode stress hardware differently: prefill is commonly
  compute-heavy; decode often becomes memory-bandwidth-bound. Measure both for the actual
  sequence and batching mix.
- Multi-GPU choices trade memory/latency/throughput against communication and complexity. Tensor
  parallelism can reduce per-request latency for a split model; expert parallelism can serve
  sparse experts at scale. Validate interconnect and synchronization costs. Multi-node execution
  requires especially careful communication measurement.
- For image, video, audio, embedding, and vision-language workloads, use modality-specific
  pipelines and metrics. Include preprocessing, chunking, encoder/decoder, and postprocessing
  costs; do not apply LLM token metrics blindly.
- Check model and dataset licenses and relevant security, privacy, and regulatory requirements
  before deployment.

For the reasoning behind these choices, use [model and hardware
fundamentals](references/model-hardware.md). For framework, format, and serving-engine
compatibility, use [software stack selection](references/software-stack.md). These references
provide decision methods, not timeless rankings: verify support for the exact model,
accelerator, runtime version, and feature set.

## Optimization playbooks

Apply only after establishing a baseline and identifying a compatible bottleneck:

### Batching and scheduling

Batching can improve accelerator utilization and amortize overhead, while increasing queueing
and per-request latency. Tune concurrency, batch limits, wait windows, and admission control
against realistic arrivals. Protect interactive traffic from batch work when their objectives
conflict.

### Quantization

Treat quantization as a quality/performance spectrum, not a binary setting. Choose which
components and precision to change based on measured memory, bandwidth, compute, and quality
constraints. Evaluate representative task quality, edge cases, and regression sets after each
change; separately observe weights, activations, attention, and KV-cache effects where the
engine allows it. Roll back if the product quality floor or reliability target is missed.

### Speculative decoding

Use a draft/proposal mechanism only when its proposal cost and acceptance behavior produce a net
gain for the target workload. Measure acceptance rate, memory overhead, batch-size impact,
per-user ITL, aggregate throughput, and cost. Benefit can shrink or reverse at high concurrency
or with weak acceptance.

### KV and prefix caching

Reuse cached state when prefixes recur and correctness/isolation requirements permit it. Measure
hit rate, saved prefill work, storage/memory cost, eviction behavior, and routing locality.
Define cache keys and tenant/security boundaries so one request cannot reuse another tenant's
private context. Long contexts and high concurrency increase KV pressure; account for placement
and transfer costs.

### Parallelism and disaggregation

Introduce parallelism when model fit, latency, or throughput measurements justify communication
overhead. Disaggregate prefill and decode only when the workload is large and stable enough to
benefit from independent resource allocation and the system can manage KV transfer/routing.
Include network bandwidth, transfer latency, failure modes, and operational complexity in the
comparison.

### Kernel and runtime optimization

Profile before writing or selecting specialized kernels. Kernel fusion can reduce memory traffic
and launch overhead, but compatibility and numerical behavior matter. Compare
framework/runtime/engine candidates using the same model revision, hardware, prompt mix, output
lengths, load, and quality checks.

For detailed comparisons of quantization methods, speculative decoding families, cache
placement, long-context serving, and distributed execution, read [the optimization
playbook](references/optimization-playbook.md).

## Benchmarking and profiling

- Use production traces or a privacy-safe workload that preserves request sizes, arrival
  patterns, and cache characteristics. Synthetic tests are useful for controlled experiments,
  but state their limits.
- Record cold and warm behavior separately. Include warmup, adequate duration/sample count,
  repeated runs, variance, and hardware/software versions.
- Sweep load to find both saturation and the maximum goodput under SLOs. Avoid comparing a
  lightly loaded run with an overloaded one.
- Change one factor at a time for attribution; after tuning, run a combined end-to-end test
  because optimizations can interact.
- Pair system benchmarks with task-quality evaluation. Performance improvements that degrade
  outputs past the product's threshold are not wins.
- Benchmarking shows **what** is slow; profiling helps explain **why**. Use the least intrusive
  profiler that can isolate the active bottleneck and validate the proposed fix with a fresh
  benchmark.

## Production readiness

- **Capacity and autoscaling:** scale on leading indicators (queue depth, utilization, memory/KV
  pressure, arrival rate) as well as completed work. Account for provisioning and model-load
  time; keep enough capacity or degrade gracefully when cold starts violate objectives.
- **Queueing and overload:** bound queues and request sizes, set timeouts/cancellation, use
  admission control, and define explicit overload responses. Unbounded queueing converts
  saturation into tail-latency failure.
- **Reliability:** test health checks, retries with limits, idempotency where needed, instance
  loss, regional/provider failure, rollout, rollback, and capacity exhaustion. Retries can
  amplify overload.
- **Deployment:** pin model artifacts and dependencies, validate startup/readiness, canary
  changes, compare quality and service metrics, and retain a tested rollback path. Avoid
  exposing secrets in images, logs, or benchmark artifacts.
- **Observability:** correlate request traces with model, engine, hardware, and configuration
  versions. Track TTFT, ITL, end-to-end percentiles, queue time, throughput/goodput, errors,
  cache hit rate, utilization, memory, cold starts, and cost.
- **Cost:** compare cost per successful request or useful output at the required quality and
  SLO, including idle capacity, replication, networking, storage, and engineering/operations
  burden. Scale-to-zero can reduce idle spend but adds cold-start latency.
- **Security and compliance:** control model and data access, isolate tenants, protect prompts
  and outputs in logs, define retention, validate dependencies and artifacts, and meet
  applicable data residency and regulatory requirements.

For container reproducibility, capacity and routing, multi-region operations, client protocols,
and rollout practices, read [production operations](references/production.md). For measurement
by modality, read [modality-specific inference considerations](references/modalities.md).

## Reference routing

| Task | Read |
| --- | --- |
| Build a workload card, establish a baseline, or locate a bottleneck | This file, then [optimization playbook](references/optimization-playbook.md) |
| Understand model execution, accelerator fit, or hardware tradeoffs | [Model and hardware fundamentals](references/model-hardware.md) |
| Select a framework, model format, kernel path, or serving engine | [Software stack selection](references/software-stack.md) |
| Tune quantization, speculation, caches, parallelism, or disaggregation | [Optimization playbook](references/optimization-playbook.md) |
| Serve vision, embeddings, audio, image, or video workloads | [Modality-specific considerations](references/modalities.md) |
| Plan autoscaling, deployment, reliability, API/client behavior, and cost | [Production operations](references/production.md) |

## Response format for inference tasks

When advising or implementing, provide:

1. Workload assumptions and missing measurements.
2. The objective and baseline metrics.
3. The leading bottleneck hypothesis and how to prove it.
4. A ranked experiment plan with expected tradeoffs.
5. Quality, tail-latency, reliability, security, and cost acceptance criteria.
6. Current vendor/model/runtime facts with links to official sources when they affect the
   recommendation.

If the workload is underspecified, state bounded assumptions and provide a measurement plan
rather than presenting guessed settings as universal best practice.
