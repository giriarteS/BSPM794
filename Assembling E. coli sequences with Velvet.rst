
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

Running an assembly with velvet
===============================

Note that a maximum k-mer of 41 will be sufficient for this exercise, but longer k-mers are required when working with longer HiSeq and MiSeq generated reads (which are now typically >100 bp). Now... assemble the small, fast data sets, using the Velvet assembler.  Herewe will set the required parameter k=21::

   cd Assembly
   velveth ecoli.21 21 -shortPaired -fastq diginorm/ecoli_ref-5m.D.fq 

This will take ~1-2 minutes and will produce a hash table of the reads using the specified k-mer length (k=21), saving them to the folder ‘ecoli.21’. The –shortPaired and fastq tag tells Velvet we are supplying short, paired end interleaved reads in fastq format. See manual for other input options. The next Velvet step to run is velvetg to build the graph. Enter:

   velvetg ecoli.21 -clean yes -exp_cov 5 -cov_cutoff 1.89 -min_contig_lgth 200

This will take ~5 minutes. Running this command will output a number of files to the same folder as velveth, including the file containing our newly assembled contigs – this will be labelled ‘contigs.fa’. Also try assembling with k=23 and k=25::

   velveth ecoli.23 23 -shortPaired -fastq diginorm/ecoli_ref-5m.D.fq 
   velvetg ecoli.23 -clean yes -exp_cov 5 -cov_cutoff 1.89 -min_contig_lgth 200

   velveth ecoli.25 25 -shortPaired -fastq diginorm/ecoli_ref-5m.D.fq 
   velvetg ecoli.25 -clean yes -exp_cov 5 -cov_cutoff 1.89 -min_contig_lgth 200

Using VelvetOptimiser to optimise de novo assembly with Velvet
==============================================================

With the settings bwllow, VelvetOptimiser will set up a series of velveth runs using odd-number kmers between 21 and 41. It then runs velvetg for each, taking the one with the best N50 as the seed for the final optimisation of the coverage cutoff, where the number of bases in contigs longer than 100bp is used as the optimising statistic. The output is the same as for a regular Velvet run, though the output folder will have the prefix ‘SRR292770’ to keep it separate from the earlier Velvet run described above.

Find the best assembly for Illumina paired end reads, trying k-values between 21 and 41::

   VelvetOptimiser.pl -s 21 -e 41 -f '-shortPaired -fastq diginorm/ecoli_ref-5m.D.fq' --o '-min_contig_lgth 200' -p ecoli

Print an estimate of how much RAM is needed by the above command, if we use eight threads at once,
and we estimate our assembled genome to be 4.5 megabases long::

   VelvetOptimiser.pl -s 21 -e 41 -f '-shortPaired -fastq diginorm/ecoli_ref-5m.D.fq' -g 4.5 -t 8

Now check out the stats for the assembled contigs for a cutoff of 1000 (minimum contig length = 1000)::

   /Users/BrodersLab/khmerEnv/sandbox/assemstats3.py 1000 ecoli.*/contigs.fa
  
Comparing and evaluating assemblies - QUAST
===========================================

Download the true reference genome::

   curl -O https://s3.amazonaws.com/public.ged.msu.edu/ecoliMG1655.fa.gz
   gunzip ecoliMG1655.fa.gz

and run QUAST::   

   quast.py -R ecoliMG1655.fa ecoli.*/contigs.fa
   
Note that here we're looking at *all* the assemblies we've generated.

Now look at the results. A description of the output can be found on the `QUAST web site <http://quast.bioinf.spbau.ru/manual.html>`__.

   more quast_results/latest/report.txt

How many large / small misassemblies were made by the assembler?

What is the actual genome coverage?


Searching assemblies -- BLAST
=============================

Build BLAST databases for the assemblies you've done::

   for i in 21 23 25 29
   do
      extract-long-sequences.py -o ecoli-$i.fa -l 1000 ecoli.$i/contigs.fa
      formatdb -i ecoli-$i.fa -o T -p F
   done

and then let's search for a specific gene -- first, download a file containing
your protein sequence of interest::

   wget http://athyra.idyll.org/~t/crp.fa

and now search::

   blastall -i crp.fa -d ecoli-21.fa -p tblastn -b 1 -v 1
   
Ordering contigs against a reference using Mauve
================================================

Once the sequence reads have been assembled into contigs, it is useful to order them against a suitable reference genome. One simple way to accomplish this is to use the ‘Move Contigs’ option available in `Mauve <http://asap.ahabs.wisc.edu/mauve/>`__.

Once you have installed Mauve and located your reference genome and contigs, we can order the contigs.

1. Launch the Mauve application.

2. From the Tools menu, select ‘Move Contigs’.

3. A dialogue box should appear, with a box labelled ‘Choose location to keep output files and folders’. Navigate to the folder with the sequences and the copied contigs, then click the ‘Create New Folder’ radio button. Give this folder a suitable name, e.g. ‘MauveOutput’ and then hit ‘OK’.

4. A message should appear telling you about the iterative process involved in reordering the contigs. Take note of it, then hit ‘OK’ to dismiss it.

5. A dialogue box should appear, with a box labelled ‘Align and Reorder Contigs’. Click the button below the box ‘Add Sequence...’ and navigate to the reference genome to align against, in this case ‘ecoliMG1655.fa’.

6. Click the ‘Add Sequence...’ button again and navigate to the fasta file of the contigs you wish to align, ‘contigs.fa’ from the assembly exercise above (ecoli.29). Check that you have put the reference genome first, and the draft second, as expected by Mauve.

7. Click ‘Start’ to run the reordering. A new window should appear marked ‘Mauve Console’ where the progress of the run will be displayed, including any error messages (see below for an example). A new window of the visualization tool should launch for each completed iteration, marked ‘Mauve unknown – alignmentX’, where X is the iteration number. If you encounter errors, check that you have specified the right files for input – they should be fasta or multi-fasta sequence files.

8. Finally, a message telling you the reorder is completed should appear. Hit ‘OK’ and quit Mauve – though you can inspect the final alignment (and the others) beforehand.

9. The final set of ordered and oriented contigs are in the fasta file located in the last of the iterated alignments. To find it, look in the ‘MauveOutput’ folder created above. For each iteration of the reordering there will be an output folder, so the final output is the contig file located in the subdirectory ‘alignmentX’ with the highest X, where X is the iteration number. Rename ‘contigs.fa’ in this subdirectory, to ‘ecoli.29.ordered.M.fasta’ and copy it to your main working directory (i.e. the one with the original sequence files, make sure you have changed the name of the ordered contigs file first as we will use the unordered contigs in a later exercise, e.g. ‘contigs.fa’. You can then delete the ‘alignmentX’ folders.

Ordering contigs against a reference using abacas
=================================================

Those who are used to Unix and sequence analysis may prefer to use a command-line based solution for ordering contigs. We recommend Abacas, which requires installation of MUMmer (http://mummer.sourceforge.net/), Perl and BioPerl.

   abacas.1.3.1.pl –r ecoliMG1655.fa -q ecoli_data_29/contigs.fa –p ‘nucmer’ –c –m –b –o ecoli.29.ordered.A.fasta

Using either method, you should end up with a set of contigs ordered against the reference strain in multi-fasta format in a file called ‘ecoli.29.ordered.A.fasta’. This is the file to use for the following steps.
