================================================ 
Quality Trimming
================================================

During this lab, we will acquaint ourselves with the software packages
Trimmomatic, khmer and Jellyfish. Your objectives are:

1. Familiarize yourself with software, how to execute it and optionally how to
   visualize results.
2. Characterize sequence quality.

The Trimmomatoc manual: http://www.usadellab.org/cms/?page=trimmomatic

The JellyFish manual: http://www.genome.umd.edu/jellyfish.html

The Khmer webpage: http://khmer.readthedocs.org/en/v2.0/

--------------

**Download data**: For this lab, we'll be using data from `Efficient de novo assembly of single-cell
bacterial genomes from short-read data sets, Chitsaz et al., 2011
<http://www.ncbi.nlm.nih.gov/pubmed/21926975>`__.

::

   mkdir Assembly
   mkdir Assembly/reads 
   cd /Assembly/reads/
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

**Eliminate illumina adapters from your sequences**

::
	
   cd ../
   mkdir trimming
   cd trimming

   #paste the below lines together as 1 command

   trimmomatic PE \
   -threads 8 \
   ../reads/ecoli_ref-5m_s1.fq \
   ../reads/ecoli_ref-5m_s2.fq \
   s1_pe s1_se s2_pe s2_se \
   ILLUMINACLIP:../reads/illuminaClipping.fa:2:30:10 

--------------

**Look at data quality using FastQC**:

   mkdir ../fastqc
   fastqc s1_* s2_* ../fastqc 

Go check it out on your local computer

::

   scp -r your_username@IP_address:/Volumes/Pegasus/your_home_folder/Assembly/fastqc .
   
   #or transfer the fastqc folder using cyberduck


It looks like a lot of bad data is present after base 70, so let’s just trim all the sequences that way. Before we do that, we want to interleave the reads again:

::

   interleave-reads.py s1_pe s2_pe > combined.fq 
    

Now, let’s use the FASTX toolkit to trim off bases over 70, and eliminate low-quality sequences. We need to do this both for our combined/paired files and our remaining single-ended files:

::

   fastx_trimmer -Q33 -l 70 -i combined.fq | fastq_quality_filter -Q33 -q 30 -p 50 > combined-trim.fq

   fastx_trimmer -Q33 -l 70 -i s1_se | fastq_quality_filter -Q33 -q 30 -p 50 > s1_se.filt
    
    
Let’s run FastQC on things again:

::

   fastqc combined-trim.fq s1_se.filt -o ../fastqc
	
