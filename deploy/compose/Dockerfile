ARG BASE_IMG=nvcr.io/nvidia/cuda
ARG BASE_IMG_TAG=13.1.1-runtime-ubuntu24.04

FROM ${BASE_IMG}:${BASE_IMG_TAG} AS base

# Update the OS
RUN apt update && \
    apt upgrade -y && \
    apt install -y build-essential wget \
    && apt clean

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Create a workspace directory
# Not sure why, but we can't install vllm at the same time as the others, I suspect it's a function of the
# --extra-index-url flag
RUN mkdir -p /workspace && \
    cd /workspace && \
    uv venv -p 3.13 --seed .venv && \
    uv pip install vllm==0.17.1 && \
    uv pip install --extra-index-url https://download.pytorch.org/whl/cu128 -U \
        torch==2.10.0 \
        flashinfer-python==0.6.4 \
        flashinfer-cubin==0.6.4 \
        'nvidia-cutlass-dsl>=4.4.0.dev1' 

# Set the working directory
WORKDIR /workspace

ENV XDG_CACHE_HOME=/workspace/.cache
ENV HF_HOME=/workspace/.cache/huggingface
RUN mkdir -p $XDG_CACHE_HOME $HF_HOME

COPY startup.sh /workspace/startup.sh
RUN wget "https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-BF16/resolve/main/super_v3_reasoning_parser.py"

RUN chmod +x /workspace/startup.sh
CMD ["/workspace/startup.sh"]
