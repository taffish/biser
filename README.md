# taf-biser

TAFFISH wrapper for [BISER](https://github.com/0xTCG/biser), a tool for
detecting and decomposing segmental duplications in one or more genome
assemblies.

This repository packages upstream BISER 1.4 as a TAFFISH tool app. It exposes
upstream `biser` through a versioned TAFFISH entry point while keeping BISER,
Python dependencies, Samtools, and HTSlib inside a portable Debian 12 container
image.

This release currently publishes a `linux/amd64` container image. Native arm64
support is not declared because upstream BISER 1.4 is distributed on PyPI as an
x86_64 wheel, and rebuilding the Codon component for arm64 requires additional
upstream-source patching and validation.

For Docker and Podman, `src/main.taf` declares `--platform linux/amd64`, so
Apple Silicon or other arm64 hosts can use normal Docker/Podman amd64 emulation
without setting a global platform variable for every run. This is emulation,
not native arm64 support; Apptainer compatibility depends on the host and site
configuration.

## Installation

Install from the public TAFFISH Hub index:

```sh
taf update
taf install biser
```

Install the exact release:

```sh
taf install biser 1.4-r2
```

For local testing before the app is published to the public index:

```sh
taf install --from .
```

## Usage

Show TAFFISH app help:

```sh
taf-biser --help
```

Show upstream BISER help and version:

```sh
taf-biser biser --help
taf-biser biser --version
```

Index input FASTA files with Samtools before running BISER:

```sh
taf-biser samtools faidx genome.fa
```

Run BISER on a single genome:

```sh
taf-biser biser -o output -t 8 genome.fa
```

Run BISER on multiple genomes:

```sh
taf-biser biser -o output -t 8 genome1.fa genome2.fa
```

BISER works best with soft-masked genome assemblies. If the input assemblies
are hard-masked, use the upstream `--hard` option:

```sh
taf-biser biser -o output -t 8 --hard genome.fa
```

Because this is a command-mode TAFFISH tool, the first non-option argument is
treated as a command available inside the container image. Use explicit command
mode for BISER itself and for companion tools:

```sh
taf-biser biser --help
taf-biser samtools faidx genome.fa
taf-biser python --version
```

When passing an upstream option through the default wrapper, use `--` so the
option is handled by BISER instead of the TAFFISH wrapper:

```sh
taf-biser -- --help
taf-biser -- --version
```

For most day-to-day use, explicit command mode is clearer:

```sh
taf-biser biser --help
taf-biser biser --version
```

## Package

```text
name: biser
command: taf-biser
version: 1.4-r2
kind: tool
image: ghcr.io/taffish/biser:1.4-r2
platforms: linux/amd64
```

## Container

The container image is built from `docker/Dockerfile`. It compiles HTSlib
1.23.1 and Samtools 1.23.1 from upstream release tarballs in a builder stage,
then installs BISER 1.4 and its Python runtime dependencies into a virtual
environment in a slim Debian 12 runtime image.

The image includes:

```text
biser
samtools
python
```

The packaged Python environment includes:

```text
biser
dill
multiprocess
ncls
numpy
tqdm
```

Samtools is included because upstream BISER expects genome FASTA files to be
indexed with `samtools faidx` before analysis. HTSlib is built with libcurl,
S3, GCS, libdeflate, lzma, and bzip2 support through the bundled Samtools
build.

The TAFFISH metadata declares a Docker smoke check:

```text
exist: biser, samtools, python
test:  biser --version 2>&1 | grep -F 'biser v1.4' >/dev/null
test:  biser --help 2>&1 | grep -F 'Segmental duplication detection tool' >/dev/null
test:  samtools --version 2>&1 | grep -F 'samtools 1.23.1' >/dev/null
test:  samtools --version 2>&1 | grep -F 'Using htslib 1.23.1' >/dev/null
test:  python -c 'import biser, ncls, numpy, tqdm, multiprocess'
test:  printf '>chr1\nACGTACGT\n' >/tmp/taf-biser.fa && samtools faidx /tmp/taf-biser.fa && test -s /tmp/taf-biser.fa.fai
```

During TAFFISH Hub indexing, this smoke metadata is used to verify that the
published image can be inspected, that BISER, Samtools, and Python are
available, that the packaged BISER command reports version 1.4, that the
bundled Samtools/HTSlib versions are pinned, and that `samtools faidx` can
create a FASTA index.

## Upstream

- Project: BISER
- Source: <https://github.com/0xTCG/biser>
- PyPI: <https://pypi.org/project/biser/>
- Documentation: <https://github.com/0xTCG/biser#readme>
- Upstream license: MIT

## Maintainer Notes

Useful checks before publishing:

```sh
taf check
taf compile -- --help
taf compile -- biser --version
taf publish --release --dry-run
docker build --check -f docker/Dockerfile .
docker build --platform linux/amd64 -t ghcr.io/taffish/biser:1.4-r2 -f docker/Dockerfile .
docker run --rm --platform linux/amd64 ghcr.io/taffish/biser:1.4-r2 biser --version
docker run --rm --platform linux/amd64 ghcr.io/taffish/biser:1.4-r2 samtools --version
```

The repository wrapper files are licensed under Apache-2.0. BISER, Samtools,
HTSlib, and third-party runtime components are distributed under their own
upstream licenses.
