# Inference software stack selection

Select a compatible, supportable path for the specific model and workload. There is no universal
engine winner. Model architecture support, accelerator, numerical format, batching, cache,
parallelism, multimodality, API contract, and operational maturity all affect the result. Verify
current feature support in official documentation and benchmark the exact versions you plan to
deploy.

## Think in layers

1. **Model artifact and format:** checkpoint layout, tokenizer/processor, configuration, weights
   and quantization metadata. Confirm the artifact is trusted, complete, licensed for the use,
   and supported by the loader. Verify provenance and integrity (checksums or signatures where
   available); do not execute untrusted custom model-loading code.
2. **Framework and model library:** model implementation, preprocessing, custom operators, and
   generation behavior. Development frameworks can be flexible; inference libraries may add
   optimized kernels or graph compilation.
3. **Kernel and accelerator runtime:** device kernels, compiler, driver, collective
   communication, and hardware-specific libraries. Version compatibility and kernel coverage can
   determine whether a nominal feature is usable.
4. **Inference engine:** request scheduling, continuous batching, KV allocation, sampling,
   streaming, model parallelism, multimodal handling, and metrics.
5. **Serving and orchestration:** API/gateway, authentication, routing, queueing, autoscaling,
   deployment, and observability. Keep model execution and fleet management responsibilities
   explicit.

## Choose by compatibility and workload

Create a short capability matrix for each candidate. Record the exact model revision,
artifact/format, accelerator and driver, engine/runtime version, context and batch limits,
precision, needed features, API behavior, and known unsupported paths. A feature documented for
an engine may not be supported for every model, accelerator, or combination of features.

| Workload need | Questions for the candidate path |
| --- | --- |
| Flexible experimentation or custom model code | Can the framework/library load the architecture and processors without fragile patches? What compilation or eager path is used in serving? |
| Online autoregressive LLM | How are continuous batching, KV allocation, prefix reuse, streaming, cancellation, and tail latency handled? |
| High-throughput/offline generation | What batch shapes and scheduling policies are supported? Does batching preserve the required sampling and ordering semantics? |
| Constrained accelerator or portable deployment | Which operators, formats, hardware targets, and runtime versions are supported? What accuracy/performance changes occur after conversion? |
| Diffusion or vision pipeline | Are the model components, scheduler, image processor, attention kernels, and output decoder supported end to end? |
| Distributed serving | Which parallel strategies, collective backends, topology constraints, KV-transfer paths, and failure behaviors exist? |
| Production API | Are protocol, streaming, health/readiness, metrics, limits, and graceful shutdown behavior compatible with the product? |

Examples of ecosystem options to evaluate include PyTorch, ONNX Runtime, TensorRT, Hugging Face
Transformers and Diffusers, vLLM, SGLang, TensorRT-LLM, and NVIDIA Dynamo. Treat names as a
starting search list, not an endorsement or feature comparison. Frameworks/libraries, optimized
runtimes, request-serving engines, and distributed orchestration have different roles and may be
composed rather than directly substituted.

## CUDA and kernel path

You usually do not need custom CUDA kernels to serve a model. First establish that the
bottleneck is inside a kernel or data movement path. Then check whether an existing
engine/kernel covers the operation and shape. Kernel selection depends on datatype, dimensions,
alignment, device generation, and launch overhead. Fusion can reduce intermediate memory traffic
and launches, but custom code brings correctness, portability, compilation, maintenance, and
numerical-validation costs. Compare against the supported baseline with end-to-end load tests.

## Fair engine comparison

Hold constant the model and revision, tokenizer/processor, hardware/topology, precision,
input/output distributions, sampling settings, load generator, warm/cold conditions, and
service-level objectives. Verify output behavior and quality, not just speed. Measure
client-observed latency, TTFT/ITL where relevant, throughput and goodput, tail latency, memory,
errors, startup, and cost. Include integration effort and feature gaps in the selection
decision.

For performance-tooling selection, use the engine's current official benchmark instructions and
a system profiler appropriate to the accelerator. A profiler run diagnoses; it does not replace
a representative service benchmark.

## Official documentation starting points

Check current versions and model support directly before choosing or upgrading:

- [PyTorch documentation](https://docs.pytorch.org/docs/)
- [ONNX Runtime documentation](https://onnxruntime.ai/docs/)
- [TensorRT documentation](https://docs.nvidia.com/deeplearning/tensorrt/latest/)
- [Hugging Face Transformers documentation](https://huggingface.co/docs/transformers/)
- [Hugging Face Diffusers documentation](https://huggingface.co/docs/diffusers/)
- [vLLM documentation](https://docs.vllm.ai/)
- [SGLang documentation](https://docs.sglang.ai/)
- [TensorRT-LLM documentation](https://nvidia.github.io/TensorRT-LLM/)
- [NVIDIA Dynamo documentation](https://docs.nvidia.com/dynamo/)

These links are discovery points; pin versioned documentation when writing deployment
instructions. Validate downloads and dependencies against official sources, pin artifacts and
versions, and record the tested matrix in the deployment repo.
