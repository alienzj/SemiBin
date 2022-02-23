# SemiBin: Semi-supervised Metagenomic Binning Using Siamese Neural Networks

Command tool for metagenomic binning with semi-supervised deep learning using
information from reference genomes in Linux and MacOS.

[![Test Status](https://github.com/BigDataBiology/SemiBin/actions/workflows/semibin_test.yml/badge.svg)](https://github.com/BigDataBiology/SemiBin/actions/workflows/semibin_test.yml)
[![Documentation Status](https://readthedocs.org/projects/semibin/badge/?version=latest)](https://semibin.readthedocs.io/en/latest/?badge=latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

_CONTACT US_: This tool is still in development. You are welcome to try it out and
feedback is appreciated, but expect some bugs/rapid changes until it
stabilizes. Please use [Github
issues](https://github.com/BigDataBiology/SemiBin/issues) for bug reports and
the [SemiBin users mailing-list](https://groups.google.com/g/semibin-users) for
more open-ended discussions or questions.

If you use this software in a publication please cite:

> SemiBin: Incorporating information from reference genomes with
> semi-supervised deep learning leads to better metagenomic assembled genomes
> (MAGs)
> Shaojun Pan, Chengkai Zhu, Xing-Ming Zhao, Luis Pedro Coelho
> bioRxiv 2021.08.16.456517; doi:
> [https://doi.org/10.1101/2021.08.16.456517](https://doi.org/10.1101/2021.08.16.456517)

## Basic usage of SemiBin

A tutorial of running SemiBin from scrath can be found [SemiBin tutorial](https://github.com/psj1997/SemiBin_tutorial_from_scratch).

Install

```bash
conda create -n SemiBin
conda activate SemiBin
conda install -c conda-forge -c bioconda semibin
```

The inputs to the SemiBin are contigs (assembled from the reads) and bam files (reads mapping to the contigs). The output are the bins in the output_recluster_bins directory (including the bin.\*.fa and recluster.\*.fa)).

Running with single-sample binning (for example: human gut samples)

```bash
SemiBin single_easy_bin -i contig.fa -b *.bam -o output --environment human_gut
```

Running with multi-sample binning 
```bash
SemiBin multi_easy_bin -i contig_whole.fa -b *.bam -o output -s :
```

Please find more options and details below and [read the docs](https://semibin.readthedocs.io/en/latest/usage/). 

## Advanced Installation

SemiBin runs on Python 3.7-3.9.

### Bioconda

The simplest mode is shown above.
However, if you want to use SemiBin with GPU (which is faster if you have one available), you need to install PyTorch with GPU support:

```bash
conda create -n SemiBin
conda activate SemiBin
conda install -c conda-forge -c bioconda semibin
conda install -c pytorch-lts pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch-lts
```

_MacOS note_: **you can only install the CPU version of PyTorch in MacOS with `conda` and you need to install from source to take advantage of a GPU** (see [#72](https://github.com/BigDataBiology/SemiBin/issues/72)).
For more information on how to install PyTorch, see [their documentation](https://pytorch.org/get-started/locally/).

### Source

You will need the following dependencies:
- [MMseqs2](https://github.com/soedinglab/MMseqs2)
- [Bedtools](http://bedtools.readthedocs.org/]), [Hmmer](http://hmmer.org/)
- [Prodigal](https://github.com/hyattpd/Prodigal)
- (optionally) [Fraggenescan](https://sourceforge.net/projects/fraggenescan/)

The easiest way to install the dependencies is with `conda`:

```bash
conda install -c conda-forge -c bioconda mmseqs2=13.45111 (for GTDB support)
conda install -c bioconda bedtools hmmer prodigal
conda install -c bioconda fraggenescan
```

Once the dependencies are installed, you can install SemiBin by running:

```bash
python setup.py install
```

## Examples of binning

_NOTE_: The `SemiBin` API is a work-in-progress. The examples refer to
version `0.6`, but this may change in the near future (after the release of
version 1.0, we expect to freeze the API for [at least 5
years](https://big-data-biology.org/software/commitments/)). We are very happy
to [hear any feedback on API
design](https://groups.google.com/g/semibin-users), though.

SemiBin runs on single-sample, co-assembly and multi-sample binning.
Here we show the simple modes as an example.
For the details and examples of every SemiBin subcommand, please [read the docs](https://semibin.readthedocs.io/en/latest/usage/).

## Easy single/co-assembly binning mode

Single sample and co-assembly are handled the same way by SemiBin.

You will need the following inputs:

1. A contig file (`contig.fa` in the example below)
2. BAM file(s) from mapping short reads to the contigs (`mapped_reads.bam` in the example below)

The `single_easy_bin` command can be used to produce results in a single step.

For example:

```bash
SemiBin \
    single_easy_bin \
    --input-fasta contig.fa \
    --input-bam mapped_reads.bam \
    --environment human_gut \
    --output output
```

Alternatively, you can train a new model for that sample, by not passing in the `--environment` flag:

```bash
SemiBin \
    single_easy_bin \
    --input-fasta contig.fa \
    --input-bam mapped_reads.bam \
    --output output
```

The following environments are supported:

- `human_gut`
- `dog_gut`
- `ocean`
- `soil`
- `cat_gut`
- `human_oral`
- `mouse_gut`
- `pig_gut`
- `built_environment`
- `wastewater`
- `global`

The `global` environment can be used if none of the others is appropriate.
Note that training a new model can take a lot of time and disk space.
Some patience will be required.
If you have a lot of samples from the same environment, you can also train a new model from them and reuse it.

## Easy multi-samples binning mode

The `multi_easy_bin` command can be used in multi-samples binning mode:

You will need the following inputs:

1. A combined contig file
2. BAM files from mapping

For every contig, format of the name is `<sample_name>:<contig_name>`, where
`:` is the default separator (it can be changed with the `--separator`
argument). _NOTE_: Make sure the sample names are unique and  the separator
does not introduce confusion when splitting. For example:

```bash
>S1:Contig_1
AGATAATAAAGATAATAATA
>S1:Contig_2
CGAATTTATCTCAAGAACAAGAAAA
>S1:Contig_3
AAAAAGAGAAAATTCAGAATTAGCCAATAAAATA
>S2:Contig_1
AATGATATAATACTTAATA
>S2:Contig_2
AAAATATTAAAGAAATAATGAAAGAAA
>S3:Contig_1
ATAAAGACGATAAAATAATAAAAGCCAAATCCGACAAAGAAAGAACGG
>S3:Contig_2
AATATTTTAGAGAAAGACATAAACAATAAGAAAAGTATT
>S3:Contig_3
CAAATACGAATGATTCTTTATTAGATTATCTTAATAAGAATATC
```

You can get the results with one line of code.

```bash
SemiBin concatenate_fasta --contig-files contig*.fa -o output
SemiBin multi_easy_bin -i output/concatenated.fa -b *.bam -o output
```

## Output

The output folder will contain:

1. Datasets used for training and clustering
2. Saved semi-supervised deep learning model
3. Output bins
4. Some intermediate files

By default, reconstructed bins are in `output_recluster_bins` directory.

For more details about the output, [read the
docs](https://semibin.readthedocs.io/en/latest/output/).
