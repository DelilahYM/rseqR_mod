# rseqR <img src="man/figures/logo.png" align="right" />

The rseqR package is designed to perform differential analysis of
RNA-seq data. It provides tools to:

  - perform quality control of raw reads in fastq format and generate
    HTML reports (using FASTQC and MULTIQC)
  - determine trancript and gene-level read count estimates (using
    Salmon)
  - run differential analysis using DESeq2, edgeR, and Limma-voom and to
    export the results into easily readable tab-delimited files
  - perform pathway analysis of differentially expressed genes

The rseqR package does not attempt to replace the tools it uses, but
instead provides an automated environment in which to use them.

The website for rseqR may be found
[here](https://anilchalisey.github.io/rseqR/).

# Installation

## Command line tools

The workflow used by rseqR requires the installation of several
command-line tools: [Fastqc
(v0.11.7)](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Fastqc%5D),
[MultiQC (v1.6)](http://multiqc.info/), and [Salmon
(v0.10.2)](https://combine-lab.github.io/salmon/). Installation
instructions for these may be found at their respective websites, but a
guide is given below for convenience.

If using a Unix-based system, open up a terminal and follow the commands
as is. If using the windows subsystem for linux (WSL) on Windows 10 then
WSL must first be set up as detailed
[here](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Once
WSL is up and running, then the tools may be installed as on any
Unix-based system.

It is assumed that root priviliges are available (if necessary) - if
not, then a system administrator may need to install these for you.

### Fastqc

``` bash
cd ~
wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.7.zip
unzip fastqc_v0.11.7.zip
cd FastQC
chmod 755 fastqc

# If using WSL
cd /usr/local/bin
sudo ln -s ~/Fastqc/fastqc .

# If using other Unix
cd ~/bin
ln -s ~/Fastqc/fastqc .
```

### MultiQC

To run MultiQC ensure there is a working python distribution.

``` bash
pip install multiqc
```

### Salmon

``` bash
cd ~
wget https://github.com/COMBINE-lab/salmon/releases/download/v0.11.2/salmon-0.11.2-linux_x86_64.tar.gz
tar -xvzf salmon-0.11.2-linux_x86_64.tar.gz

# If using WSL
cd /usr/local/bin 
sudo ln -s ~/salmon-0.11.2-linux_x86_64/bin/salmon .

# If using other Unix
cd ~/bin
ln -s ~/salmon-0.11.2-linux_x86_64/bin/salmon .
```

### R package dependencies

There are a number of dependencies to the rseqR package detailed in the
`DESCRIPTION` file. These will all be installed automatically during the
installation of rseqR.

### Install rseqR

Once all dependencies are installed, then rseqR may be installed as
follows:

``` r
devtools::install_github("anilchalisey/rseqr", build_vignettes = TRUE)
```

## Usage

The entire workflow of rseqR may be run by a call to a single function
`run_dea()`, as shown
below.

``` r
result <- run_dea(sample.info = `system.file("extdata", "paired-example.txt", pkg = "rseqr")`,
                  reference = c("WT", "KO1", "KO2"), species = "human", output.dir = "results-dea",
                  threads = NULL, index.dir = NULL)
```

The arguments to the `run_dea()` function are:

### sample.info:

This is the path to a tab-delimited file with at least the columns:

  - `condition`: treatment or condition labels on the basis of which the
    differential analysis will be performed (e.g. ???CONTROL???, ???TREATED???;
    or ???WILD-TYPE???, ???KNOCK-OUT1???, ???KNOCK-OUT2???).
  - `sample`: names of the samples - these labels will be used in plots
    and as sample column headers in the output.
  - `file1`: absolute or relative path to fastq or salmon quant.sf
    files.

Optional columns include:

  - `file2`: if fastq files and PE reads, then this column should also
    be present, specifying the absolute or relative paths to the second
    pair of fastq reads.  
    \# `batch`: if a batch effect is to be included in the design, then
    this should be identified under this column (e.g.??litter number or
    sequencing run).

If starting from fastq files, then the workflow will include QC of raw
reads, and read count estimation with salmon. If a salmon index has not
been identified then an index will be built automatically prior to read
count estimation. If starting from quant.sf files, then the analysis
will start directly at the differential analysis stage.

Dummy examples of the tab-delimited files accepted by rseqR may be found
at `system.file("extdata", "paired-example.txt", pkg = "rseqr")`,
`system.file("extdata", "sinle-example.txt", pkg = "rseqr")`, and
`system.file("extdata", "quants-example.txt", pkg = "rseqr")`.

### reference:

The order in which the condition labels should be evaluated when
performing differential analysis. For example, c(???A???, ???B???, ???C???, ???D???)
would mean ???A??? is the reference condition to which ???B???, ???C??? and ???D??? are
compared; in addition, ???C??? and ???D??? will be compared to ???B???, and ???D??? will
be compared to ???C???. If  then the comparisons will be arranged
alphabetically.

### species:

May be one of ???human??? or ???mouse??? - other options will be added in later
versions.

### output.dir:

Directory to which results should be saved.

### threads:

Number of threads to be used when parallelisation is possible. If `NULL`
then one less than the maximum numbre of threads available will be used.

### index.dir:

The path to the salmon index.

Other arguments are also possible, and details for these may be found in
the manual, but for most users, the default settings are satisfactory.

## Output

Once complete, all the results will be saved in the specified output
directory with the following structure:

    #>                levelName
    #> 1  results              
    #> 2   ??--DE_results.rda   
    #> 3   ??--DESeq2           
    #> 4   ??--edgeR            
    #> 5   ??--fastqc           
    #> 6   ??--index            
    #> 7   ??--Limma-voom       
    #> 8   ??--metadata.rda     
    #> 9   ??--multiqc          
    #> 10  ??--salmon           
    #> 11  ??--upset_KO1-WT.png 
    #> 12  ??--upset_KO2-WT.png 
    #> 13  ??--upset_KO2-KO1.png

The directories `DESeq2`, `edgeR`, `fastqc`, `Limma-voom`, and `multiqc`
contain the results of the respective analyses including relevant plots.
Within the `salmon` directory one will find individual directories for
the quantification results of each sample. The `index` directory
contains the salmon index for human or mouse if it had not been created
and specified beforehand. `DE-results.rda` contains the results of the
differential expression analysis and comprises a list of three objects:
`DE_limma`, `DE_edger`, and `DE_deseq2`. The `metadata.rda` object
contains the metadata for the samples including the count data for each
sample; this may be used to re-run the analysis from an intermediate
step.

## Running only part of the rseqR pipeline

If there is a need to reanalyse only a portion of the data, then this
may be done using the `metadata.rda` object, which contains all the
necessary information. For example, it is possible to run each of the
differential analysis tools separately using the commands
`limma_voom_analysis()`, `edger_analysis()`, or `deseq2_analysis()`. For
example:

``` r
load("results/metadata.rda") # this will load an object called 'metadata' into the environment
DE_limma <- limma_voom_analysis(metadata = metadata, species = "human")
```

## Pathway analysis

Gene ontology and KEGG pathway analysis may be performed using the
`gage` package. This is performed by reading in the differential
analysis results written out to file and submitting this to the
`pathway_analysis()` function. In this example, we use the edgeR
results.

``` r
DEGs <- read.csv("results/edgeR/KO1-WT/edgeR_differential_expression.txt")
pa1_deseq2 <- pathway_analysis(DEGs, species = "human")
```

This may be repeated as necessary for the other comparisons.

## About rseqR

The rseqR package has been developed in the Chris O???Callaghan Group at
the Centre for Cellular and Molecular Biology, University of Oxford by
Anil Chalisey and Chris O???Callaghan.
