taf-biser 1.4-r1

TAFFISH wrapper for BISER, a tool for detecting and decomposing segmental
duplications in one or more genome assemblies.

Usage:
  taf-biser [TAF-APP-OPTION]
  taf-biser biser [BISER-ARGS...]
  taf-biser samtools [SAMTOOLS-COMMAND] [COMMAND-ARGS...]
  taf-biser python [PYTHON-ARGS...]
  taf-biser -- [BISER-OPTION...]
  taf-biser <COMMAND> [COMMAND-ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Upstream help:
  taf-biser biser --help
  taf-biser biser --version
  taf-biser samtools --version

Recommended BISER examples:
  taf-biser samtools faidx genome.fa
  taf-biser biser -o output -t 8 genome.fa
  taf-biser biser -o output -t 8 genome1.fa genome2.fa
  taf-biser biser -o output -t 8 --hard genome.fa
  taf-biser biser -o output -t 8 --keep-contigs scaffolds.fa
  taf-biser biser -o output -t 8 --no-decomposition genome.fa

Common BISER options:
  -o, --output       Output prefix
  -t, --threads      Number of threads
  -T, --temp         Temporary directory location
  -H, --hard         Treat input genomes as hard-masked
  --keep-contigs     Keep contigs and custom assembly sequences
  --keep-temp        Keep the temporary directory
  --resume           Resume a previous run kept with --keep-temp
  --no-decomposition Skip SD decomposition output
  --gc-heap          Set GC_INITIAL_HEAP_SIZE for large runs

Notes:
  - This command runs BISER inside the TAFFISH container image.
  - Use "taf-biser biser <ARGS...>" for ordinary BISER workflows.
  - Input FASTA files should be indexed first with "taf-biser samtools faidx".
  - BISER works best with soft-masked genome assemblies. Use "--hard" only for
    hard-masked inputs.
  - Use explicit command mode for companion tools available in the image, such
    as "samtools" and "python".
  - Use "--" before upstream options when an option may be handled by the
    TAFFISH wrapper itself, such as "--help" or "--version".
  - Input and output paths should be accessible from the current working
    directory or from mounted user paths.
  - This image packages upstream BISER 1.4 with Samtools 1.23.1 and HTSlib
    1.23.1 in a Debian 12 runtime image.
  - This release currently declares linux/amd64 only. Native arm64 support is
    not claimed for BISER 1.4-r1.

Container:
  image: ghcr.io/taffish/biser:1.4-r1
  platforms: linux/amd64
  supported backends: apptainer, podman, docker

Upstream:
  project: BISER
  source:  https://github.com/0xTCG/biser
  pypi:    https://pypi.org/project/biser/
