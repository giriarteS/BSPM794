========================================
Bacterial genome annotation using Prokka
========================================

Prokka is a pipeline script which coordinates a series of genome feature predictor tools and sequence similarity tools to annotate the genome sequence (contigs).

**Run prokka on the contigs**

::

   cd Assembly
   mkdir Prokka
   cd Prokka
   prokka --outdir anno --prefix prokka ../Ecoli.fasta
   cat ./anno/prokka.txt
   
How many genes did Prokka find in the contigs?

Copy the anno/prokka.gff file to your latop using the scp command or cyberduck.

**Install Artemis**

`Artemis <http://www.sanger.ac.uk/science/tools/artemis>`__ is a graphical Java program to browse annotated genomes. It is a a bit like IGV but sepcifically designed for bacteria. You will need to install this on your laptop computer instead of the death-star.

**Load the annotated genome**

1. Start Artemis

2. Click OK

3. Go to File -> Open File Manager

4. Navigate to the ~/Downloads folder

5. Choose the prokka.gff file you copied from the death-star

**Browse the genome**

You will be overwhelmed and/or confused at first, and possibly permanently. Here are some tips:

1. There are 3 panels: feature map (top), sequence (middle), feature list (bottom)

2. Click right-mouse-button on bottom panel and select Show products

3. Zooming is done via the vertical scroll bars in the two top panels
