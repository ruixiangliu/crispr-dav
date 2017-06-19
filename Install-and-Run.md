<center><h3>Installing and Running CRISPR-DAV Pipeline</h3></center>

<br>
The CRISPR-DAV pipeline can be run via a docker container or a physical installation.

### I. Running via docker container

The docker repository for CRISPR-DAV is called [**pinetree1/crispr-dav**](https://hub.docker.com/r/pinetree1/crispr-dav/). It's based on the official fedora image at Docker Hub, and has included the pipeline and prerequisite tools. No physical installation of them is required but you need to be able to run docker on your system. 

The pipeline includes two example projects. Here are steps to test run example1. Running example2 is quite similar. You may replace /Users/xyz/temp with your own absolute path. 

(1) Start the container interactively and mount a path of host to the container:

        docker run -it -v /Users/xyz/temp:/Users/xyz/temp pinetree1/crispr-dav bash

The docker image is about 1GB, and takes a few minutes to start up. This command mounts /Users/xyz/temp in the host to /Users/xyz/temp in the container. Inside the container, the pipeline's path is /opt/crispr-dav.

(2) After starting up, at the container prompt, go to example1 directory:

        cd /opt/crispr-dav/Examples/example1

(3) Start the pipeline:
      
        sh run.sh

(4) When the pipeline is finished, move the results to the shared directory in container:

        mv deliverables /Users/xyz/temp

(5) Exit from the container:

        exit

(6) On the host, open a browser to view the report file, index.html, in /Users/xyz/temp/deliverables/GENEX_CR1.


The general steps for analyzing your own project via the docker are similar. You'll need to prepare a set of input files: conf.txt, amplicon.bed, site.bed, sample.site, fastq.list, and run.sh, similar to those in the examples; and prepare reference genome or amplicon sequence. The important thing is to share your data directories with the container. For example, assuming that there are 3 directories on the host related to your project:

      /Users/xyz/temp/project: contains the input files.
      
      /Users/xyz/temp/rawfastq: contains the fastq files.
      
      /Users/xyz/temp/genome: contains the genome files.

You'll mount these directories to the container using the same paths:

    docker run -it -v /Users/xyz/temp/project:/Users/xyz/temp/project \
      -v /Users/xyz/temp/rawfastq:/Users/xyz/temp/rawfastq \
      -v /Users/xyz/temp/genome:/Users/xyz/temp/genome \
      pinetree1/crispr-dav bash

    cd /Users/xyz/temp/project

Then edit conf.txt, fastq.list, and run.sh to reflect the paths in the container. 

Start the pipeline by: sh run.sh. The results will be present in the project directory of the container and the host.


### II. Running via a physical installation

#### 1. Clone the repository
  
        git clone https://github.com/pinetree1/crispr-dav.git

In the resulting crispr-dav directory, all the Perl programs (\*.pl) use this line to invoke the perl in your environment: \#!/usr/bin/env perl. The path of env on your system may differ. If so, the path should be changed accordingly in all \*.pl files in crispr-dav directory.

#### 2. Install prerequisite tools    

The pipeline utilizes a set of tools, most of which are common in bioinformatics field. These include Perl and Python modules, R, and NGS tools.  

**A. Perl modules**

The following modules are required but may not be present in default perl install.  

    Config::Tiny
    Excel::Writer::XLSX
    JSON

Run this command to check whether they are already installed: 

    perl -e "use <module>", e.g. perl -e "use Config::Tiny"

If there is no output, the module is already installed. Error message will show up if it's not installed.

If you have root privilege, installing a perl module could be simple:

    cpanm <module>, e.g. cpanm Config::Tiny
 
If you prefer to install modules as a non-root user, these steps show how to install Config::Tiny into local directory $HOME/perlmod:   

    wget http://search.cpan.org/CPAN/authors/id/R/RS/RSAVAGE/Config-Tiny-2.23.tgz 
    tar xvfz Config-Tiny-2.23.tgz
    cd Config-Tiny-2.23
    perl Makefile.PL INSTALL_BASE=$HOME/perlmod
    make
    make install

The modules can be found in CPAN. Install the other modules similarly. Keep in mind that these modules could have dependencies. You will need to install them as well.

If a module is installed globally by root, it is already in @INC which has paths that perl searches for a module. 

But if the module is installed in a local path, you'll need to add the path to @INC by setting PERL5LIB: 

	export PERL5LIB=$HOME/perlmod/lib/perl5:$PERL5LIB

You may add the line to the pipeline script run.sh in crispr-dav directory.

**B. NGS tools**

- ABRA: Assembly Based ReAligner. Recommended version: [0.97]( https://github.com/mozack/abra/releases/download/v0.97/abra-0.97-SNAPSHOT-jar-with-dependencies.jar). **Java 1.7 or later is needed to run the realigner.**

- BWA: Burrows-Wheeler Aligner. **Make sure your version supports "bwa mem -M" command.** Recommended version: [0.7.15](https://sourceforge.net/projects/bio-bwa/files/bwa-0.7.15.tar.bz2/download). Also make sure the executable bwa is in PATH, because bwa in PATH is to be used by ABRA. 

- Samtools: Recommended version: [1.3.1](https://sourceforge.net/projects/samtools/files/samtools/1.3.1/samtools-1.3.1.tar.bz2/download). Older version of samtools is OK. 

- Bedtools2: **Make sure your version supports -F option in 'bedtools intersect' command.** Recommended version: [2.25.0]( https://github.com/arq5x/bedtools2/releases/download/v2.25.0/bedtools-2.25.0.tar.gz)

- PRINSEQ: Recommended version: [0.20.4](https://sourceforge.net/projects/prinseq/files/standalone/prinseq-lite-0.20.4.tar.gz/download). **Be sure to make the program prinseq-lite.pl executable:** 

        chmod +x prinseq-lite.pl

**C. R packages**

- R packages: ggplot2, reshape2, naturalsort

To install the packages, after starting R, type:

        >install.packages("ggplot2")
        >install.packages("reshape2")
        >install.packages("naturalsort")

If you get permission errors, check with your admin or install R in a local directory.

**D. Python program** 

Required: Pysamstats https://github.com/alimanfoo/pysamstats 

To install it as root, check the web site for instructions. 

To install it in home directory, you may try these steps:

- ***Install prerequisite pysam module:***

        pip install --install-option="--prefix=$HOME" pysam==0.8.4

This would install pysam in $HOME/lib/python2.7/site-packages, assuming your Python version is 2.7. The 'lib' could be lib64.

Then make pysam module searchable:

		export PYTHONPATH=$PYTHONPATH:$HOME/lib/python2.7/site-packages

- ***Install pysamstats:***

        git clone https://github.com/alimanfoo/pysamstats.git 
        cd pysamstats
        python setup.py install --prefix=$HOME

This would install an executable script pysamstats in $HOME/bin and pysamstats module in the same place as pysam.

Check whether the modules can be loaded:

        $ python
        >>>import pysam
        >>>import pysamstats
        >>>exit()

If there is no output, the installation is successful. 

You should add it to the pipeline's run.sh script, for example:
    
        export PYTHONPATH=$PYTHONPATH:$HOME/lib/python2.7/site-packages


#### 3. Test run 

CRISPR-DAV includes two examples in Examples directory. The example1 uses a genome as reference, whereas example2 uses an amplicon sequence as reference. The procedure to run the pipelines is similar in the examples.
 
        cd crispr-dav/Examples/example1

        Edit the conf.txt and run.sh accordingly. Remember to add commands of setting PERL5LIB and PYTHONPATH in run.sh if the Perl and Python modules were installed locally.  

        Start the pipeline: sh run.sh. This shell script invokes the main program crispr.pl which starts the pipeline.

The pipeline would create these directories: 

- align: contains the intermediate files. They can be removed once the HTML report is produced. For description of the file types, please check the README file in the directory.  

    Make sure not to put your source fastq files in this directory. They could be overwritten there.

- deliverables: contains the results. The HTML report file index.html is in a subdirectory.

### III. Preparing input files for the pipeline

- **Fastq files:**

These are the raw fastq files. They must be gzipped with file extension .gz. Put the fastq files in a directory outside the pipeline's output directory. Don't put these fastq files inside the pipeline's 'align' directory, as they could get overwritten.

- **Reference files:** 

An amplicon sequence or a genome can be used as a reference. If an amplicon sequence is used for reference, all you need is a fasta file containing the sequence.
 
If a genome is used as reference, you'll prepare a fasta file, BWA index, and refGene coordinate files 

A. Prepare fasta file:

For example, to parepare human genome hg19, download the chromosome sequence files from UCSC browser, combine them into one file, e.g. hg19.fa. 

B. Create bwa index: bwa index hg19.fa 

C. Download refGene table:

Go to UCSC Genome Broser (http://genome.ucsc.edu/cgi-bin/hgBlat), click Tools and select TableBrowser. Then make these selections:

    Assembly: hg19
    Group: Genes and Gene Predictions
    Track: RefSeq Genes
    Table: refGene
    Region: Genome
    Output format: all fields from selected table. 
    
The downloaded tab-delimited file should have these columns: bin, name, chrom, strand, txStart, txEnd, cdsStart, cdsEnd, exonStarts, exonEnds,...

- **amplicon.bed:**

A tab-delimited text file with 6 columns for: chr, start, end, genesymbol, refseq_accession, strand. Only one amplicon is allowed. The start and end are 0-based according to BED format. The start is inclusive and the end is exclusive. Genesymbol should have no space. Refseq_accession must match the value in the "name" field (2nd column) in genome's refGene table for the gene.

- **site.bed:**

A tab-delimited text file with 6 or 7 columns for: chr, start, end, crispr_name, sgRNA_sequence, strand, HDR_new_bases_and_positions. This file can contain multiple rows, but crispr_name and sgRNA_sequence must be unique. All the CRISPR sites must belong to the same amplicon. Start and end are 0-based. 

The 7th field is optional. If HDR is performed, enter expected base changes in the field. 

HDR format: <Pos1><NewBase1>,<Pos2><NewBase2>,... The bases are desired new bases on ***positive strand***, e.g.101900208C,101900229G,101900232C,101900235A. No space. The positions are 1-based.


- **sample.site:**

This file controls what samples to be analyzed. It's a tab-delimited text file with 2 or more columns for: sample name, sgRNA_sequence1, sgRNA_sequence2, ... 

- **fastq.list:**

A tab-delimited text file with 2 or 3 columns for: sample name, read1 file, optional read2 file. Fastq files must be gzipped with .gz extension. The sample name must match what's in sample.site. 

- **conf.txt:**

Use the conf.txt in crispr.pl script directory as template, modify the paths and settings accordingly.

Please note that none of the tab-delimited files should have a column header row. The names of these files can be changed, as long as they match what's in the shell script run.sh.


### IV. Troubleshooting

- **Errors before pipeline starts:**

These are errors related to prerequisite tools and data inputs. For example, if a required tool or module is not found, there will be error messaging indicating the issue. You may try setting PERL5LIB and PYTHONPTH if the module is installed. If an input file that should be tab-separated, the pipeline may report missing columns.  
 
- **Errors during pipeline:**

  There are several log files per sample. Check the README in 'align' directory for descriptions. For example, if the installed Bedtools version does not support "bedtools insert -F", there will be error messages in the <sample>.log file. 