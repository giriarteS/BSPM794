=========================================
A tutorial in basic digital normalization
=========================================

**Three-pass diginorm for microbial genomic DNA**

First, do a first round of digital normalization to C=20

::

   mkdir Assembly/diginorm
   cd Assembly/diginorm
   /Users/BrodersLab/khmerEnv/bin/normalize-by-median.py -C 20 -k 20 -N 4 -x 2.5e8 -p --savegraph ecoli_ref.kh -o ecoli_ref.fq.keep ../trimming/ecoli_ref-5m.trim.fq
 
(wait a while...) ...this should eliminate about 2/3 of the data.
 
Next, trim low-abundance k-mers

::

   /Users/BrodersLab/khmerEnv/bin/filter-abund.py ecoli_ref.kh ecoli_ref.fq.keep -o ecoli_ref.fq.keep.abundfilt
   
(wait a while...) ...this should get rid of another 1% or so.

Finally, do the second round of normalization to C=5

::

   /Users/BrodersLab/khmerEnv/bin/normalize-by-median.py -C 5 -k 20 -N 4 -x 1e8 -o ecoli_ref-5m.D.fq ecoli_ref.fq.keep.abundfilt

(wait a while...) ...this will get rid of another 70%, around 400,000 sequences of the original 5m.

And voila – the sequences to assemble are in ‘ecoli_ref-5m.D.fq’, in FASTA format. See Short Read Assembly with Velvet.


**Run jellyfish on normalized data**

::

   jellyfish count -m 25 -s 200M -t 8 -C -o ../jelly/ecoli_ref.D.jf ecoli_ref-5m.D.fq
   cd ../jelly
   jellyfish histo ecoli_ref.D.jf -o ecoli_ref.D.histo


**OPEN RSTUDIO**: Import and visualize the histogram dataset on your computer.

::

Import all 3 histogram datasets: raw, trimmed and normalized

Plot: Make sure and change the names to match what you import.

What does this plot show you?? 

::

   barplot(c(ecoli_raw$V2[1],ecoli_trim$V2[1],ecoli_norm$V2[1]),
   names=c('raw', 'trim', 'norm'),
   main='Number of unique kmers')


Plot differences between non-unique kmers

::

   plot(ecoli_raw$V2[2:300] - ecoli_trim$V2[2:300], type='l',
   xlim=c(2,300), xaxs="i", yaxs="i", frame.plot=F,
   ylim=c(60000,65000), col='red', xlab='kmer frequency',
   lwd=4, ylab='count',
   main='Diff in 25mer counts of freq 2 to 300 \n raw vs. trimmed')
