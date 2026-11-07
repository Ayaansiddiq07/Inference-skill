# Workload and optimization playbook

Use this reference when diagnosing or improving an inference deployment. Keep the evaluation
tied to one explicit service contract and baseline.

## Workload card

Record enough detail to reproduce the decision:

- Task, modality, model/revision, tokenizer or processor, and quality evaluation.
- Online interactive, streaming, or offline/batch workload; user-visible deadline and
  service-level objective.
- Input, output, and context-length distributions; for multimodal work include image resolution,
  frame count, audio duration, and chunking.
- Arrival rate and burstiness, concurrency, queueing policy, batch formation, and request
  cancellation behavior.
- Hardware, accelerator count/topology, memory, runtime/engine versions, precision, and all
  relevant launch/configuration options.
- Quality floor, latency percentiles, throughput or goodput target, error budget, and cost
  ceiling.

If the workload is unknown, use production traces or a representative synthetic distribution.
Uniform fixed-size requests can conceal the tail and batching behavior that determine real
capacity.

## Metrics that answer distinct questions

- **TTFT:** time from request arrival to first generated token; includes relevant queueing and
  prefill behavior when measured end to end.
- **ITL/TPOT:** time between output tokens or average decode time per output token. State the
  exact definition used by the serving engine.
- **End-to-end latency:** request arrival through completed response, including preprocessing,
  queueing, model, postprocessing, transport, and client overhead.
- **Throughput:** completed requests, input tokens, output tokens, images, audio seconds, or
  videos per unit time. Name the unit and whether it is aggregate or per accelerator.
- **Goodput:** work completed within both quality and latency requirements. Use it when
  comparing a high-throughput configuration with a slower but more reliable one.
- **Tail latency and errors:** p95/p99, timeouts, rejected/queued requests, and partial stream
  failures reveal overload that averages hide.
- **Quality:** task-appropriate evaluation (including human or domain-specific review where
  needed), not just generic perplexity or model size.

State whether measurements begin at the client, gateway, scheduler, or model worker. Report warm
and cold paths separately when cold starts are possible.

## Benchmark discipline

1. Pin model artifacts and every material software/hardware setting.
2. Warm up kernels, compilation, allocators, and model loading where applicable; report startup
   separately.
3. Use enough repetitions and requests to estimate the stated percentiles. Keep concurrency and
   input/output distributions representative.
4. Benchmark both single-request behavior and the target traffic mix at load. Avoid measuring
   with a closed-loop client that hides overload if production arrivals are open-loop.
5. Capture client-observed and component-level timings. Check clock state, GPU utilization,
   memory, throttling, queue depth, and errors.
6. Save raw results and configuration; compare against the same baseline. Use a profiler to
   explain changes, not to replace service-level measurement.

## Bottleneck map

| Signal | Likely constraint to investigate | Candidate directions (only after confirmation) |
| --- | --- | --- |
| Long prompt, slow first token | Prefill compute, attention/context work, queueing | Prompt/context design, efficient attention/runtime, prompt caching, batching policy |
| Slow token stream after first token | Decode serial work, weight bandwidth, KV cache, contention | Quantization, decode kernels, batching/scheduling, speculative decoding, cache capacity |
| Good isolated latency, poor loaded p95 | Queueing, batching, saturation, noisy neighbors | Admission control, queue limits, batching/concurrency tuning, autoscaling/capacity |
| GPU underused while requests are slow | CPU/preprocessing, host-device transfer, network, synchronization | Profile stages and overlap, optimize data path, reduce dispatch/communication overhead |
| Memory exhausted or context throughput collapses | Weights, KV cache, activations, fragmentation | Lower memory footprint, cache policy, context limits, supported quantization, parallel placement |
| Multi-GPU run is slower than expected | Communication, topology, synchronization, small workload | Measure communication and placement; compare fewer GPUs or alternate parallel strategy |
| Startup dominates small/rare workloads | Image/model load, compilation, scale-from-zero | Keep warm capacity, reduce load path, preload/cache artifacts, choose cold-start policy |

These are hypotheses, not diagnoses. Validate them with instrumentation and a profile from the
live execution path.

## Optimization order

1. **Model/task fit:** compare models on the actual evaluation. A smaller or specialized model
   may meet the quality floor with lower cost and latency. Fine-tuning or distillation is a
   quality/model-development decision; evaluate lifecycle cost and maintainability.
2. **Serving engine and supported path:** compare compatible frameworks/engines and model
   formats. Check model operators, kernels, hardware, precision, and feature maturity in current
   official documentation.
3. **Batching and scheduling:** tune batch size, maximum wait, concurrency, fairness, and
   prefill/decode scheduling against latency and goodput. Larger batches often trade latency or
   memory for throughput.
4. **Quantization:** test the candidate format and quantization method end to end. Validate
   quality, kernel availability, memory footprint, speed, and numerical behavior.
5. **Caching:** measure reuse and hit rate. Consider prefix/KV reuse, cache placement,
   locality-aware routing, storage tier, invalidation, isolation, and memory contention.
6. **Speculative decoding:** test draft overhead and acceptance rate against the target output
   mix. Preserve sampling/quality semantics required by the product.
7. **Parallelism or disaggregation:** compare communication and scheduling overhead with the
   actual model size, context, and target. Tensor parallelism can help latency; expert
   parallelism can address MoE throughput; separate prefill/decode only when workload asymmetry
   compensates for transfer/orchestration cost.
8. **Hardware and capacity:** compare total cost for required goodput, including utilization,
   memory capacity/bandwidth, interconnect, availability, region, and headroom—not just peak
   FLOPS or hourly price.

For each change, keep the measured outcome, quality check, rollback path, and any new
operational burden. Do not stack several unexplained changes into one benchmark.

## Technique selection details

### Quantization

Quantization reduces numeric precision to lower weight or activation storage and, on supported
hardware/kernels, memory traffic or compute time. Do not assume lower-bit storage yields faster
execution: unsupported formats may dequantize inefficiently, lose optimized kernels, or reduce
batch efficiency.

- Distinguish weight-only quantization from quantized activations and KV cache. Each changes
  different memory and compute costs.
- Compare candidate formats (for example, reduced-precision floating point and integer formats)
  only where both the hardware and serving engine have a supported path. Record scale
  granularity and calibration/conversion method; these affect accuracy and runtime behavior.
- Check which layers remain at higher precision, metadata overhead, outlier handling, activation
  ranges, and whether the artifact format is directly loadable by the target engine.
- Evaluate task quality on representative and adversarial/regression examples. Track
  structured-output validity and downstream task success in addition to aggregate scores.
- Measure peak memory, concurrency, throughput, TTFT/ITL, tail latency, and cost. A memory
  reduction is valuable even without a latency gain if it safely enables more useful
  concurrency.

### Attention and image generation kernels

Attention optimizations aim to reduce intermediate materialization and high-bandwidth-memory
traffic, improve tiling, or use an implementation suited to the sequence and batch shape. Confirm
the model's attention variant, mask, precision, and engine support; compare long-context quality
and latency as well as short prompts. A faster attention kernel will not fix unrelated queueing
or host overhead.

For image generation, profile text encoding, denoising/refinement steps, attention, latent-to-image
decoding, and orchestration separately. Test supported fused kernels, compilation, and precision
changes against representative resolutions and batch sizes. Fewer-step or distilled models can
reduce serial work, but they need separate visual-quality and prompt-adherence evaluation; they
are model alternatives, not a free runtime switch.

### Speculative decoding

Speculation proposes multiple future tokens or branches so target-model verification can accept
useful work in fewer sequential decode steps. Candidate families include a separate draft model,
auxiliary prediction heads (such as multi-token heads), and prompt/ngram-based proposals or
lookahead. Their proposal cost, memory needs, compatibility, and acceptance behavior differ;
treat each as an independent experiment.

Record accepted tokens per verification step, proposal/verification time, extra memory, output
semantics, per-request ITL, aggregate goodput, and performance across concurrency levels. Check
support for the exact sampling settings and engine. Speculation helps only when saved target
decode work exceeds proposal, verification, and scheduling overhead; acceptance rate alone does
not establish a win.

### KV and prefix cache placement

KV state may live in accelerator memory, host memory, or a remote/distributed store depending on
engine design. Faster tiers are scarce; slower tiers add transfer, serialization, and routing
costs. Compare the time and capacity saved by reuse with lookup, transfer, eviction, and
contention costs.

- Track cache hit rate by workload and prefix length, avoided prefill time, bytes
  retained/transferred, eviction rate, and impact on active-sequence capacity.
- Use stable, correct cache keys that include model/tokenizer revision and all context-affecting
  inputs. Define tenant isolation and authorization boundaries; never reuse private context
  across principals.
- Cache-aware routing can reduce recomputation but may create hotspots or reduce load-balancing
  freedom. Measure locality benefit against skew, failover, and queueing.
- For long contexts, measure prompt processing time, KV growth, attention behavior, and
  concurrency collapse by context bucket. Apply context limits, chunking, retrieval, windowing,
  or cache compression only after validating task quality and semantics.

### Parallelism and disaggregation

Tensor parallelism shards layer computation and can reduce per-request latency or make weights
fit, at the cost of frequent synchronization. Pipeline parallelism splits layer ranges and can
increase aggregate throughput when stages stay balanced, while introducing bubbles and
activation transfer. Expert parallelism places MoE experts across devices and depends on routing
balance and all-to-all capacity. Data parallel replicas improve independent-request capacity but
duplicate model memory. Combine strategies only when measurements show a benefit.

For each strategy, measure communication time, utilization per device, synchronization,
batch/sequence shape, tail latency, failure behavior, and cost. Validate topology and
collective-library support. More accelerators do not guarantee faster requests.

Prefill/decode disaggregation assigns prompt processing and token generation to separately sized
pools. It is most plausible when phases have different resource profiles and enough sustained
traffic to amortize scheduling and KV transfer. Measure transfer bandwidth and latency, cache
reuse across pools, routing/queue delay, capacity imbalance, and recovery when a pool is
unavailable. Dynamic reallocation adds control-loop complexity; include stability and
observability in evaluation.
