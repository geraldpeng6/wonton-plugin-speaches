# Speaches 官方镜像（ghcr.io/speaches-ai/speaches 的 *-cuda 标签）只发到 CUDA 12.6，
# RTX 50 系（Blackwell，sm_120）需要 CUDA 12.8 的运行库。本 Dockerfile 复刻上游
# speaches-ai/speaches Dockerfile@993994f7984bf3fe9655b267448328cf66fccb42，只改两处：
#   1. 基础镜像换成官方 nvidia/cuda:12.8.1-base-ubuntu24.04；
#   2. 源码从上游仓库按提交号取（上游用 COPY . .，要求整个仓库做构建上下文）。
# 不改上游任何代码和依赖锁（uv sync --frozen，依赖按上游 uv.lock 固定）。
FROM nvidia/cuda:12.8.1-base-ubuntu24.04
LABEL org.opencontainers.image.source="https://github.com/speaches-ai/speaches"
LABEL org.opencontainers.image.licenses="MIT"
# `ffmpeg` is installed because without it `gradio` won't work with mp3(possible others as well) files
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends ca-certificates curl ffmpeg
# "ubuntu" is the default user on ubuntu images with UID=1000. This user is used for two reasons:
#   1. It's generally a good practice to run containers as non-root users.
#   2. Docker Spaces on HuggingFace don't support running containers as root.
RUN useradd --create-home --shell /bin/bash --uid 1000 ubuntu || true
USER ubuntu
ENV HOME=/home/ubuntu \
    PATH=/home/ubuntu/.local/bin:$PATH \
    UV_LINK_MODE=copy \
    UV_CACHE_DIR=/home/ubuntu/.cache/uv \
    UV_PYTHON_CACHE_DIR=/home/ubuntu/.cache/uv/python
WORKDIR $HOME/speaches
# https://docs.astral.sh/uv/guides/installing-uv
COPY --chown=ubuntu --from=ghcr.io/astral-sh/uv:0.10 /uv /bin/uv
ARG SPEACHES_COMMIT=993994f7984bf3fe9655b267448328cf66fccb42
# 上游源码按提交号取，解到工作目录（提交在上游 main 上，官方镜像同一来源）。
RUN curl -fsSL "https://github.com/speaches-ai/speaches/archive/${SPEACHES_COMMIT}.tar.gz" \
    | tar -xz -C /tmp \
    && cp -a "/tmp/speaches-${SPEACHES_COMMIT}/." . \
    && rm -rf "/tmp/speaches-${SPEACHES_COMMIT}"
RUN --mount=type=cache,target=/home/ubuntu/.cache/uv,uid=1000,gid=1000 \
    uv sync --frozen --compile-bytecode --no-install-project --no-dev
RUN --mount=type=cache,target=/home/ubuntu/.cache/uv,uid=1000,gid=1000 \
    uv sync --frozen --compile-bytecode --no-dev
# Creating a directory for the cache to avoid a PermissionError when the cache volume is mounted.
RUN mkdir -p $HOME/.cache/huggingface/hub
# Register pip-installed CUDA library directories with ldconfig so that
# C++ libraries (e.g. ctranslate2) can find them via the system dynamic linker.
USER root
RUN find /home/ubuntu/speaches/.venv -maxdepth 7 -path "*/nvidia/*/lib" -type d \
    > /etc/ld.so.conf.d/venv-nvidia.conf && ldconfig
USER ubuntu
ENV UVICORN_HOST=0.0.0.0
ENV UVICORN_PORT=8000
ENV PATH="$HOME/speaches/.venv/bin:$PATH"
ENV DO_NOT_TRACK=1
ENV GRADIO_ANALYTICS_ENABLED="False"
ENV DISABLE_TELEMETRY=1
ENV HF_HUB_DISABLE_TELEMETRY=1
ENV PYANNOTE_METRICS_ENABLED=0
EXPOSE 8000
CMD ["uvicorn", "--factory", "speaches.main:create_app"]
