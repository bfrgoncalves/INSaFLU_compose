FROM ubuntu:16.04
MAINTAINER Bruno Gonçalves, bfgoncalves@medicina.ulisboa.pt

# Install package dependencies
RUN apt-get update && apt-get -y install binutils libproj-dev gdal-bin postgis* \
        bioperl python3 libdatetime-perl libxml-simple-perl libdigest-md5-perl \
        git default-jre bioperl bash python wget bzip2 gzip openjdk-8-jre perl \
        unzip parallel make python-pip locales

RUN pip install -U setuptools && locale-gen --purge en_US.UTF-8 && \
        DEBIAN_FRONTEND="noninteractive" dpkg-reconfigure locales && \
        update-locale LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8

WORKDIR /NGStools

# Install MINICONDA
RUN wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
RUN chmod +x Miniconda3-latest-Linux-x86_64.sh
RUN ./Miniconda3-latest-Linux-x86_64.sh -b -p /NGStools/miniconda
ENV PATH="/NGStools/miniconda/bin:$PATH"

# Configure channels for bioconda
RUN conda update -y -n base conda && conda config --add channels conda-forge && \
        conda config --add channels defaults && \
        conda config --add channels bioconda

# Install IGVTools (missing)

# Install SPAdes 3.11
RUN wget http://cab.spbu.ru/files/release3.11.1/SPAdes-3.11.1-Linux.tar.gz && \
        tar -xf SPAdes-3.11.1-Linux.tar.gz

ENV PATH="/NGStools/SPAdes-3.11.1-Linux/bin:$PATH"

# Install Abricate
# ADD Virulence_finder database
RUN git clone https://bitbucket.org/genomicepidemiology/virulencefinder_db.git && \
        cd virulencefinder_db && cat *.fsa >> sequences
# ADD BacMet database
RUN mkdir /NGStools/bacmet
WORKDIR /NGStools/bacmet
RUN wget http://bacmet.biomedicine.gu.se/download/BacMet2_EXP_database.fasta
RUN mv BacMet2_EXP_database.fasta sequences

WORKDIR /NGStools

RUN conda install -y abricate
RUN abricate --check && abricate --setupdb
RUN cd /NGStools/miniconda/db && mkdir virulencefinder && \
        cp /NGStools/virulencefinder_db/sequences virulencefinder/ && \
        abricate --setupdb
RUN cd /NGStools/miniconda/db && mkdir bacmet && \
        cp /NGStools/bacmet/sequences bacmet/ && abricate --setupdb

# Install fastqc
ADD http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.7.zip /NGStools/
RUN	unzip fastqc_*.zip && chmod +x FastQC/fastqc && rm -rf fastqc_*.zip
ENV PATH="/NGStools/FastQC:$PATH"

# Install trimmomatic
ADD http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.36.zip /NGStools/
RUN unzip Trimmomatic-0.36.zip
WORKDIR /NGStools/Trimmomatic-0.36
RUN mv trimmomatic-0.36.jar trimmomatic.jar
WORKDIR /NGStools

# Install prokka
# Barrnap
RUN wget https://github.com/tseemann/barrnap/archive/0.8.tar.gz && \
        tar xf 0.8.tar.gz && rm 0.8.tar.gz

# Infernal
RUN wget http://eddylab.org/infernal/infernal-1.1.2-linux-intel-gcc.tar.gz && \
        tar xf infernal-1.1.2-linux-intel-gcc.tar.gz && \
        rm infernal-1.1.2-linux-intel-gcc.tar.gz && \
        cd infernal-1.1.2-linux-intel-gcc && ./configure && make && \
        make install && cd easel && make install && cd /NGStools && \
        rm -rf infernal-1.1.2-linux-intel-gcc/

# GenomeTools
RUN apt-get install -y libpango1.0-dev && \
        wget http://genometools.org/pub/genometools-1.5.9.tar.gz && \
        tar xf genometools-1.5.9.tar.gz && rm genometools-1.5.9.tar.gz && \
        cd genometools-1.5.9 && make && make install && cd gtpython && \
        python setup.py install && cd /NGStools && rm -rf genometools-1.5.9/

# gff3toembl
RUN mkdir gff3toembl && cd gff3toembl && \
        git clone https://github.com/sanger-pathogens/gff3toembl.git && \
        python gff3toembl/setup.py install && cd /NGStools

# Blast+
RUN wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.6.0/ncbi-blast-2.6.0+-x64-linux.tar.gz && \
        tar xf ncbi-blast-2.6.0+-x64-linux.tar.gz && \
        rm ncbi-blast-2.6.0+-x64-linux.tar.gz

# CD-HIT
RUN wget https://github.com/weizhongli/cdhit/releases/download/V4.6.8/cd-hit-v4.6.8-2017-0621-source.tar.gz && \
        tar xf cd-hit-v4.6.8-2017-0621-source.tar.gz && \
        rm cd-hit-v4.6.8-2017-0621-source.tar.gz && \
        cd cd-hit-v4.6.8-2017-0621 \
        && make && cd cd-hit-auxtools && make && \
        cd /NGStools

# Exonerate
RUN wget http://ftp.ebi.ac.uk/pub/software/vertebrategenomics/exonerate/exonerate-2.2.0-x86_64.tar.gz && \
        tar xf exonerate-2.2.0-x86_64.tar.gz --no-same-owner && \
        chmod 755 exonerate-2.2.0-x86_64/ && rm exonerate-2.2.0-x86_64.tar.gz

# Prokka
RUN git clone https://github.com/tseemann/prokka.git && \
        prokka/bin/prokka --setupdb

# tbl2asn
RUN wget ftp://ftp.ncbi.nih.gov/toolbox/ncbi_tools/converters/by_program/tbl2asn/linux.tbl2asn.gz && \
    gunzip linux.tbl2asn.gz && \
    mv linux.tbl2asn tbl2asn && \
    chmod +x tbl2asn && \
    mv tbl2asn /NGStools/prokka/binaries/linux/

ENV PATH="/NGStools/gff3toembl/gff3toembl/scripts:/NGStools/ncbi-blast-2.6.0+/bin:/NGStools/cd-hit-v4.6.8-2017-0621:/NGStools/cd-hit-v4.6.8-2017-0621/cd-hit-auxtools:/NGStools/exonerate-2.2.0-x86_64/bin:/NGStools/prokka/bin:${PATH}" LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH" PYTHONPATH="/NGStools/gff3toembl/gff3toembl:$PYTHONPATH"

# Install mauve
RUN wget http://darlinglab.org/mauve/snapshots/2015/2015-02-13/linux-x64/mauve_linux_snapshot_2015-02-13.tar.gz
RUN tar xf mauve_linux_snapshot_2015-02-13.tar.gz && rm mauve_linux_snapshot_2015-02-13.tar.gz
ENV PATH="/NGStools/mauve_snapshot_2015-02-13:/NGStools/mauve_snapshot_2015-02-13/linux-x64:$PATH"

RUN echo $PATH

# Install mafft
RUN conda install -c bioconda mafft

# Install EMBOSS
RUN wget ftp://emboss.open-bio.org/pub/EMBOSS/EMBOSS-6.6.0.tar.gz && \
        gunzip EMBOSS-6.6.0.tar.gz && \
        tar xvf EMBOSS-6.6.0.tar
RUN cd EMBOSS-6.6.0 && ./configure && make

# Install FastTree
RUN wget http://www.microbesonline.org/fasttree/FastTreeDbl

# Install FreeBayes
RUN git config --global url.https://github.com/.insteadOf git://github.com/
RUN git clone --recursive git://github.com/ekg/freebayes.git
RUN apt-get -y install libbz2-dev liblzma-dev
RUN cd freebayes && make && make install

# Install Snippy
RUN conda install -c conda-forge -c bioconda snippy

# Get convertAlignment.pl
WORKDIR /NGStools/scripts/
RUN wget https://raw.githubusercontent.com/lskatz/lyve-SET/master/scripts/convertAlignment.pl

# Get fastq-tools
WORKDIR /NGStools
RUN apt-get -y install autoconf libtool
RUN git clone https://github.com/dcjones/fastq-tools.git && cd fastq-tools && \
        chmod a+x ./autogen.sh && ./autogen.sh && ./configure && make install

# Install INSaFLU web app
RUN git clone https://github.com/INSaFLU/INSaFLU.git
RUN apt-get -y install python3-pip
RUN cd INSaFLU/ && pip3 install -r requirements.txt

RUN echo "AQUI!"
COPY entrypoint.sh entrypoint.sh
RUN chmod a+x entrypoint.sh

ENTRYPOINT ["/NGStools/entrypoint.sh"]









