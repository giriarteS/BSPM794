=======================================
K-mer analysis and genome size estimate
=======================================

Genome size can be calculated by counting k-mer frequency of the read data. The k should be sufficiently large that most of the genome can be distinguished. For most eukaryotic genomes at least 17 are usually used and calculation upto 31 is easiliy doable with Jellyfish.

**Outline**

1. count k-mer occurence using Jellyfish (jellyfish count)

2. summarize as histogram (jellyfish histo)

3. plot graph with R

4. determine the total number of k-mer analyzed and the peak position

5. compare the peak shape with poisson distribution


The JellyFish manual: http://www.genome.umd.edu/jellyfish.html

--------------

**Run Jellyfish** 

Count k-mer occurence

::

  cd
  mkdir jelly
  cd jelly
  jellyfish count -m 25 -s 200M -t 8 -C -o ecoli_raw.jf <(gzcat ../reads/ecoli_ref-5m.fastq.gz)


-t 8      
      specifies the number of threads to be used. This value should be equal to the number of cores on the machine.

`-C`        
      specifies the both strands are considered. If you do not specify this, the apparent depth would be half, that is undesirable

-m 25     
      specified that now you are counting for 25 mer (i.e., k=25)

-s 200M   
      is some kind of magical number specification of hash size. This should be as high as the physical memory allows. The higher the faster, but exceeding the available memory leads to failure or extremely slow counting.

-o ecoli_raw.jf  
      output file name.


Summarize as histogram

::

   jellyfish histo ecoli_raw.jf -o ecoli_raw.histo

   
Confirm that you got the output

::

   ls *.histo

Check the histograms

::

   head *.histo
   
   
Open up a new terminal window using the buttons command-t

::

   scp -r your_username@IP_address:/Volumes/Pegasus/your_home_folder/Assembly/jelly/*histo .
   
   #or transfer the histograms using cyberduck 
   
--------------  

**OPEN RSTUDIO**: Import and visualize the histogram dataset on your computer.

::

   ecoli_raw <- read.table("~/Desktop/ecoli_raw.histo", quote="\"")
   
   #Plot: Make sure and change the names to match what you import.
   
   
This will plot from 5 th line in the histo file. In general, the very low frequency k-mer are extremely high number that would make the scale for the y-axis too large. For x values, selection of the higher bound should be determined in accordance with read depth. The data contains ~1000. However if you plot them, it would be too large. Thus some good number should be chosen.

::

   plot(ecoli_raw[1:1026,],type="l")
   plot(ecoli_raw[10:200,],type="l")
   
**Determine the total number of k-mer analyzed and the peak position**

::

First, you should determine the border between the peak corresponding to single copy region and the initial peak corresponding to sequence errors. You might plot points on the line so that the region would be obvious   

::

   points(ecoli_raw[10:200,])
   
Now, we would calculate the total number of k-mer in the distribution

::

   sum(as.numeric(ecoli_raw[20:1026,1]*ecoli_raw[20:1026,2]))

   #[1] 327958644

Next, we want to know the peak position. From the graph, we can see its close to 70. Thus we examine the number close to 70 and find the maximum value

::

   plot(ecoli_raw[20:130,],type="l")
   points(ecoli[20:130,])


In this example, the peak is at 72. Then, the genome size can be estimated as:

::

   sum(as.numeric(ecoli_raw[20:1026,1]*ecoli_raw[20:1026,2]))/72
   
   #[1] 4554981  ~  4.5 Mbp
   
   
The size of single copy region can be roughly calculated as

::

   sum(as.numeric(ecoli_raw[20:130,1]*ecoli[20:130,2]))/72
   
   #[1] 4443047

It is 4.4 Mb or 0.97% of the genome. The proportion could be calculated as:  

::

  (sum(as.numeric(ecoli_raw[20:130,1]*ecoli_raw[20:130,2]))/sum(as.numeric(ecoli_raw[20:1026,1]*ecoli_raw[20:1026,2])))
  
  #[1] 0.975426
  

**Compare the peak shape with poisson distribution**

::

Now that we have some nice curve, we could compare it to ideal curve as poisson distribution scaled to the estimated single copy region size

::

   singleC <- sum(as.numeric(ecoli[20:130,1]*ecoli[20:130,2]))/72
   plot(1:200,dpois(1:200, 72)*singleC, type = "l", col=3, lty=2)
   lines(ecoli_raw[1:200,],type="l")
