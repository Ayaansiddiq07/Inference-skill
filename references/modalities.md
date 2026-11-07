# Modality-specific inference considerations

Read the section that matches the workload. Use product-level latency and quality measures;
accelerator metrics alone do not describe the user experience.

## Language models

- Split prompt prefill from autoregressive decode. Report TTFT and ITL/TPOT separately, plus
  full response latency and output-token throughput.
- Track prompt/output/context distributions, concurrent sequences, KV-cache use, cache reuse,
  scheduling, and queue delay.
- For mixture-of-experts models, inspect expert routing, token imbalance, and all-to-all
  communication as well as active compute.
- Evaluate quantization, speculative decoding, prefix caching, batching, tensor/expert
  parallelism, and prefill/decode disaggregation against the same request distribution and
  sampling requirements.

## Vision-language and omni-modal models

- Measure preprocessing and modality encoding as well as decoder time. Include image
  count/resolution, video frame sampling, audio length, and fused sequence length.
- Validate that resolution, frame selection, crops, and audio chunking preserve task quality.
  These choices can change both encoder cost and language-model context cost.
- For video, measure decode, frame extraction, transfer, and model stages independently; avoid
  assuming frame count is the only cost driver.

## Embeddings and rerankers

- Optimize documents or pairs per second, batch latency, and end-to-end pipeline cost. Record
  sequence-length distributions and truncation behavior.
- Benchmark realistic batch sizes and padding; preprocessing, tokenization, and data movement
  may dominate small models.
- Verify vector quality and downstream retrieval/ranking outcomes when changing model,
  precision, truncation, or pooling behavior.

## Speech recognition and diarization

- For interactive/chunked recognition measure time to first partial/final result, real-time
  factor, chunk latency, end-to-end delay, and transcript quality.
- For long recordings measure total processing time, audio throughput, segmentation overhead,
  memory, and diarization quality.
- Chunk size trades responsiveness against context continuity and overhead; evaluate overlap and
  stitching behavior on representative audio.

## Text-to-speech and speech-to-speech

- Measure time to first audio, real-time factor, chunk cadence, end-to-end conversational delay,
  and intelligibility/naturalness.
- Include text normalization, tokenization, vocoding, transport, and playback buffering. Profile
  streaming stages to find stalls.
- Preserve speaker identity, pronunciation, prosody, and any consent or safety constraints when
  optimizing model/runtime or chunk size.

## Image generation

- Track latency per image and images per unit time alongside quality. Record resolution, batch
  size, step count, guidance settings, model, and scheduler.
- Determine whether cost is dominated by denoising steps, attention, memory movement, decoding,
  or host-side orchestration. Profile stages before kernel or precision changes.
- Fewer steps, lower precision, reduced resolution, or caching may alter visual quality;
  evaluate on the product's actual prompts and acceptance criteria.

## Video generation

- Track clip duration, frame rate, resolution, step count, latency per clip, frames per second,
  memory, and visual/temporal quality.
- Identify attention, temporal context, context parallelism, communication, and decode costs.
  Test representative clip lengths and concurrency.
- Evaluate temporal consistency and prompt adherence after changing precision, attention,
  parallelism, or frame strategy.

For any modality, include the client and data pipeline in the end-to-end path. The right metric
is the one that corresponds to the user's task deadline and quality expectation.
