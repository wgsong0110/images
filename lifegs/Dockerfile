FROM nvidia/cuda:12.1.0-runtime-ubuntu22.04

LABEL org.opencontainers.image.source=https://github.com/wgsong0110/images

ENV DEBIAN_FRONTEND=noninteractive

# Minimal system: ssh (vast manages sshd) + git/tmux/curl, conda. No rvfs, no supervisord, no idle-watchdog.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        openssh-server tmux curl git ca-certificates sudo && \
    curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        -o /usr/share/keyrings/githubcli-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
        > /etc/apt/sources.list.d/github-cli.list && \
    apt-get update && apt-get install -y --no-install-recommends gh && \
    mkdir -p /run/sshd && \
    echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config && \
    rm -rf /var/lib/apt/lists/*

# rclone
RUN curl -fsSL https://rclone.org/install.sh | bash

# Miniforge (conda/mamba)
RUN curl -fsSL https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh \
        -o /tmp/miniforge.sh && \
    bash /tmp/miniforge.sh -b -p /opt/conda && \
    rm /tmp/miniforge.sh
ENV PATH="/opt/conda/bin:$PATH"

# LifeGS conda env
COPY environment.yml /tmp/environment.yml
COPY requirements.txt /tmp/requirements.txt
RUN cd /tmp && /opt/conda/bin/mamba env create -f /tmp/environment.yml -n LifeGS --channel-priority flexible && \
    /opt/conda/bin/mamba clean -afy && \
    rm -f /tmp/environment.yml /tmp/requirements.txt

# git hooks: auto-download CUDA .so on pull/checkout
COPY githooks-post-merge /opt/githooks/post-merge
COPY githooks-post-checkout /opt/githooks/post-checkout
COPY githooks-pre-push /opt/githooks/pre-push
RUN chmod +x /opt/githooks/post-merge /opt/githooks/post-checkout /opt/githooks/pre-push

# wgsong user (UID 1000, matching host paths)
RUN useradd -m -u 1000 -s /bin/bash wgsong && \
    mkdir -p /etc/sudoers.d && \
    echo 'wgsong ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/wgsong && \
    mkdir -p /home/wgsong/.ssh && chmod 700 /home/wgsong/.ssh && \
    chown -R wgsong:wgsong /home/wgsong && \
    su -s /bin/bash wgsong -c 'export HOME=/home/wgsong && \
        git config --global --add safe.directory "*" && \
        git config --global credential.helper store && \
        git config --global core.hooksPath /opt/githooks && \
        /opt/conda/bin/conda init bash && \
        echo "conda activate LifeGS" >> /home/wgsong/.bashrc'
