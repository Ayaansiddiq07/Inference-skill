# Production inference operations

Use this reference when capacity, deployment, reliability, or cost is in scope. Start from
expected demand and a service objective, then confirm limits with load tests.

## Capacity and autoscaling

- Size for workload mix and target goodput at the latency objective, with explicit headroom for
  bursts, failures, and model-loading time.
- Concurrency and batch size are coupled. Tune queue depth, admission control, batching delay,
  fairness, and cancellation together; unbounded queues convert overload into extreme tail
  latency.
- Choose scaling signals that reflect demand and saturation: queue age/depth, in-flight work,
  token or modality throughput, accelerator utilization/memory, and latency. CPU utilization
  alone can miss a saturated accelerator service.
- Include scale-up delay, model/image download, initialization, warmup, and readiness in
  cold-start measurements. Decide whether scale-to-zero savings justify cold latency and
  reliability costs.
- Scale independent pipeline stages separately only when their resource needs and bottlenecks
  differ, and account for buffering and backpressure between them.

## Deployment and reliability

- Pin and record model artifacts, runtime, drivers, libraries, and serving configuration.
  Validate compatibility in a reproducible container or deployment image.
- Fetch model artifacts and executable dependencies from trusted sources; verify signatures or
  checksums when available, and review custom model code before enabling it.
- Require authentication and authorization at the serving boundary. Apply per-tenant request,
  concurrency, token, and resource limits so one caller cannot exhaust shared capacity.
- Test startup, readiness, liveness, graceful shutdown, request draining, cancellation, retries,
  timeouts, and overload behavior.
- Roll out gradually with model/runtime health and quality signals. Keep a tested rollback path
  and avoid sending traffic to a worker before the model is ready.
- Plan for accelerator capacity interruptions, regional constraints, provider limits, and
  fallback behavior. Do not assume nominal cloud capacity is always available.
- For multi-region or multi-cloud routing, measure user latency, capacity, data residency, and
  failover behavior. Verify contractual, privacy, and licensing requirements for the data and
  model.

## Observability

Correlate request-level traces with aggregate metrics. Separate queue, preprocessing, model
execution, postprocessing, and transport. Record at least:

- Request rate, concurrency, queue age/depth, rejection, timeout, and cancellation rate.
- End-to-end latency and component latency distributions. For LLMs include TTFT and ITL/TPOT;
  for other modalities include first-result/first-audio and stage-specific timings as
  applicable.
- Input/output tokens or modality units, batch size, throughput, goodput, and error rate.
- Accelerator utilization, memory, power/throttling where available, host CPU/RAM, network, and
  interconnect signals.
- Cache capacity, hit/miss/eviction, reuse, and locality metrics when caching is enabled.
- Cost per request or useful output, plus idle capacity and startup costs.

Avoid high-cardinality or sensitive labels. Do not log prompts, completions, audio, or images by
default; use approved privacy controls and retention if content telemetry is required.

## Cost model

Compare total cost for the required quality and goodput. Include accelerator and CPU time,
idle/headroom capacity, storage/network, model loading, orchestration, retries, and
engineering/operational burden. For a configuration:

`cost per useful unit = total serving cost over the measurement window / units meeting the
quality and latency contract`

Define the unit (successful request, output token, image, audio second, or another task unit)
and include failed or out-of-SLO work in the numerator. Compare like-for-like workload mixes,
regions, and utilization assumptions. Recheck current provider pricing and availability before a
purchase or architecture recommendation.

## Client and API path

- Measure from the client as well as inside the server; connection setup, TLS, retries,
  serialization, and geographic distance can dominate small or streaming requests.
- Use asynchronous jobs for long-running work when the product can tolerate polling or
  callbacks. Define idempotency, status, expiration, and cancellation behavior.
- For streaming, ensure proxies and clients flush chunks, propagate disconnects/cancellation,
  handle partial output and errors, and use timeouts suited to the service contract.
- Avoid automatic retries that amplify overload or duplicate expensive work. Bound retries, use
  backoff, and make repeated submissions safe where possible.
