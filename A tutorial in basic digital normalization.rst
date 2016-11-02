=========================================
A tutorial in basic digital normalization
=========================================

**Three-pass diginorm for microbial genomic DNA**

First, do a first round of digital normalization to C=20

::

   mkdir Assembly/diginorm
   cd Assembly/diginorm
   /Users/BrodersLab/khmerEnv/bin/normalize-by-median.py -C 20 -k 20 -N 4 -x 2.5e8 --savehash ecoli_ref.kh ../reads/ecoli_ref_5m.fastq.gz
 
(wait a while...) ...this should eliminate about 2/3 of the data.
 
Next, trim low-abundance k-mers

::

   /Users/BrodersLab/khmerEnv/bin/filter-abund.py ecoli_ref.kh ecoli_ref_5m.fq.gz.keep
   
(wait a while...) ...this should get rid of another 25% or so.

Finally, do the second round of normalization to C=5

::

   /Users/BrodersLab/khmerEnv/bin/normalize-by-median.py -C 5 -k 20 -N 4 -x 1e8 ecoli_ref.fq.gz.keep.abundfilt

(wait a while...) ...this will get rid of another 75%, leaving under 400,000 sequences of the original 5m.

And voila – the sequences to assemble are in ‘ecoli_ref.fq.gz.keep.abundfilt.keep’, in FASTA format. See Short Read Assembly with Velvet.

If you want to split into paired and orphan reads, do:
