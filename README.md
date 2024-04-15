# Seminar_2024_04_29
* Bacteria genome assembly and annotation (Canu and Prokka)

## 0.Before start

### 0.1 Anaconda install

Anaconda link: https://www.anaconda.com/

    wget https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh
    sh ./Anaconda3-2024.02-1-Linux-x86_64.sh

* anaconda 폴더 위치: /var2 내 자신의 folder로 지정 

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
    
    wget https://github.com/gmarcais/Jellyfish/releases/download/v2.3.0/jellyfish-linux
    chmod 777 jellyfish-linux

### 1.2 Genome size estimation

    jellyfish count -m 21 -t 8 -s 100M -o Jellyfish_Illumina_21mer_out -C *.fastq

* count: k-mer counting
* -m: length of mer
* -t: number of threads
* -s: initial hash size
* -o: output file
* -C: count both strand
* *.fastq: input file (illumina forward and reverse)
                 
      jellyfish histo -t 8 -o Jellyfish_Illumina_21mer_histo Jellyfish_Illumina_21mer_out

* histo: plot the histogram
* -t: number of threads
* -o: output file
* Jellyfish_Illumina_21mer_out: input file (output of jellyfish count)

### 1.3 Genome size calculation

http://genomescope.org/genomescope2.0

* K-mer length: used k-mer length
* Ploidy = 1

## 2.Bacterial genome assembly

### 2.1 Canu installation

Canu links: https://github.com/marbl/canu

    conda install -c conda-forge -c bioconda -c defaults canu

### 2.2 Genome assembly

    canu -p Seminar -d Canu_output genomeSize=5.4m maxThreads=8  -nanopore-raw B04.fastq

* -p: prefix
* -d: output
* genomeSize=: predicted genome size
* maxThreads=: number of threads
* -nanopore-raw: using nanopore data
* B04.fastq: input file (nanopore)

## 3. Circularization (이전 seminar에서 진행하였음)
Circulator tool은 BWA, Prodigal, Samtools, Mummer tool들을 필요로 하여 먼저 설치 후 path 설정해주어야 함.

Circlator link: https://github.com/sanger-pathogens/circlator

### 3.1 필요한 tool들 설치


* BWA

BWA link: https://github.com/lh3/bwa  
       
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

Fasta 파일 내 첫 sequence (가장 긴 서열) parsing

    awk '/^>/{if(NR>1) exit; print; next} {if (!/^>/) print}' 06.fixstart.fasta > circularized.fasta

## 4.Polishing (option)
Pilon을 사용하기 위해서 BWA, Bowtie2를 설치해주어야함.

Pilon link: https://github.com/broadinstitute/pilon

### 4.1 필요한 tool들 설치

* BWA (이전 단계에서 설치함)

BWA link: https://github.com/lh3/bwa  

    git clone https://github.com/lh3/bwa.git
    cd bwa
    make

* Bowtie2

Bowtie2 link: https://github.com/BenLangmead/bowtie2
  
    wget https://github.com/BenLangmead/bowtie2/releases/download/v2.5.3/bowtie2-2.5.3-linux-x86_64.zip
    unzip bowtie2-2.5.3-linux-x86_64.zip

### 4.2 Pilon install

    conda install bioconda::pilon

### 4.3 Polising by Pilon

 -Xms -Xmx value change (vi pilon)

    bowtie2-build circularized.fata Round1

* circularzied.fasta: input file (circularized genome)
* Round1: prefic

      bowtie2 -p 8 -x Round1 -1 B04_S13_R1_001.fastq -2 B04_S13_R2_001.fastq -S Round1.sam

* -p: number of threads
* -x: prefix
* -1: illumina forward data
* -2: illumina reverse data
* -S: output file (sam file)
    
      samtools sort -@ 8 Round1.sam -o Round1.sorted.bam

* sort: sort alignment file
* -@: number of threads
* -o: output file (bam file)
    
      samtools index -@ 8 Round1.sorted.bam

* index: indexing alignmnet file
* -@: number of threads
* Round1.sorted.bam: input file (sorted bam file)

      pilon --genome circularized.fasta --frags Round1.sorted.bam --output Round1

* --genome: input file (genome file)
* --frags: input file (indexed and sorted bam file)
* --output: output file 

## 5.Bacterial genome annotation
Prokka 설치를 위해서 anaconda를 먼저 설치해주어야함.

Prokka link: https://github.com/tseemann/prokka

### 5.1 Prokka install

    conda install -c conda-forge -c bioconda -c defaults prokka
    
### 5.2 Prokka DB setup

    prokka --setupdb
    
### 5.3 Annotation by Prokka

    prokka -outdir ./Prokka_output 06.fixstart.fasta --cpus 8

* --outdir: output directory
* 06.fixstart.fasta: circularised genome sequence
* --cpus: number of threads
