FROM ubuntu:22.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install base compilers, tools, and Perl via apt (more reliable than URL installer)
RUN apt-get update -y && apt-get upgrade -y && \
    apt-get install -y \
        gfortran build-essential git wget curl unzip \
        pigz mafft fasttree zlib1g bc \
        python3.9 python3-pip \
        perl \
        libc6 libreadline6-dev && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="${PATH}:/bin"

# Install samtools dependencies
RUN apt-get update -y && \
    apt-get install -y libncurses-dev libbz2-dev liblzma-dev && \
    rm -rf /var/lib/apt/lists/*

# Install R dependencies
RUN apt-get update -y && \
    apt-get install -y \
        libblas-dev liblapack-dev libpcre2-dev libpcre3 libpcre3-dev \
        libcurl4-openssl-dev libreadline-dev && \
    rm -rf /var/lib/apt/lists/*

# Install IRMA dependencies and iVar build dependencies
RUN apt-get update -y && \
    apt-get install -y \
        minimap2 parallel muscle \
        autoconf automake libtool libhts-dev && \
    rm -rf /var/lib/apt/lists/*

# Install Python packages (--no-cache-dir prevents pip cache buildup)
RUN pip3 install --no-cache-dir --upgrade pip && \
    pip3 install --no-cache-dir \
        click scikit-learn==1.3.1 numpy pandas \
        pybind11 setuptools wheel scipy

# Install samtools — download, build, install, then remove source and archive
RUN wget https://github.com/samtools/samtools/releases/download/1.12/samtools-1.12.tar.bz2 && \
    tar xvjf samtools-1.12.tar.bz2 && \
    cd samtools-1.12/ && \
    ./configure && make && make install && \
    cd .. && \
    rm -rf samtools-1.12/ samtools-1.12.tar.bz2

# Install R 4.2.0 — enable deb-src, build, then remove source and archive
RUN cp /etc/apt/sources.list /etc/apt/sources.list~ && \
    sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list && \
    apt-get update -y && \
    wget https://cran.r-project.org/src/base/R-4/R-4.2.0.tar.gz && \
    tar -xf R-4.2.0.tar.gz && \
    cd R-4.2.0 && \
    ./configure --without-x && \
    make -j9 && make install && \
    cd .. && \
    rm -rf R-4.2.0/ R-4.2.0.tar.gz && \
    rm -rf /var/lib/apt/lists/*

# Install IRMA v1.3.1 — download, extract, move to known path, remove archive
RUN wget https://github.com/CDCgov/irma/releases/download/v1.3.1/irma-v1.3.1-universal.zip && \
    unzip irma-v1.3.1-universal.zip && \
    rm irma-v1.3.1-universal.zip && \
    IRMA_FOUND=$(find / -maxdepth 4 -iname "irma" -type f \
        -not -path "*/proc/*" -not -path "*/sys/*" 2>/dev/null | head -1) && \
    IRMA_DIR=$(dirname "$IRMA_FOUND") && \
    IRMA_BIN=$(basename "$IRMA_FOUND") && \
    mv "$IRMA_DIR" /irma_dir && \
    chmod +x "/irma_dir/$IRMA_BIN" && \
    if [ "$IRMA_BIN" != "irma" ]; then \
        ln -sf "/irma_dir/$IRMA_BIN" /usr/local/bin/irma; \
    fi
ENV PATH="/irma_dir:${PATH}"

# Install iVar — clone, build, install, then remove source
RUN git clone https://github.com/andersen-lab/ivar.git && \
    cd ivar && \
    ./autogen.sh && ./configure && make && make install && \
    cd .. && rm -rf ivar/

# Install classLog — clone, install, then remove source
RUN git clone https://github.com/flu-crew/classLog.git && \
    cd classLog && \
    git clone https://github.com/arendsee/rpalign.git && \
    cd rpalign && pip3 install --no-cache-dir . && \
    cd .. && pip3 install --no-cache-dir . && \
    cd .. && rm -rf classLog/

# Install Dorado — download, extract, then remove archive (binary kept)
RUN wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-1.4.0-linux-x64.tar.gz && \
    tar -xf dorado-1.4.0-linux-x64.tar.gz && \
    rm dorado-1.4.0-linux-x64.tar.gz
ENV PATH="${PATH}:/dorado-1.4.0-linux-x64/bin/"

# Install FluPore
COPY ./flupore ./flupore
COPY ./REFS ./REFS
RUN chmod -R +x flupore
ENV PATH="${PATH}:/"

ENTRYPOINT ["flupore"]
