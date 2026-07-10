```

alias cls='clear' ; alias dir='ls -la' ; alias more='less -iX'


https://repo.anaconda.com/archive/

==> Dockerfile.anaconda <==
FROM ubuntu:24.04

# Install basic dependencies
RUN apt-get update && apt-get install -y \
    bzip2 \
    ca-certificates \
    git \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender1 \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Install Anaconda
RUN wget -O /tmp/anaconda.sh https://repo.anaconda.com/archive/Anaconda3-2025.12-2-Linux-x86_64.sh && \
    bash /tmp/anaconda.sh -b -p /opt/anaconda && \
    rm /tmp/anaconda.sh

# Put conda in PATH
ENV PATH="/opt/anaconda/bin:${PATH}"

RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r

RUN conda init
RUN conda create --name my_env python=3.10 pandas numpy -y
RUN conda activate my_env

COPY Dockerfile.anaconda /

CMD ["/bin/bash"]


# docker build -t my-anaconda-image -f Dockerfile.anaconda .

# docker run -d \
#   -p 8888:8888 \
#   --name my_jupyter_container \
#   my-anaconda-image \
#   jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root --NotebookApp.token='' --NotebookApp.password=''


==> Dockerfile.anaconda.pt1 <==
FROM ubuntu:24.04

# Install basic dependencies
RUN apt-get update && apt-get install -y \
    bzip2 \
    ca-certificates \
    git \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender1 \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Install Anaconda
RUN wget https://repo.anaconda.com/archive/Anaconda3-2025.12-2-Linux-x86_64.sh -O /tmp/anaconda.sh && \
    bash /tmp/anaconda.sh -b -p /opt/anaconda && \
    rm /tmp/anaconda.sh

# Put conda in PATH
ENV PATH="/opt/anaconda/bin:${PATH}"

COPY Dockerfile.anaconda.pt1 /

CMD ["/bin/bash"]


# docker build -t my-anaconda-image.pt1 -f Dockerfile.anaconda.pt1 .


==> Dockerfile.anaconda.pt2 <==
FROM my-anaconda-image.pt1

RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r

RUN conda init
RUN conda create --name my_env python=3.10 pandas numpy -y
RUN conda activate my_env

COPY Dockerfile.anaconda.* /

CMD ["/bin/bash"]


# docker build -t my-anaconda-image -f Dockerfile.anaconda.pt2 .

# docker run -d \
#   -p 8888:8888 \
#   --name my_jupyter_container \
#   my-anaconda-image \
#   jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root --NotebookApp.token='' --NotebookApp.password=''


==> Dockerfile.miniconda <==
FROM ubuntu:22.04

# Install basic dependencies
RUN apt-get update && apt-get install -y \
    wget \
    bzip2 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Miniconda (smaller, faster alternative to full Anaconda)
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh

# Put conda in PATH
ENV PATH="/opt/conda/bin:${PATH}"

# Copy an environment file if you have one (optional)
# COPY environment.yml .
# RUN conda env create -f environment.yml

COPY Dockerfile.miniconda /
CMD ["/bin/bash"]


# docker build -t my-miniconda-image -f Dockerfile.miniconda .

# docker run -d \
#   -p 8888:8888 \
#   --name my_jupyter_container \
#   my-miniconda-image \
#   jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root --NotebookApp.token='' --NotebookApp.password=''


# ===== crane

export CH_IMAGE_USER=$DOCKER_USER 
export CH_IMAGE_PASSWORD=$DOCKER_TOKEN 

{ wget -qO- https://github.com/google/go-containerregistry/releases/latest/download/go-containerregistry_Linux_x86_64.tar.gz | tar xz crane; sudo mv crane /usr/local/bin/; }
{ echo "$CH_IMAGE_PASSWORD" | crane auth login index.docker.io -u "$CH_IMAGE_USER" --password-stdin; }


ch-image build -t ubuntu-curl:latest .

tar -vczf ./ubuntu-curl.tar -C /var/tmp/root.ch/img/ubuntu-curl+latest/ .

tar -tvf ubuntu-curl.tar | grep -i Dockerfile


# === doesn't work
crane append \
  --base ubuntu:latest \
  --new_layer ./ubuntu-curl.tar \
  --cmd '["/bin/bash"]' \
  -o ./my-final-image.tar


crane push ./my-final-image.tar docker.io/${CH_IMAGE_USER}/ubuntu-curl:latest





```

