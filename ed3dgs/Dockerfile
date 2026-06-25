FROM ghcr.io/wgsong0110/run:latest
USER root
RUN /opt/conda/bin/conda create -n ed3dgs --clone base-project -q && \
    /opt/conda/envs/ed3dgs/bin/pip install openmim --no-cache-dir -q && \
    /opt/conda/envs/ed3dgs/bin/mim install mmcv -q
COPY requirements.txt /tmp/requirements.txt
RUN /opt/conda/envs/ed3dgs/bin/pip install -r /tmp/requirements.txt --no-cache-dir -q && \
    /opt/conda/bin/mamba clean -afy && \
    rm -f /tmp/requirements.txt
USER wgsong
