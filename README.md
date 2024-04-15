# Seminar_2024_04_29
* Bacteria genome assembly and annotation (Canu and Prokka)

## 0.Before start

### 0.1 Anaconda install

Anaconda linl: https://www.anaconda.com/

    wget https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh
    sh ./Anaconda3-2024.02-1-Linux-x86_64.sh

### 0.2 Path setting

In home directory

    vi .profile

    press 'i'

    PATH='folder 위치':$PATH 입력하기 

    press 'esc'

    press ':'

    wq!

    enter

    
## 1.Bacterial genome size estimation
Canu는 assemble하고자 하는 genome의 대략적인 size를 알아야함.
따라서 genomes sequencing 데이터에서 k-mer 기반으로 genome size estimation 과정을 먼저 진행함.

### 1.1 Jellyfish installation

Jellyfish link: https://github.com/gmarcais/Jellyfish
    
    wget https://github.com/gmarcais/Jellyfish/releases/download/v2.3.1/jellyfish-2.3.1.tar.gz
    tar -zxvf jellyfish-2.3.1.tar.gz 
    cd jellyfish-2.3.1/
    ./configure
    make
    make install
또는

    wget https://github.com/gmarcais/Jellyfish/releases/download/v2.3.0/jellyfish-linux
    chmod 777 jellyfish-linux

### 1.2 Genome size estimation

    jellyfish count -m 21 -t 60 -s 900G -o /var2/sgmn0223/Seminar/Z.Jellyfish/Jellyfish_Illumina_17mer_out -C /var2/sgmn0223/Seminar/1.Raw_data_illumina/*.fastq

    jellyfish histo -t 70 /var2/sgmn0223/Seminar/Z.Jellyfish/Jellyfish_Illumina_21mer_out -o /var2/sgmn0223/Seminar/Z.Jellyfish/Jellyfish_Illumina_21mer_histo

### 1.3 Genome size calculation

http://genomescope.org/genomescope2.0

* K-mer length: 21
* Ploidy: 1

## 2.Bacterial genome assembly

### 2.1 Canu installation

Canu links: https://github.com/marbl/canu

    conda install -c conda-forge -c bioconda -c defaults canu

### 2.2 Genome assembly

    canu -p 2GT5 -d 2GT5_Canu_output genomeSize=5.5m maxThreads=70  -pacbio-raw /home/sgmn0223/Syn/2GT5/Pacbio/Raw_data/2GT5_rawdata1/Analysis_Results/All.fastq

## 3. Circularization (이전 seminar에서 진행하였음)
Circulator tool은 BWA, Prodigal, Samtools, Mummer tool들을 필요로 하여 먼저 설치 후 path 설정해주어야 함.

Circlator link: https://github.com/sanger-pathogens/circlator

### 3.1 필요한 tool들 설치


* BWA

BWA link: https://github.com/lh3/bwa  

      wget https://github.com/lh3/bwa/archive/refs/tags/v0.7.17.tar.gz
      tar -zxvf v0.7.17.tar.gz
      cd bwa-0.7.17/
      make
또는 
       
    git clone https://github.com/lh3/bwa.git
    cd bwa
    make
    
* Prodigal

Prodigal link: https://github.com/hyattpd/Prodigal

      wget https://github.com/hyattpd/Prodigal/releases/download/v2.6.3/prodigal.linux
      chmod 777 prodigal.linux
      mv prodigal.linux prodigal

* Samtools

Samtools link: https://www.htslib.org/
      
      wget https://github.com/samtools/samtools/releases/download/1.19.2/samtools-1.19.2.tar.bz2
      tar -jxvf samtools-1.19.2.tar.bz2
      cd samtools-1.19.2
      ./configure
      make

* Mummer

Mummer link: https://github.com/mummer4/mummer

      wget https://github.com/mummer4/mummer/releases/download/v4.0.0rc1/mummer-4.0.0rc1.tar.gz
      tar -zxvf v4.0.0rc1.tar.gz
      cd mummer-4.0.0rc1
      ./configure
      make

### 3.2 Circlator install

      pip3 install circlator
  
### 3.3 Circularization by Circlator

    circlator all --threads 8 SPAdes_chromosome.fasta B04.fastq Circlator
      
* --threads:  number of threads
* SPAdes_chromosome.fasta: assembled genome sequence
* B04.fastq: Nanopre data
* Circlator: output directory


* Output file인 04.merge.circularise.log 확인하여  circularization check
* Circularised genome data: 06.fixstart.fasta

## 4.Polishing (option)
Pilon을 사용하기 위해서 BWA, Bowtie2를 설치해주어야함.

Pilon link: https://github.com/broadinstitute/pilon

### 4.1 필요한 tool들 설치

* BWA (이전 단계에서 설치함)

BWA link: https://github.com/lh3/bwa  

      wget https://github.com/lh3/bwa/archive/refs/tags/v0.7.17.tar.gz
      tar -zxvf v0.7.17.tar.gz
      cd bwa-0.7.17/
      make
또는 
       
    git clone https://github.com/lh3/bwa.git
    cd bwa
    make

* Bowtie2

Bowtie2 link: https://github.com/BenLangmead/bowtie2
  
    wget https://github.com/BenLangmead/bowtie2/releases/download/v2.5.3/bowtie2-2.5.3-linux-x86_64.zip
    unzip bowtie2-2.5.3-linux-x86_64.zip

### 4.2 Pilon install

    conda install bioconda::pilon

### Polising by Pilon

 -Xms -Xmx value change

    bowtie2-build circularized.fata Round1

    bowtie2 -p 8 -x Round1 -1 /home/sgmn0223/0.IBV_Seminar/20240325/0.Raw_data/1.Raw_data_illumina/B04_S13_R1_001.fastq -2 /home/sgmn0223/0.IBV_Seminar/20240325/0.Raw_data/1.Raw_data_illumina/B04_S13_R2_001.fastq -S Round1.sam
    
    samtools sort -@ 8 Round1.sam -o Round1.sorted.bam
    
    samtools index -@ 8 Round1.sorted.bam

    pilon --genome circularized.fasta --frags Round1.sorted.bam --output Round1


## 5.Bacterial genome annotation
Prokka 설치를 위해서 anaconda를 먼저 설치해주어야함.

Prokka link: https://github.com/tseemann/prokka

### 5.1 Prokka install

    conda install -c conda-forge -c bioconda -c defaults prokka
    
### 5.2 Prokka DB setup

    prokka --setupdb
    
### 5.3 Annotation by Prokka

    prokka -outdir ./Prokka_output 06.fixstart.fasta --cpus 60
    
* 06.fixstart.fasta: circularised genome sequence
* --outdir: output directory
