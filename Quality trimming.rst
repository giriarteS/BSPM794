================================================ 
Quality Trimming
================================================

During this lab, we will acquaint ourselves with the software packages
Trimmomatic, khmer and FastQC. Your objectives are:

1. Familiarize yourself with software, how to execute it and optionally how to
   visualize results.
2. Characterize sequence quality.

The cutadap manual: http://cutadapt.readthedocs.io/en/stable/index.html

The Khmer webpage: http://khmer.readthedocs.org/en/v2.0/

The FastQC webpage: http://www.bioinformatics.babraham.ac.uk/projects/fastqc/

--------------

**Download data**: For this lab, we'll be using data from `Efficient de novo assembly of single-cell
bacterial genomes from short-read data sets, Chitsaz et al., 2011
<http://www.ncbi.nlm.nih.gov/pubmed/21926975>`__.

::

   mkdir Assembly
   mkdir Assembly/reads 
   cd /Assembly/reads
   wget https://s3.amazonaws.com/public.ged.msu.edu/ecoli_ref-5m.fastq.gz
   gunzip -c ecoli_ref-5m.fastq.gz | less
   
   #(use ‘q’ to quit the viewer). This is what raw FASTQ looks like!

Note that in this case we’ve given you the data interleaved, which means that paired ends appear next to each other in the file. Most of the time sequencing facilities will give you data that is split out into s1 and s2 files. We’ll need to split it out into these files for some of the trimming steps, so let’s do that 

::  
   
   /Users/BrodersLab/khmerEnv/bin/split-paired-reads.py ecoli_ref-5m.fastq.gz
   mv ecoli_ref-5m.fastq.gz.1 ecoli_ref-5m_s1.fq
   mv ecoli_ref-5m.fastq.gz.2 ecoli_ref-5m_s2.fq
   less ecoli_ref-5m_s1.fq

This uses the khmer script ‘split-paired-reads’ to break the reads into left (/1) and right (/2). (This takes a long time! 5m reads is a lot of data...). We’ll also need to get some Illumina adapter information

::

   wget https://s3.amazonaws.com/public.ged.msu.edu/illuminaClipping.fa
   less illuminaClipping.fa
	
--------------

**Testing your knowledge**

There are two FASTQ files for this one lane of paired-end Illumina data: one file for the forward reads and one file for the reverse reads. Look at the first few reads.

::

   head ecoli_ref-5m_s1.fq 
   
   
How long are the reads? (hint: use awk '{print length}')

::

   awk '{if(NR%4==2) print length}' ecoli_ref-5m_s1.fq | gsort | guniq -c
   # 2500000 100

How many lines are there in both files? (hint: use wc -l)

::

   cat *.fq | wc -l
   # 20,000,000 lines
   
How many lines per read?

::

   4 lines per read
   

How many reads are there in both files?

::

   20000000 / 4 = 5,000,000 reads
   

How many bases are sequenced?

::

   5000000 * 100 = 500,000,000 bp
   

BONUS!! Assuming the genome is 4.5 Mbp, what is the depth of coverage?

::

   500000000 / 4500000 = 111.111 fold coverage


--------------

**Eliminate illumina adapters from your sequences**


::
	
   cd ../
   mkdir trimming
   cd trimming

Split reads

::

   #paste the below lines together as 1 command
   
   cutadapt \
            --max-n 0 --minimum-length 50 -q 3,3 \
            -a GATCGGAAGAGCGGTTCAGCAGGAATGCCGAG \
            -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
            -o ecoli_ref-5m.1.trim.fq -p ecoli_ref-5m.2.trim.fq \
            ../reads/ecoli_ref-5m_s1.fq ../reads/ecoli_ref-5m_s2.fq

Interleaved reads

::

   cutadapt \
           --max-n 0 --minimum-length 50 -q 3 \
           -a GATCGGAAGAGCGGTTCAGCAGGAATGCCGAG \
           -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
           -o ecoli_ref-5m.trim.fq ../reads/ecoli_ref-5m.fastq.gz
	   
--------------

**Look at data quality using FastQC**:

::

   mkdir ../fastqc
   fastqc ../reads/*.fq -o ../fastqc
   fastqc *.fq -o ../fastqc
   

Go check it out on your computer. Open up a new terminal window using the buttons command-t. What does the FastQC report tell you? 

::

   scp -r your_username@IP_address:/Volumes/Pegasus/your_home_folder/Assembly/fastqc .
   
   #or transfer the fastqc folder using cyberduck


Now, we want to interleave the reads again:

::

   /Users/BrodersLab/khmerEnv/bin/interleave-reads.py s1_pe s2_pe > combined.fq 
    
	
