FROM golang:1.15.2 as builder

ARG SINGULARITY_COMMITISH="v3.7.0"

WORKDIR $GOPATH/src/github.com/sylabs
RUN apt-get update && apt-get install -y \
    build-essential \
    uuid-dev \
    libgpgme-dev \
    squashfs-tools \
    libseccomp-dev \
    wget \
    pkg-config \
    git \
    cryptsetup-bin
RUN git clone https://github.com/sylabs/singularity.git \
    && cd singularity \
    && git checkout "$SINGULARITY_COMMITISH" \
    && ./mconfig -p /usr/local/singularity \
    && cd builddir \
    && make \
    && make install

FROM jupyter/datascience-notebook:399cbb986c6b

USER root
COPY --from=builder /usr/local/singularity /usr/local/singularity
ENV PATH="/usr/local/singularity/bin:$PATH"
RUN apt-get update && apt-get install -y ca-certificates libseccomp2 \
   squashfs-tools fuse s3fs netbase less \
    && rm -rf /tmp/*

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip" && unzip awscliv2.zip \
  && ./aws/install && rm -rf ./aws awscliv2.zip

RUN wget https://github.com/kahing/goofys/releases/download/v0.24.0/goofys \
  && chmod +x goofys && mv goofys /usr/local/bin

USER $NB_USER

RUN pip install --no-cache-dir itkwidgets==0.32.0 jupytext nilearn plotly pyvista
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager jupyter-matplotlib jupyterlab-datawidgets itkwidgets@0.32.0
RUN jupyter labextension install jupyterlab-plotly --no-build \
  && jupyter labextension install plotlywidget --no-build
RUN export NODE_OPTIONS=--max-old-space-size=4096 \
  && jupyter lab build

RUN conda install --yes datalad rclone && conda clean -tipsy
RUN wget https://raw.githubusercontent.com/DanielDent/git-annex-remote-rclone/v0.6/git-annex-remote-rclone \
  && chmod +x git-annex-remote-rclone && mv git-annex-remote-rclone /opt/conda/bin
RUN pip install --no-cache-dir nbgitpuller nibabel dipy pybids pynidm pydra-ml \
  reproschema nda-tools datalad-container datalad-neuroimaging pyontutils \
  datalad-osf
RUN pip install --no-cache-dir --upgrade https://github.com/datalad/datalad/archive/d062f6f39315d6b82c287796e1c4c70ff4df6317.zip
RUN pip install --no-cache-dir https://github.com/ReproNim/neurodocker/archive/f104933e7faf01ed3da7e3201cea469f9ddd87fe.zip

RUN mkdir -p /home/$NB_USER/.singularity/build
ENV SINGULARITY_TMPDIR=/home/$NB_USER/.singularity/build
