
========================================
Assembling E. coli sequences with Velvet
========================================

The goal of this tutorial is to show you the basics of assembly using
`the Velvet assembler
<http://en.wikipedia.org/wiki/Velvet_assembler>`__.

We'll be using data from `Efficient de novo assembly of single-cell
bacterial genomes from short-read data sets, Chitsaz et al., 2011
<http://www.ncbi.nlm.nih.gov/pubmed/21926975>`__.

Getting the data
================

The first data set (ecoli_ref-5m-trim.fq) is the trimmed PE data sets from the document `Quality trimming`, and the second
data set is a specially processed data set using `digital normalization` (ecoli_ref-5m.D.fq) that will assemble quickly. ::

Running an assembly
===================

Now... assemble the small, fast data sets, using the Velvet assembler.  Here
we will set the required parameter k=21::

   cd Assembly
   velveth ecoli.21 21 -shortPaired -fastq diginorm/ecoli_ref-5m.D.fq
   velvetg ecoli.21 -exp_cov auto

Also try assembling with k=23 and k=25::

   velveth ecoli.23 23 -shortPaired -fastq diginorm/ecoli_ref-5m.D.fq
   velvetg ecoli.23 -exp_cov auto

   velveth ecoli.25 25 -shortPaired -fastq diginorm/ecoli_ref-5m.D.fq
   velvetg ecoli.25 -exp_cov auto

Find the best assembly for Illumina paired end reads, trying k-values between 21 and 31::

VelvetOptimiser.pl -s 21 -e 31 -f '-shortPaired -fastq diginorm/ecoli_ref-5m.D.fq'

Print an estimate of how much RAM is needed by the above command, if we use eight threads at once,
and we estimate our assembled genome to be 4.5 megabases long::

VelvetOptimiser.pl -s 21 -e 31 -f '-shortPaired -fastq diginorm/ecoli_ref-5m.D.fq' -g 4.5 -t 8

Find the best assembly for Illumina paired end reads just for k=31, using four threads, 
but optimizing for N50 for k-mer length rather than sum of large contig sizes::

VelvetOptimiser.pl -s 29 -e 29 -f '-shortPaired -fastq diginorm/ecoli_ref-5m.D.fq' -t 4 --optFuncKmer 'n50'

Now check out the stats for the assembled contigs for a cutoff of 1000 (minimum contig length = 1000)::

   /Users/BrodersLab/khmerEnv/sandbox/assemstats3.py 1000 ecoli.*/contigs.fa
  
Comparing and evaluating assemblies - QUAST
===========================================

Download the true reference genome::

   curl -O https://s3.amazonaws.com/public.ged.msu.edu/ecoliMG1655.fa.gz
   gunzip ecoliMG1655.fa.gz

and run QUAST::   

   quast.py -R ecoliMG1655.fa ecoli.*/contigs.fa
   
