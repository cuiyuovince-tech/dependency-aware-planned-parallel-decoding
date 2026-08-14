# Dependency-Aware Planned Parallel Decoding

> A research proposal for accelerating long-form language generation through structured planning, dependency-aware scheduling, parallel generation, and global semantic verification.

## 中文摘要

长文本生成通常依赖逐 token 的自回归解码，延迟会随输出长度线性增长。本项目探索一种分层生成框架：模型先生成结构化计划，再预测段落或文本块之间的语义依赖图；真正独立的文本块可以同时生成，有依赖的文本块仅接收所需的语义摘要，而不必等待完整前文。最后由验证器检查事实、术语、覆盖率、重复与矛盾，并触发局部修订。

核心问题不是“能否并行”，而是：**如何可靠地识别哪些内容可以并行，以及如何在并行情况下保持全局一致性。**

## Motivation

Autoregressive language models generate one token after another. This produces strong local dependency modeling, but it creates an inference latency bottleneck for long outputs.

Many long-form answers have internal structure:

- a definition section;
- independent examples;
- a comparison section;
- a final synthesis.

A high-quality plan may resolve much of the semantic uncertainty before surface text is generated. If a planner can specify what each section must say, which concepts it may use, and which conclusions it must preserve, then some sections can be generated in parallel.

## Research Hypothesis

> A dependency-aware plan can reduce the sequential critical path of long-form generation while retaining higher coherence than fixed-outline parallel generation.

The system should outperform plain autoregressive decoding in end-to-end latency, and outperform naive plan-then-parallel generation in consistency and coverage.

## Proposed Architecture

```text
Task
  ↓
Structured planner
  - sections / spans
  - target length
  - semantic contracts
  - shared terminology
  - required claims
  - dependency graph
  ↓
Dependency-aware scheduler
  ↓
Parallel generators
  - autoregressive within each span, or
  - diffusion generation across spans
  ↓
Shared semantic state
  - verified definitions
  - confirmed claims
  - compact summaries
  ↓
Global verifier and local revision
  ↓
Final answer
```

### Semantic Contract

Each span receives more than a broad outline. Its contract may contain:

- local objective;
- required facts and claims;
- allowed terminology;
- forbidden duplication;
- target length and style;
- incoming semantic dependencies;
- outgoing summary for downstream spans.

This distinction matters: a section need not wait for earlier *surface text* if it already has the relevant *semantic state*.

## Main Novelty Target

Planning and parallel expansion are established ideas. The proposed contribution is not merely “make an outline, then write sections in parallel.”

The target contribution is:

1. **Learn or infer a dynamic dependency graph** instead of assuming all spans are independent.
2. **Use semantic interfaces** (verified summaries, definitions, claims) rather than full preceding text as inter-span dependencies.
3. **Schedule spans by dependency level**, maximizing safe parallelism.
4. **Verify global coherence** and repair only the spans responsible for a conflict.
5. Explore the same scheduler for both parallel autoregressive blocks and planned diffusion generation.

## Relation to Prior Work

- [Skeleton-of-Thought](https://arxiv.org/abs/2307.15337) creates an answer skeleton and expands its points in parallel.
- [Planned Diffusion](https://arxiv.org/abs/2510.18087) produces a short autoregressive plan, partitions output into independent spans, and generates those spans in parallel with diffusion.
- [Plan, Verify and Fill](https://arxiv.org/abs/2601.12247) uses planning and verification to improve structured parallel decoding for diffusion language models.

This project asks a different question: **can independence be predicted dynamically and represented as a semantic dependency graph, rather than being assumed from a fixed partition?**

## Experimental Plan

### Baselines

| Method | Purpose |
| --- | --- |
| Plain autoregressive decoding | Quality and latency baseline |
| Skeleton-of-Thought-style expansion | Fixed plan + parallel expansion baseline |
| Planned Diffusion | Plan + parallel diffusion baseline |
| Proposed method | Dependency graph + semantic state + verifier |

### Metrics

- End-to-end latency
- Tokens per second and GPU cost
- Dependency-graph accuracy
- Factual consistency
- Terminology consistency
- Coverage of planned claims
- Cross-section repetition
- Human or LLM-as-a-judge quality score
- Number of verifier-triggered repairs

### Evaluation Tasks

- Long-form instructional answers
- Multi-part technical explanations
- Reports with independent evidence sections
- Structured comparisons
- Coding explanations and design documents

## Initial Implementation Milestone

1. Use an existing 7B-8B instruction model.
2. Prompt it to emit JSON plans and dependency graphs.
3. Generate ready spans concurrently.
4. Pass compact semantic summaries to dependent spans.
5. Run a verifier for contradiction, repetition, and missing claims.
6. Compare latency and quality with sequential generation.

No foundation-model pretraining is required for the first prototype.

## Open Questions

- How detailed must a plan be before parallel generation becomes safe?
- Can a model accurately predict semantic independence?
- What is the best representation for a semantic interface?
- Does verification cost erase the latency gain?
- When should the scheduler choose autoregressive blocks versus diffusion blocks?
- How robust is the method for reasoning tasks whose later steps genuinely depend on earlier derivations?

## Status

This repository currently contains a research proposal and experimental roadmap. The next step is a small, reproducible prototype with public prompts, evaluation tasks, and latency measurements.

## License

No license is currently granted. All rights reserved unless a license is added later.
