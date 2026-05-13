ARG BISER_VERSION=1.4
ARG SAMTOOLS_VERSION=1.23.1
ARG HTSLIB_VERSION=1.23.1

FROM debian:12-slim AS samtools-builder

ENV DEBIAN_FRONTEND=noninteractive
ARG SAMTOOLS_VERSION
ARG HTSLIB_VERSION
ENV LD_LIBRARY_PATH=/opt/samtools/lib

RUN set -eux; \
    apt-get -o Acquire::Retries=5 update; \
    for attempt in 1 2 3 4 5; do \
      if apt-get \
          -o Acquire::Retries=5 \
          -o Acquire::http::Pipeline-Depth=0 \
          install -y --no-install-recommends \
            build-essential \
            ca-certificates \
            curl \
            libbz2-dev \
            libcurl4-gnutls-dev \
            libdeflate-dev \
            liblzma-dev \
            libncurses-dev \
            libssl-dev \
            zlib1g-dev; then \
        break; \
      fi; \
      if [ "$attempt" = "5" ]; then exit 1; fi; \
      apt-get -o Acquire::Retries=5 update; \
      sleep 10; \
    done; \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*.deb

WORKDIR /tmp/build

RUN curl -fsSL "https://github.com/samtools/htslib/releases/download/${HTSLIB_VERSION}/htslib-${HTSLIB_VERSION}.tar.bz2" \
      -o htslib.tar.bz2 \
 && tar -xjf htslib.tar.bz2 \
 && curl -fsSL "https://github.com/samtools/samtools/releases/download/${SAMTOOLS_VERSION}/samtools-${SAMTOOLS_VERSION}.tar.bz2" \
      -o samtools.tar.bz2 \
 && tar -xjf samtools.tar.bz2

WORKDIR /tmp/build/htslib-${HTSLIB_VERSION}

RUN ./configure --prefix=/opt/samtools \
 && make -j"$(nproc)" \
 && make install

WORKDIR /tmp/build/samtools-${SAMTOOLS_VERSION}

RUN ./configure \
      --prefix=/opt/samtools \
      --with-htslib=/opt/samtools \
 && make -j"$(nproc)" \
 && make install \
 && /opt/samtools/bin/samtools --version 2>&1 | grep -F "samtools ${SAMTOOLS_VERSION}" >/dev/null \
 && /opt/samtools/bin/samtools --version 2>&1 | grep -F "Using htslib ${HTSLIB_VERSION}" >/dev/null \
 && /opt/samtools/bin/samtools faidx --help 2>&1 | grep -F "Usage: samtools faidx" >/dev/null

FROM debian:12-slim

ENV DEBIAN_FRONTEND=noninteractive
ARG BISER_VERSION
ARG SAMTOOLS_VERSION
ARG HTSLIB_VERSION
ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PIP_NO_CACHE_DIR=1

RUN set -eux; \
    apt-get -o Acquire::Retries=5 update; \
    for attempt in 1 2 3 4 5; do \
      if apt-get \
          -o Acquire::Retries=5 \
          -o Acquire::http::Pipeline-Depth=0 \
          install -y --no-install-recommends \
            ca-certificates \
            libbz2-1.0 \
            libcurl3-gnutls \
            libdeflate0 \
            liblzma5 \
            libncursesw6 \
            libssl3 \
            python3 \
            python3-venv \
            zlib1g; then \
        break; \
      fi; \
      if [ "$attempt" = "5" ]; then exit 1; fi; \
      apt-get -o Acquire::Retries=5 update; \
      sleep 10; \
    done; \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*.deb

COPY --from=samtools-builder /opt/samtools /opt/samtools

ENV PATH=/opt/biser/bin:/opt/samtools/bin:${PATH}
ENV LD_LIBRARY_PATH=/opt/samtools/lib

RUN python3 -m venv /opt/biser \
 && /opt/biser/bin/python -m pip install --upgrade pip setuptools wheel \
 && /opt/biser/bin/python -m pip install --only-binary=:all: \
      "biser==${BISER_VERSION}" \
      "dill==0.4.1" \
      "multiprocess==0.70.19" \
      "ncls==0.0.70" \
      "numpy==2.4.4" \
      "tqdm==4.67.3" \
 && biser --version 2>&1 | grep -F "biser v${BISER_VERSION}" >/dev/null \
 && biser --help 2>&1 | grep -F "Segmental duplication detection tool" >/dev/null \
 && samtools --version 2>&1 | grep -F "samtools ${SAMTOOLS_VERSION}" >/dev/null \
 && samtools --version 2>&1 | grep -F "Using htslib ${HTSLIB_VERSION}" >/dev/null \
 && python -c 'import biser, ncls, numpy, tqdm, multiprocess' \
 && printf ">chr1\nACGTACGT\n" >/tmp/taf-biser.fa \
 && samtools faidx /tmp/taf-biser.fa \
 && test -s /tmp/taf-biser.fa.fai \
 && rm -f /tmp/taf-biser.fa /tmp/taf-biser.fa.fai

RUN ln -sf /opt/biser/bin/biser /usr/local/bin/biser \
 && ln -sf /opt/samtools/bin/samtools /usr/local/bin/samtools \
 && printf '%s\n' '#!/bin/sh' 'exec /opt/biser/bin/python "$@"' >/usr/local/bin/python \
 && chmod +x /usr/local/bin/python \
 && ln -sf /usr/local/bin/python /usr/local/bin/python3

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=biser
