# CI Mirror Gaps: CPU-Only Checks on Fork

## What is Mirrored

This fork's CI mirror runs **CPU-only linting and pre-commit checks** on every PR and via manual trigger:

- **Pre-commit hooks**: yapf (Python formatting), isort (imports), autoflake (unused imports), clang-format (C++ formatting), and other style/lint checks defined in `.pre-commit-config.yaml`
- **Trigger**: Automatically on PR open/sync/reopen, manually via `/run-ci-mirror` comment on PRs, or via `workflow_dispatch`
- **Runtime**: GitHub-hosted `ubuntu-latest` (no GPU, no NVIDIA infra)

These checks validate code style and basic correctness without requiring GPU hardware or NVIDIA-internal Blossom infrastructure.

## Permanent Acknowledged Gaps (Not Mirrored)

The following are **GPU-intensive test suites that require NVIDIA Blossom CI or self-hosted runners** and are NOT mirrored on this fork:

1. **GPU Inference Tests (L0/L1)** (`tests/integration/defs/`)
   - Requires GPU hardware (H100, A100, L40S, etc.)
   - Models and weights from `LLM_MODELS_ROOT`
   - End-to-end inference validation
   - Staged on NVIDIA Blossom CI

2. **Dynamic KV Cache & Memory Tests**
   - GPU memory allocation and cache management validation
   - Requires actual GPU compute

3. **Distributed Inference (TP/PP)**
   - Multi-GPU parallelism tests
   - Requires multiple GPUs and MPI/NCCL setup

4. **Quantization Tests**
   - INT8, INT4, FP8, NVFP4 kernel validation
   - Requires GPU execution

5. **Model-Specific Conversion Tests**
   - Weight conversion and model loading from HuggingFace
   - Model-specific forward passes
   - Requires weights and GPU

6. **Blossom CI Bot Commands**
   - `/bot run` triggers NVIDIA-internal CI pipeline
   - Not available on fork CI
   - User must open PR on upstream and trigger via comment for full validation

## Why This Gap Exists

- **Fork CI resources**: Fork runs on GitHub-hosted runners (ubuntu-latest) — no GPU access
- **Intentional design**: This mirror validates *style and static checks only*
- **Upstream workflow**: Full validation happens on the upstream repo's Blossom CI after PR is opened

## Using This Mirror

1. Open a PR to **this fork** (branch → main on `CodersAcademy006/TensorRT-LLM`)
2. CI mirror runs automatically → lint/style checks only
3. Once passing, open PR to **upstream NVIDIA/TensorRT-LLM**
4. Upstream CI runs full GPU inference suite + Blossom stages
5. Reference CI mirror runs for quick dev-loop turnaround before upstream submission

## See Also

- `.github/workflows/fork-ci-mirror-fast.yml` — the workflow definition
- `.pre-commit-config.yaml` — lint/style hook config (shared with upstream)
- Upstream `pr-check.yml` and `l0-test.yml` for GPU-heavy tests (not mirrored)
