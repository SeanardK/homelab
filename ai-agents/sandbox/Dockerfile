FROM nikolaik/python-nodejs:python3.11-nodejs20

# Base image sudah punya Python 3.11 + Node 20 — cukup buat mayoritas AI
# agent framework (CrewAI, AutoGen, LangGraph, Open Interpreter, dst yang
# basisnya Python, atau yang basisnya Node/TS). Tambahan di bawah cuma
# tooling umum yang hampir selalu dibutuhin pas install sesuatu yang baru.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git curl ca-certificates build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# uv — package manager Python yang jauh lebih cepat dari pip biasa,
# dipakai banyak proyek AI agent modern (termasuk Hermes Agent).
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:${PATH}"

WORKDIR /workspace
CMD ["sleep", "infinity"]
