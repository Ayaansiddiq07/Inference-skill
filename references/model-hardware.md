# Model and hardware fundamentals

Use this reference to turn a model and traffic description into a hardware hypothesis. It is a
compact mental model for inference decisions, not a model-architecture textbook. Validate
quantitative claims with the actual model, engine, hardware, and runtime.

## Connect the product to execution

Start with the product's task and quality bar. An AI-native workflow may contain retrieval, tool
calls, multiple model invocations, or human review; optimize completed task quality, delay, and
cost rather than one model call in isolation. Separate interactive requests from offline jobs
when their latency and batching objectives conflict. Consumer workloads often have spiky
arrivals and strong first-token expectations; enterprise workloads may add tenancy, audit,
data-residency, and predictable-capacity constraints. These are workload questions to measure,
not assumptions about all products in either category.

Select models by representative task evaluation. Public leaderboards are candidate filters, not
proof of fit. Include difficult cases, regressions, output validity, and human/domain review
where automated metrics miss product quality. Fine-tuning can address a repeatable capability or
behavior gap; distillation can reduce serving cost after a capable teacher and acceptance
criteria exist. Compare lifecycle cost, data and evaluation requirements, update cadence, and
maintenance against prompting, retrieval, or a different model.

## Model execution: enough to reason about cost

Neural network inference is a sequence of tensor operations. Dense layers perform matrix
multiplications; elementwise activations and normalization add work and memory traffic. Fusing
adjacent operations can avoid writing intermediate tensors to memory, but only when a supported
kernel preserves needed numerical behavior and improves the measured path.

A decoder-only transformer alternates attention and feed-forward computation. During
**prefill**, the model processes the input context and builds attention state. During
**decode**, it generates output sequentially while reading model weights and updating/reading
key-value (KV) state. Prefill and decode have different shapes and bottlenecks; a single average
tokens-per-second number hides that distinction. Longer contexts increase attention work and KV
storage, while concurrency multiplies live state.

Mixture-of-experts (MoE) models route each token to a subset of expert networks. Sparse active
computation does not make the full model free: all expert weights may need to reside across
devices, routing can be imbalanced, and expert-parallel execution adds communication. Measure
tokens per expert, imbalance, all-to-all traffic, and effective goodput.

Image and video generation commonly perform repeated denoising or refinement over a latent
representation. Cost depends on architecture, resolution, latent dimensions, steps, attention,
and decoding; fewer steps or smaller resolution can change quality. Video adds temporal context
and potentially large activation and communication costs. Measure the actual pipeline rather
than extrapolating from text-token throughput.

## Build a bottleneck hypothesis

Use the roofline intuition: operations per byte (arithmetic intensity) helps reason whether a
workload is compute-limited or limited by moving data. Compare the work's arithmetic intensity
with the accelerator's attainable compute-to-memory-bandwidth ratio. This is a hypothesis aid,
not a substitute for profiling: kernels, batch shape, cache reuse, launch overhead, sparsity,
and communication change realized behavior.

- **Compute pressure:** high arithmetic work per transferred byte; often appears in large-batch
  or long prefill, image/video denoising, or heavy encoders. Check achieved compute, kernel
  occupancy, and matrix shape efficiency.
- **Memory-bandwidth pressure:** repeated weight or KV reads with relatively little work per
  byte; commonly affects autoregressive decode. Check effective bandwidth, batch composition, KV
  traffic, and cache behavior.
- **Capacity pressure:** weights, activations, KV state, workspaces, and fragmentation exceed
  usable memory or leave too little concurrency headroom. Track peak and steady-state
  allocation, not just checkpoint size.
- **Communication pressure:** tensor/expert parallelism or multi-node execution spends time
  transferring activations, tokens, or KV state. Check topology, link utilization,
  synchronization, and the payload-size/compute ratio.
- **Host or pipeline pressure:** preprocessing, tokenization, orchestration, transfer,
  compilation, or serialization dominates while accelerators wait. Time stages end to end.

For LLMs, separately vary prompt length, generated length, and concurrency. For diffusion, vary
resolution, batch, and steps. For audio/video, vary duration, chunking, and frame strategy. A
change that improves one shape may hurt another.

## Hardware fit and comparison

Estimate whether the model can fit with runtime state and headroom. A first-order weight
estimate is parameter count times bytes per stored parameter; it omits scale metadata,
unquantized layers, allocator overhead, activations, KV cache, and engine workspace. Estimate KV
demand from layers, KV heads, head dimension, sequence length, cache precision, and active
sequences using the selected architecture's actual layout. Validate estimates against peak
memory under the intended load.

Compare accelerators on the dimensions that match the bottleneck:

- Supported numerical formats and measured compute for the actual kernels.
- Memory capacity, bandwidth, and cache behavior for weights, activations, and KV state.
- Device-to-device and node-to-node interconnect bandwidth/latency for parallel execution.
- Host CPU, memory, storage, networking, power, and software ecosystem.
- Availability, reliability, region, price, and operational fit.

Peak FLOPS and device price alone do not predict cost per useful request. Compare a complete
serving instance and software path at the same workload and SLO. Multi-GPU can enable model fit
or lower latency, but incurs communication and may reduce efficiency for small workloads. GPU
partitioning such as multi-instance GPU can isolate capacity, while reducing the resources
available to a replica; benchmark partition size and interference. Non-NVIDIA datacenter
accelerators, desktop GPUs, and mobile NPUs have different model, kernel, memory, and ecosystem
support. Use the same evaluation and acceptance envelope across options.

Local, desktop, and mobile inference can reduce network dependence, improve offline use, or keep
data on-device. Decide using device memory and sustained compute, thermal and battery limits,
supported operators/formats, model-update path, and quality/latency targets. Include fallback and
synchronization behavior when work moves between device and server. A model that fits at load
may still exceed runtime memory under the target context or concurrent workload.

Hardware product names, generations, specifications, availability, and instance prices change.
Check vendor datasheets and cloud-provider capacity/pricing before procurement or architecture
commitments. Do not encode a generation ranking in a durable skill.

## Decision sequence

1. Define task quality, workload distributions, product deadline, and constraints.
2. Evaluate candidate models and establish a task-level quality floor.
3. Profile representative prefill/decode or modality-specific stages to identify the limiting
   resource.
4. Estimate capacity including runtime state and concurrency headroom; validate on target
   hardware.
5. Compare the smallest set of compatible hardware/configurations against quality, goodput,
   reliability, and total cost.
6. Re-evaluate after engine, precision, parallelism, or traffic changes; these change the
   bottleneck.
