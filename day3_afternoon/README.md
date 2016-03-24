# Day 3 Afternoon
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

## Klebsiella pneumoniae comparative genomic analysis 

To finish up the workshop we are going to go through the process of working up a complete dataset, from start to finish.  This set of genomes originated from a regional outbreak of bla-KPC carrying Klebsiella pneumoniae – one of the most concerning healthcare associated pathogens. 
The goal is to follow up on a previously published epidemiologic analysis (http://cid.oxfordjournals.org/content/53/6/532.abstract), and see if genomics supports prior epidemiologic conclusions and can provide additional insights. We have our genomes, and we know in which regional facility each isolate originated. 

The goal of this exercise is to:
1) process our genomes (QC, variant calling), 
2) perform a phylogenetic analysis and 
3) overlay our meta-data. 

To make this more difficult, the instructions will be much more vague than in previous sessions, and you will be challenged to use what you have learned, both in the past three days and in the prior workshop, to complete this analysis. Hopefully we’ve prepared you to take on the challenge, but remember this is an open book test! 

Feel free to lean on materials from the workshops, manuals of tools and Google (and of course instructors and neighbors). 

Execute the following command to copy files for this afternoon’s exercises to your scratch directory:

```
cd /scratch/micro612w16_fluxod/username
cp –r  /scratch/micro612w16_fluxod/shared/data/day3_after .
```

## Perform QC on fastq files
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

On the first morning you ran FastQC to evaluate the quality of a single genome. However, a typical project will include many genomes and you will want to check the quality of all of your samples. From the bash workshop, I hope you can appreciate that you do not want to process 100 genomes by typing 100 commands – rather you want to write a short shell script to do the work for you!


>i. Write a shell script to run FastQC on all fastq files

The fastq files are located in:

```
/scratch/micro612w16_fluxod/shared/data/day3_after_fastq/
```

Rather than copying these to your directory, analyze the files in that directory directly, so everyone doesn’t have to copy 25G to their home directories. 

**HINTS** 
- Your shell script will include a for loop that loops over all of the genomes in the target directory
- The tricky part of this exercise is that each fastq command contains two files (forward and reverse reads). So, you need to take advantage of the fact that the forward and reverse read files both have the same prefix, and you can loop over these prefixes. 
- You should be able to get prefixes by piping the following unix commands: ls, cut, sort, uniq

>ii. Examine output of FastQC to verify that all samples are OK

## Examine results of SPANDx pipeline
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

On the afternoon of day 1 we saw how many steps are involved in calling variants relative to a reference genome. However, the same steps are applied to every sample, which makes this very pipeline friendly!  So, you could write your own shell script to string together these commands, or take advantage of one of several published pipelines. Here, we will use the output of the SPANDx pipeline, which takes as input a directory of fastq files and produces core variant and indel calls.

Because it takes a while to run, we have pre-run it for you. Your task will be to sort through the outputs of SPANDx.

>i. Look at overall statistics for variant calling in excel

SPANDx produces an overall summary file of its run that includes:
1) numbers of SNPs/indels, 
2) numbers of filtered SNPs/indels and 
3) average coverage across the reference genome. 

This summary file is in:  Outputs/Single_sample_summary.txt

Use less to look at this file and then apply unix commands to extract and sort individual columns 

**HINTS**
The following unix commands can be used to get sorted lists of coverage and numbers of SNPs/indels: tail, cut, sort

>ii. Look at filtered variants produced by SPANDx in excel

SPANDx also produces a summary file of the variants/indels it identified in the core genome. 

This summary file is: Outputs/Comparative/All_SNPs_annotated.txt 

Use sftp to download this file and view in excel

- View SPANDx manual for interpretation of different columns which can be found [here](https://github.com/dsarov/SPANDx/blob/master/SPANDx%20Manual _v3.1.pdf)
- Back on Flux, use grep to pull SNPs that have HIGH impact
- What types of mutations are predicted to have “HIGH” impact?
- How many genomes do these HIGH impact mutations tend to be present in? How do you interpret this?

## Recombination detection and tree generation
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

>i. Plot the distribution of variants across the genome in R

The positions of variants are embedded in the first column of Outputs/Comparative/All_SNPs_annotated.txt, but you have to do some work to isolate them! 

**HINTS**  
- You will need to pipe together two “cut” commands: the first command will use tab as a delimiter and the second will use _. 
- Note that for cut you can specify tab as the delimiter as follows: cut –d$’\t’ and _ as: cut -d ‘_’
- You should redirect the output of your cut commands (a list of SNP positions) to a file called ‘snp_positions.txt’.
- Finally, download this file, read it into R using ‘read.table’ and use ‘hist’ to plot a histogram of the positions
- Do you observe clustering of variants that would be indicative of recombination?

>ii.  Create fasta file of variants from nexus file

SPANDx creates a file of core SNPs in a slightly odd format (transposed nexus). 
This file is called: Outputs/Comparative/Ortho_SNP_matrix.nex

For convenience, apply the custom perl script located in the same directory to convert it to fasta format

```
perl transpose_nex_to_fasta.pl Ortho_SNP_matrix.nex
```

This file Outputs/Comparative/Ortho_SNP_matrix.fasta should now exist

>iii. Create maximum likelihood tree in Seaview

```
Download Ortho_SNP_matrix.fasta to your home computer
Import the file into Seaview and construct a tree using PhyML (100 bootstraps)
Save tree for later analysis
```

## Phylogenetic tree annotation and visualization
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

>i. Load the maximum likelihood tree into iTOL

Note that because the out-group is so distantly related it is difficult to make out the structure of the rest of the tree. To remedy this: 
- Click on the KPNIH1 leaf, go to the “tree structure” menu and “delete leaf” 
- Click on the extended branch leading to where KPNIH1 was, go to the “tree structure” menu and click “collapse branch”

>ii. Load the annotation file ‘Rush_KPC_facility_codes_iTOL.txt’ to view the facility of isolation  Play with tree visualization properties to understand how isolates group by facility o Circular vs. normal tree layout o Bootstrap values o Ignoring branch lengths

```
Which facilities appear to have a lot of intra-facility transmission based on grouping of isolates from the same facility? 
Which patient’s infections might have originated from the blue facility?
```

## Assessment of genomic deletions
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

>i. Download genome coverage bed file and load into R

This file is located in: Outputs/Comparative/Bedcov_merge.txt
This file contains information regarding locations in the reference genome that each sequenced genome does and does not map to.
The first 3 columns of the file are:
1) the name of the reference, 
2) the start coordinate of the window and 
3) the end coordinate of the window

The remaining columns are your analyzed genomes, with the values indicating the fraction of the window covered by reads in that genome.

In essence, this file contains information on parts of the reference genome that might have been deleted in one of our sequenced genomes.

After you download this file, read it into R

**HINTS**
- Use the read.table function with the relevant parameters being: header and sep

>ii. Plot heatmap of genome coverage bed file

**HINTS**
- The first 3 columns of the bed file specify the name of the chromosome and the genome coordinates – therefore you want to subset your matrix to not include these columns 
- Use the heatmap3 function to make your heatmap with the following parameters: scale = “none” (keeps original values), Rowv = NA (suppress clustering by rows – why might we not want to cluster by rows for this analysis?)

Note a large genomic deletion among a subset of isolates. Does this deletion fit with the phylogeny from above?

iii. Explore genomic deletion in more detail with ACT

- Use abacus to orient contigs from Rush_KPC_298 to KPNIH 
- Load KPNIH.gb, Rush_KPC_298_ordered and the .crunch alignment into ACT

```
What genes appear to have been lost?
```