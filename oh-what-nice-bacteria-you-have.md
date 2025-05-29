---
title: oh, what nice bacteria you have! a pipeline in R for metagenomic (16s rRNA)
  analysis
author: R package build
date: '2022-02-22'
slug: oh-what-nice-bacteria-you-have-a-pipeline-in-r-for-metagenomic-16s-rrna-analysis
categories:
  - R
  - GitHub
tags:
  - academia
---

# Motivation

In June of 2021, I moved to Salt Lake City, UT and started my postdoc in the [Werner Lab](https://sites.google.com/view/werner-lab/home?authuser=0) at the University of Utah. With a new city, a new position, and new PI comes exciting new projects, new study organisms, and new methods! Here I'll try to document my first foray into bioinformatics. I hope it will help someone along the way :)

# 16S-rRNA-analysis

The main purpose of this post is to show how to process 16S rRNA sequencing data with the DADA2 pipeline.

Here I write my notes on how I conducted metagenomic and 16S analysis of GSL samples from 2020.

Modified versions of the pipeline and scripts are detailed [here](https://github.com/tkarasov/pathodopsis) and [here](https://github.com/LangilleLab/microbiome_helper/wiki/DADA2-16S-Chemerin-Tutorial).

```{r session-info}
# Display current R session information
sessionInfo()
```

## Unix basics

You can find a nice basic tutorial [here](http://korflab.ucdavis.edu/bootcamp.html).

### Navigating files

#### bash

With macOS Catalina, Apple is now using Zsh as the default shell. To change a user's default shell on macOS, run:

```{bash, eval=FALSE, engine="sh"}
chsh -s /bin/bash
```

You'll have to enter your user account's password (cursor doesn't show up but don't worry about it). You have to refresh the terminal window to activate the change.

#### ls

```{bash, eval=FALSE, engine="sh"}
ls
```

This is the command prompt ls lists the contents of your directory

```{bash, eval=FALSE, engine="sh"}
ls /
```

This lists the contents of your root directory. first forward slash that appears in a list of directory names always refers to the top level directory of the file system (known as the root directory). The remaining forward slash (between 'home' and 'ubuntu') delimits the various parts of the directory hierarchy.

#### pwd

```{bash, eval=FALSE, engine="sh"}
pwd
```

Prints your working directory. Helps you find out where you are.

#### mkdir

```{bash, eval=FALSE, engine="sh"}
mkdir <new_directory_name>
```

Makes a new directory.

#### cd

```{bash, eval=FALSE, engine="sh"}
cd <new_directory_name>
```

Changes directory to the one you just made.

```{bash, eval=FALSE, engine="sh"}
cd /
```

Changes directory to the root directory. You can also change to home.

Note that the command prompt shows you the name of the directory that you are currently in. BUT when you are in your home directory it shows you a \~ instead. Unix uses the \~ character as a short-hand way of specifying a home directory.

```{bash, eval=FALSE, engine="sh"}
cd 
cd ~
```

Both commands jump straight back to your home directory.

```{bash, eval=FALSE, engine="sh"}
mkdir -p Outer_directory/Inner_directory
```

mkdir (like most Unix commands) supports command-line options. These are optional arguments that are placed after the command name. Often in the format of dash + a single letter. In this case, -p option allows you to create two directories in a single step.

#### rmdir

rmdir will remove empty directories! Note that you have to be outside a directory before you can remove it with rmdir

```{bash, eval=FALSE, engine="sh"}
cd ..
```

Navigates up 1 level.

#### ..

Two dots refers to the parent directory of wherever you are (all except the root has one).

```{bash, eval=FALSE, engine="sh"}
cd ../..
```

Navigates up 2 levels.

```{bash, eval=FALSE, engine="sh"}
cd ~/directory_name
```

Note that you can change directory relative to where we are now OR make absolute changes.

```{bash, eval=FALSE, engine="sh"}
ls ../../
```

lists the two directories that are above you.

#### ls options

```{bash, eval=FALSE, engine="sh"}
ls -l /home
```

will give you a longer output compared to the default, including file ownership and modification times). The 'd' at the start of each line indicates that these are directories. There are many options for ls.

```{bash, eval=FALSE, engine="sh"}
ls -l 
ls -R 
ls -l -t -r 
ls -lh
```

The last one combines multiple options only using one dash. This is a common way of specifying multiple command-line options.

```{bash, eval=FALSE, engine="sh"}
man ls 
man cd
man man # yes even the man command has a manual page
```

every Unix command has an associated 'manual' that you can access by using the man command. When you are using the man command, press space to scroll down a page, b to go back a page, or q to quit. You can also use the up and down arrows to scroll a line at a time.

HOT TIPS:

1.  You can tab complete the names of files and programs on most Unix systems.If pressing tab doesn't do anything, then you have not have typed enough unique characters. In this case pressing tab twice will show you all possible completions.

2.  Unix stores a list of all the commands that you have typed in each login session. You can access this list by using the history command or more simply by using the up and down arrows to access anything from your history.

### Working with files

#### touch

```{bash, eval=FALSE, engine="sh"}
touch heaven.txt
touch earth.txt
```

command touch lets us create a new, empty file.

```{bash, eval=FALSE, engine="sh"}
mkdir Temp
mv heaven.txt Temp/
ls
ls Temp/
```

For the mv command, we always have to specify a source file (or directory) that we want to move, and then specify a target location.

If we had wanted to we could have moved both files in one go by typing any of the following commands:

```{bash, eval=FALSE, engine="sh"}
mv *.txt Temp/ 
mv *t Temp/ 
mv *ea* Temp/
```

#### \*

The asterisk \* acts as a wild-card character, essentially meaning 'match anything'. The second example works because there are no other files or directories in the directory that end with the letters 't' (if there was, then they would be moved too). Likewise, the third example works because only those two files contain the letters 'ea' in their names. Using wild-card characters can save you a lot of typing.

```{bash, eval=FALSE, engine="sh"}
touch rags
mv rags Temp/riches 
mv Temp/riches Temp/rags
```

touch creates new file 'rags'. mv renames: ie. moves it to a new location while changing the name to 'riches'. We must use mv to rename a file without moving it (you have to use mv to do this as Unix does not have a separate 'rename' command).

```{bash, eval=FALSE, engine="sh"}
mv Temp/riches Temp/rags
mkdir Temp2
mv Temp2 Temp
ls Temp/
```

Moving directories is just like moving files.

```{bash, eval=FALSE, engine="sh"}
rm -i earth.txt heaven.text rags
```

Use rm with the -i command-line option, which will ask for comfirmation before deleting anything.

```{bash, eval=FALSE, engine="sh"}
touch file1
cp file1 file2
ls
```

Copying files with the cp (copy) command is very similar to moving them. Remember to always specify a source and a target location. Here we create a new file and make a copy of it.

Copy files from a different directory to our current directory:

```{bash, eval=FALSE, engine="sh"}
touch ~/file3 
    #puts a  file in our home directory (specified by ~)
cp ~/file3 . 
    # copies it to the current directory
```

#### .

The current directory can be represented by a . (dot) character. You will mostly use this only for copying files to the current directory that you are in.

```{bash, eval=FALSE, engine="sh"}
ls 
ls . 
ls ./
```

All these should have the same result.

You can also use the cp command to copy entire directories.

#### echo

```{bash, eval=FALSE, engine="sh"}
echo "Call me Ishmael."
```

Echo echoes text back to the screen (like print). We can redirect that text into an output file by using the \> symbol (file redirection).

#### \>

```{bash, eval=FALSE, engine="sh"}
echo "Call me Ishmael." > opening_lines.txt
```

Careful when using file redirection (\>), it will overwrite any existing file of the same name.

```{bash, eval=FALSE, engine="sh"}
less opening_lines.txt
```

#### less

Less lets you view (but not edit) text files. When you are using less, you can bring up a page of help commands by pressing h, scroll forward a page by pressing space, or go forward or backwards one line at a time by pressing j or k. To exit less, press q (for quit). The less program also does about a million other useful things (including text searching).

```{bash, eval=FALSE, engine="sh"}
echo "The primroses were over." >> opening_lines.txt
```

#### \>\>

Add another line to the file. \>\> (not just \>) will **append** to a file. If you only use \>, we would end up overwriting the file.

#### cat

```{bash, eval=FALSE, engine="sh"}
cat opening_lines.txt
```

The cat command displays the contents of the file(s) and then returns you to the command line. Unlike less, you have no control over how you view the text.

```{bash, eval=FALSE, engine="sh"}
cat opening_lines.txt > file_copy.txt
```

You can use cat to quickly combine multiple files or, if you wanted to, make a copy of an existing file.

```{bash, eval=FALSE, engine="sh"}
ls -l
```

shows us a long listing, which includes the size of the file in bytes

#### wc

```{bash, eval=FALSE, engine="sh"}
wc opening_lines.txt
```

By default this tells you how many lines, words, and characters are in a specified file(or files).

```{bash, eval=FALSE, engine="sh"}
wc -l opening_lines.txt
```

You can use command-line options to give you just one of those statistics (in this case we count lines with wc -l).

#### nano

Nano is a lightweight editor installed on most Unix systems. There are many more powerful editors (such as 'emacs' and 'vi'), but these have steep learning curves. Nano is very simple. You can edit (or create) files by typing:

```{bash, eval=FALSE, engine="sh"}
nano opening_lines.txt
```

The bottom of the nano window shows you a list of simple commands which are all accessible by typing 'Control' plus a letter. E.g. Control + X exits the program.

#### echo

```{bash, eval=FALSE, engine="sh"}
echo $USER
    #juliejung
echo $HOME
    #/Users/juliejung
echo $PATH
    #/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin
```

One other use of the echo command is for displaying the contents of something known as environment variables. These contain user-specific or system-wide values that either reflect simple pieces of information (your username), or lists of useful locations on the file system. Some examples above.

#### path

The last one shows the content of the \$PATH environment variable, which displays a --- colon separated --- list of directories that are expected to contain programs that you can run. This includes all of the Unix commands that you have seen so far. These are files that live in directories which are run like programs (e.g. ls is just a special type of file in the /bin directory).

Knowing how to change your \$PATH to include custom directories can be necessary sometimes (e.g. if you install some new bioinformatics software in a non-standard location).

#### grep

show lines that match a specified pattern ignore case when matching (-i) only match whole words (-w) show lines that don't match a pattern (-v) Use wildcard characters and other patterns to allow for alternatives (\*, ., and \[\])

```{bash, eval=FALSE, engine="sh"}
grep was opening_lines.txt
```

shows lines that have 'was' in them.

```{bash, eval=FALSE, engine="sh"}
grep -v was opening_lines.txt
```

show lines that don't have 'was' in them (-v)

```{bash, eval=FALSE, engine="sh"}
grep all opening_lines.txt
```

shows lines that have 'all' in them.

```{bash, eval=FALSE, engine="sh"}
grep -i all opening_lines.txt
```

ignore case when matching (-i)

```{bash, eval=FALSE, engine="sh"}
grep in opening_lines.txt
grep -w in opening_lines.txt
```

only match whole words (-w)

```{bash, eval=FALSE, engine="sh"}
grep -w o.. opening_lines.txt
grep [aeiou]t opening_lines.txt
grep -w -i [aeiou]t opening_lines.txt
```

#### pipes

One of the most poweful features of Unix is that you can send the output from one command or program to any other command (as long as the second commmand accepts input of some sort). We do this by using what is known as a pipe. This is implemented using the '\|' character (below delete key on a mac). Think of the pipe as simply connecting two Unix programs. Here's an example which introduces some new Unix commands:

```{bash, eval=FALSE, engine="sh"}
grep was opening_lines.txt | wc -c
```

searches the specified file for lines matching 'was', it sends the lines that match through a pipe to the wc program. We use the -c option to just count characters in the matching lines (316).

```{bash, eval=FALSE, engine="sh"}
grep was opening_lines.txt | sort | head -n 3 | wc -c
```

sends the output of grep to the Unix sort command. This sorts a file alphanumerically by default. The sorted output is sent to the head command which by default shows the first 10 lines of a file. We use the -n option of this command to only show 3 lines. These 3 lines are then sent to the wc command as before.

Whenever making a long pipe, test each step as you build it!

#### misc power commands

View the penultimate 10 (20?) lines of a file (using head and tail commands):

```{bash, eval=FALSE, engine="sh"}
tail -n 20 file.txt | head
```

Show lines of a file that begin with a start codon (ATG) (the ^ matches patterns at the start of a line):

```{bash, eval=FALSE, engine="sh"}
grep "^ATG" file.txt
```

Cut out the 3rd column of a tab-delimited text file and sort it to only show unique lines (i.e. remove duplicates):

```{bash, eval=FALSE, engine="sh"}
cut -f 3 file.txt | sort -u
```

Count how many lines in a file contain the words 'cat' or 'bat' (-c option of grep counts lines):

```{bash, eval=FALSE, engine="sh"}
grep -c '[bc]at' file.txt
```

Turn lower-case text into upper-case (using tr command to 'transliterate'):

```{bash, eval=FALSE, engine="sh"}
cat file.txt | tr 'a-z' 'A-Z'
```

Change all occurences of 'Chr1' to 'Chromosome 1' and write changed output to a new file (using sed command):

```{bash, eval=FALSE, engine="sh"}
cat file.txt | sed 's/Chr1/Chromosome 1/' > file2.txt
```

## About the dataset

This dataset consists a total of 15 samples generated from individual nematodes. The bacteria could be what the nematode ate or it could be bacteria found on the surface - we can't be sure.

Amplicon sequencing is a common method of identifying which taxa are present in a sample based on amplified marker genes. This approach contrasts with shotgun metagenomics where all the DNA in a sample is sequenced. The most common marker gene used for prokaryotes is the 16S ribosomal RNA gene. It features regions that are conserved among these organisms, as well as variable regions that allow distinction among organisms. These characteristics make this gene useful for analyzing microbial communities at reduced cost compared to shotgun metagenomics approaches. Only a subset of variable regions are generally sequenced for amplicon studies and you will see them referred to using syntax like "V3-V4" (i.e. variable regions 3 to 4) in scientific papers.

For our purposes, the region used for amplification and sequencing is the V4 region of the 16S rRNA gene. The primers used were 515F and 806R, creating an amplicon of around 250bp. Sequencing was performed in 2x300 bp paired-end mode on an Illumina MiSeq machine. This means the amplicon is basically covered twice by each read-pair, which is important to assure high quality 16S amplicon sequences.

## Prepare your raw data

### metadata

The metadata associated with each sample is indicated in the mapping file (map.txt). In this mapping file, the genotypes of interest can be seen. Although optional, it's generally a good idea to include as much metadata as possible, since this data can easily explored later on.

Typical mapping files include SampleID, BarcodeSequence, LinkerPrimerSequence, BarcodeName, Repeats, ProjectName, Description. All we really care about for demultiplexing are this first and second columns, the sample name and barcode.

The [barcodes](https://ipyrad.readthedocs.io/en/latest/5-demultiplexing.html) file is a simple table linking barcodes to samples. Barcodes can be of varying lengths. Each line should have one name and then one barcode, separated by whitespace (a tab or spaces).

### raw data

We'll start with raw data in the form of a bunch of files for each sample you sequenced. These files carry the raw sequence data in FastQ format.

The file names will look like this: samplename\_S70\_L001\_R1\_001.fastq.gz

The first part is the sample name. S70 tells you it is sample 70 of this specific sequencing run (can also be index sequences used in multiplexing). L001 is always lane 1 for the MiSeq. R1 means it's the forward read of the sequencing data (there should be another file that says R2, while is the read pair). 001 is always 001 for some reason. fastq.gz means that the file is in FastQ format and the file is compressed using the Gzip compression.

To look at what is in the file, first we need to navigate to the directory that's holding your raw\_reads, and decommpress each file (gunzip). For example, here let's pass the output the stdout (-c), pipe it to the next command, and display the first four lines in the file:

```{bash, eval=FALSE, engine="sh"}
gunzip -c 1_S1_L001_R1_001.fastq.gz | head -n 4
```

We get something that looks like this:

```{bash, eval=FALSE, engine="sh"}
@M06923:26:000000000-JV49B:1:1101:9099:1079 1:N:0:CGTCGGTAA
CTGAGTGTCAGCCGCCGCGGTAATACGGAGGGTGCAAGCGTTAATCGGAATTACTGGGCGTAAAGCGCGCGTAGGTGGCTTGATAAGCCGGTTGTGAAAGCCCCGGGCTCAACCTGGGAACGGCATCCGGAACTGTCAGGCTAGAGTGCAGGAGAGGAAGGTAGAATTCCCGGTGTAGCGGTGAAATGCGTAGAGATCGGGAGGAATACCAGTGGCGAAGGCGGCCTTCTGGACTGACACTGACACTGAGGTGCGAAAGCGTGGGTAGCAAACAGGATTAGATACCCTGGTAGTCCACGCC
+
CCCCCGGGGGGGGGGGGGGGGGGGGFGGCFGGGGGGGGGGGGGFGGGGGGGGGGGGGGGGGGD7ECEGGGGGGGGGGGGGGGGGFGGGGGGGGGGGGGGGGGFDEEGGEGGGGGGGGGGGGGGGGGGGGGGGGGGFFFF7FGGGGGGGGGGGGGGGGGCGGFFGGGGGGGGGGGGGGGGEGGGGFGFFFGCEGGGFFFEGGGEC8E?FGGGGGGGGGECGGGGGGG*CFGGGGGGGGD679?:*69<FFFF<>F>:>6>?DFGGFF)1:AFFF>?BF0<<?):9<F66><?)44:)6<?><
```

This output is the standard format of the FastQ sequencing data format. One sequence in FastQ format always consists of four lines:

-   Line 1: This line is the start of the entry and always starts with the \@ symbol, followed by a unique sequence ID. This ID is the same in both read files (R1+R2) and defines which sequences belong together. When looking at Illumina sequencing data, this ID consists of the unique instrument ID (M06923), the run ID (26), the flowcell ID (JV49B) and the lane (1), followed by coordinates of the sequence cluster on the flowcell. After the space there are more informations on the sequence, however these do not belong to the unique sequence ID. The last sequence is the barcode/index (CGTCGGTAA).

-   Line 2: This is the actual sequence obtained from basecalling of the raw data. In our case this is the 16S amplicon sequence.

-   Line 3: This line can contain additional information about the sequence, however usually only contains a mandatory ‚+' symbol.

-   Line 4: In this line the quality of every single base in the sequence is encoded. The first position of this line corresponds to the first position of the sequence, the second to the second, and so on. Each ASCII-character encodes a special value between 0 and 41 (in the most widely used Illumina 1.8 "Phred+33" format; +33 stands for an offset of 33 in the ASCII characters). The highest Phred score a base can get is in this case 41, encoded by the letter J (ASCII: 74), the lowest is 0, encoded by ! (ASCII: 33). The Phred score (Q) is defined as the probability (P), that the base at this position is incorrect: P(Q)=10-Q10. This corresponds to 0.01% error probability at Q=40 and 10% at Q=10.

Phred+33 encoding of quality scores:

```{bash, eval=FALSE, engine="sh"}
Symbol	 !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJ
        |                                        |          
ASCII	  33                                      73
Q Score	0.2......................26...31........41                      
```

Of course it is not possible to check each single sequence by 'hand', however there are a lot of very handy tools to check if the sequencing in general yielded satisfactory data. One tool we want to introduce is the very easy to use and freely available FastQC. Please go the [website](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) using the browser in the VirtualBox, download the Linux version of the tool and unpack it. For this, please create a folder in the "16S-rRNA-analysis" folder called "software", double-click on the downloaded archive and move the 'FastQC' folder into the newly created 'software' folder. Go to the terminal again and change to the FastQC software folder:

```{bash, eval=FALSE, engine="sh"}
cd /Users/juliejung/Desktop/Github/16S-rRNA-analysis/software/FastQC  
```

To use fastqc, we first have to make the binary file executable. For this type this into the terminal:

```{bash, eval=FALSE, engine="sh"}
chmod +x fastqc
```

The command chmod can be used to change the mode or access permission of a file. Here we add permission to execute (+x) to the fastqc binary. Now we can start FastQC:

```{bash, eval=FALSE, engine="sh"}
./fastqc
```

This should open the FastQC software. In the Menu bar go to: "File \> Open ..." and navigate to the raw\_data folder in Shared\_Folder. By holding the Shift key you can select multiple files at once. Please select the R1 and R2 read of the Sample "B-1105-W" and click the "OK" button. You will see the files being processed and quickly the reports should open, each as a tab in the FastQC window. On the left side you will see several green, orange and red symbols. These are the different quality criteria assessed by FastQC. However, let's start with the first one called "Basic statistic". Here you can already see some basic properties of the opened files, for example that each of the files holds 555172 sequences (as they are the paired reads of the same sample this should be the same; if it is not, there is something wrong), the sequence lengths are between 35 and 301 nt and the GC content differs slightly between R1 and R2, however both being around 50%.

Next please have a look at the "Per base sequence quality" tab for both files. What you see here is a pretty typical (though not perfect) example of how amplicon sequencing data normally looks like:

the R1 is relatively good quality all along, only dropping a little at the end of the read, however mostly staying in the green area until \~280 nt. the R2 is usually of worse quality than the R1. This is is basically what you can expect and mainly due to technical reasons. Only if you see really strange patterns of very drastic drops in quality (also maybe only at a single position across all samples) things might have to be investigated more deeply.

Move on to the tab called "Per base sequence content". This tab will likely be marked with a red cross, indicating a problem. When you look at the image in this tab, you can see the per-position composition of the four nucleotides and notice, that there is only very little variation in the first 8-9 nt. This is just as it is supposed to be, as the amplification primers have to bind in a non- or at least less-variable region of the 16S gene to work. As FastQC is intended to be used for any kind of sequencing data, it assumes that this is an error, however in our case, again, this is what we are expecting.

The same rules apply for GC content (of course there is a bias in GC content when only looking at a specific sequence) and duplicated/overrepresented sequences: we expect to have many duplicated or similar sequences, as this is exactly the purpose of using an amplicon.

In general we can say: our data look just like we expect it to look!

One more very convenient thing about FastQC is, that you can also use it for batch processing of FastQ file. Close the FastQC window and type:

```{bash, eval=FALSE, engine="sh"}
wosho=/Users/juliejung/Desktop/Github/16S-rRNA-analysis
echo $wosho 
raw=$wosho/data/raw_reads
echo $raw
./fastqc $raw/*_001* --outdir /Users/juliejung/Desktop/Github/16S-rRNA-analysis
```

In the first line, we set a variable named "wosho". This is very easy in the bash environment and might look a little different in other programming languages. In general, many things can be stored in variables, however, here we use it for a path to a folder. In the third line, we set another variable "raw", which has the same content as "wosho", plus "/raw\_reads" at the end. As you can see, the content of the variables can be show by using "echo" followed by the variable with a dollar sign prepended. The last line starts the fastqc binary, applied to all files in the folder that is stored in the "raw" variable that have "\_001" in the middle of the name. The asterisk "\*" is a so-called "wildcard" and matches every character when searching for filenames.

Now look at your output directory. For each input sample, there should be two html files you can open with your browser. These file hold all the information we saw before in the FastQC window and this way of processing can be easily applied to a whole batch of sequencing files for fast processing to get an idea if your data looks usable.

Another software that is very nice for getting an overview of sequencing data in general is [MultiQC](http://multiqc.info), which is able to aggregate the output of different QC tools to create reports. However, this is currently more aiming at RNAseq or RRBS experiments, but worth keeping an eye on.

## Demultiplex

For this part of the analysis, I used [this](https://astrobiomike.github.io/amplicon/demultiplexing) and [this](https://github.com/tkarasov/pathodopsis) and [this](http://protocols.faircloth-lab.org/en/latest/protocols-computer/sequencing/sequencing-demultiplex-a-run.html).

Several samples can be combined into a single sequencing run by using "multiplexing" where a barcode/index sequence identifying the sample is inserted into the sequencing construct. Once sequenced, the data generally need to be demultiplexed by their barcode/index sequences into something approximating the sample names that you want to associate the sequence data with.

**Demultiplexing** refers to the step in processing where you'd use the barcode information in order to know which sequences came from which samples after they had all been sequenced together. Here we would sort sequenced reads into separate files for each sample. If we received our data already demultiplexed, with a separate file for each sample, and barcodes clipped. With current Illumina software and standard library preparation protocols, the demultiplexing is usually done for you and the basespace download includes one FASTQ file for each sample; the index reads are not included.

**Barcodes** refer to the unique sequences that were ligated to each of your individual samples' genetic material before the samples got all mixed together. Depending, you may get your samples already split into individual fastq files, or they may be lumped together all in one fastq file with barcodes still attached for you to do the splitting. If this is the case, you should also have a "mapping" file telling you which barcodes correspond with which samples (see metadata section). With Illumina sequencing, the [barcode](https://drive5.com/usearch/manual/pipe_demux.html) is usually positioned before the sequencing primer so does not appear in the forward reads that contain the biological sequence. Barcodes are obtained by making one (single-indexing) or two (dual-indexing) additional reads which are sometimes called i1 for single indexing and i5+i7 for dual indexing.

After navigating into the directory with map.txt in it, you can view using:

```{bash, eval=FALSE, engine="sh"}
column -t map.txt | head  
```

**Indexes** There seems to be quite a bit of ambiguity out there with regard to barcodes vs [indexes.](https://kb.10xgenomics.com/hc/en-us/articles/115002777072-How-do-I-demultiplex-by-sample-index-and-barcode-) These terms are sometimes used [interchangeably](https://drive5.com/usearch/manual/pipe_demux.html) in the genomics world - for example, what Illumina's Sequencing Analysis Viewer refers to as barcodes, are what we call sample indices - but in the context of our products, they have distinct meanings. During library preparation, the barcoding steps are done before the indexing steps (final PCR step), and this order is reversed during the bioinformatics pipeline - we first need to demultiplex by sample index to separate reads into their respective libraries before dealing with the library-specific barcodes in subsequent steps.

[I](https://astrobiomike.github.io/amplicon/demultiplexing) think indexes refer to sample-identifying sequences that are sequenced separately from the primary forward and reverse reads of our target fragment, and this is why they come in separate fastq files but in the same order as your reads. I've only ever worked with amplicon data that had barcodes that were sequenced with the forward read, and therefore were in the forward read ahead of the primers. So that is what was exemplified here.

If you received Undetermined files from the sequence run, you can demultiplex those based on the index calls in the header of the sequence. For example:

    @A00484:41:H3G5VDRXX:1:1101:1018:1000 1:N:0:GGCGTTAT+CCTATTGG
                                                ^^^^ indexes ^^^^

So for the example that we ran above (sample 1), the index is CGTCGGTAA.

    @M06923:26:000000000-JV49B:1:1101:9099:1079 1:N:0:CGTCGGTAA
                                                      ^^^^ index 

## Examine Undetermined samples

Let's look closer into the Undetermined sample.

```{bash, eval=FALSE, engine="sh"}
gunzip -c Undetermined_S0_L001_R1_001.fastq.gz | head -n 4
```

    @M06923:26:000000000-JV49B:1:1101:15377:1000 1:N:0:NNNNNNNNN
    NTGAGTGTCAGCAGCCGCGGTAATACGTAGGGTGCAAGCGTTGTCCGGAATTATTGGGCGTAAAGAGCTCGTAGGCGGTGTGTCGCGTCTGCTGTGAAAATCCAAGGCTCAACCTTGGACCTGCAGTGGGTACGGGCACACTAGAGTGCGGTAGGGGAGATTGGAATTCCTGGTGT
    +
    #8BCCGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG

First, it's always good to get an idea of read counts for a given batch of samples. If you have all of your R1 and R2 files in a directory, you can use something like the following to count reads in each file:

```{bash, eval=FALSE, engine="sh"}
for i in *_R1_*; do echo $i; gunzip -c $i | wc -l; done 

10_S10_L001_R1_001.fastq.gz
 2220688
11_S11_L001_R1_001.fastq.gz
 2476064
12_S12_L001_R1_001.fastq.gz
 2347596
13_S13_L001_R1_001.fastq.gz
 2284768
14_S14_L001_R1_001.fastq.gz
 2592524
15_S15_L001_R1_001.fastq.gz
 2007384
16_S16_L001_R1_001.fastq.gz
 2366752
1_S1_L001_R1_001.fastq.gz
 2386132
2_S2_L001_R1_001.fastq.gz
 1892144
3_S3_L001_R1_001.fastq.gz
 2613432
4_S4_L001_R1_001.fastq.gz
 2020652
5_S5_L001_R1_001.fastq.gz
 2094560
6_S6_L001_R1_001.fastq.gz
     632
7_S7_L001_R1_001.fastq.gz
 2965096
8_S8_L001_R1_001.fastq.gz
 2484772
9_S9_L001_R1_001.fastq.gz
 2722520
Undetermined_S0_L001_R1_001.fastq.gz
 12519388
```

These are line counts, so be sure to divide these by 4 to get read counts. A pro-tip is that you can turn this into columns using the following find (.*)\\n(.*)\\n\* and replace \$1,\$2\\n commands work for your favorite text editor.

R2 should have same readcounts as R1

You want to compare this list to what you expect, being aware of samples that are either: (1) completely missing or (2) have very little data, like so:

```{bash, eval=FALSE, engine="sh"}
6_S6_L001_R1_001.fastq.gz
     632
```

These samples are likely some with incorrect indexes (we expected them to get lots of reads, but, in reality, they received very few).

Next take a peak into the undetermined file to get the sequencing machine name in the header line:

```{bash, eval=FALSE, engine="sh"}
gunzip -c Undetermined_S0_L001_R1_001.fastq.gz| head
#or
gunzip -c Undetermined_S0_L001_R1_001.fastq.gz| less
```

First, I [nucleotide BLASTED](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastSearch) the first line of nucleotides to see if best hits are plausible bacteria (one possibility is that we had some E.coli contamination or something). I'm getting that the best hit is:

Pontimonas salivibrio 313 313 97% 5e-81 99.42% 1760810 CP026923.1

So I think it's worth investigating further.

From our bash result, this was sequencer JV49B. So now, we can parse out all the indexes in the Undetermined\_S0\_L001\_R1\_001.fastq.gz file and count them to see if you can see what happened. Run the following:

```{bash, eval=FALSE, engine="sh"}
gunzip -c Undetermined_S0_L001_R1_001.fastq.gz| less
```

Then, enter "search mode" in less by typing /. I then generally search for the index from sample 6 (with low reads: CACTTCTGG) to see if something got mixed up.

It does seem like that is the case, bc in the search we're getting a few instances of that barcode coming up.

```{bash, eval=FALSE, engine="sh"}
gunzip -c  Undetermined_S0_L001_R1_001.fastq.gz | grep "CACTTCTGG" | awk -F: '{print $NF}' | sort | uniq -c | sort -nr > R1_barcode_count.txt
```

Let's take a look in the R1\_barcode\_count.txt we just created.

```{bash, eval=FALSE, engine="sh"}
less R1_barcode_count.txt
```

Then, enter "search mode" in less by typing /CACTTCTGG. This will highlight all the instances of that barcode.

So it seems that there was a mixup and the Undetermined sequences are actually the sample 6 (with super low read counts). This sample has already been demultiplexed, so we just still need to clip the barcodes out.

## DADA2 pipeline

[This](https://astrobiomike.github.io/amplicon/16S_and_18S_mixed) is a useful resource.

```{r installpackages, eval=FALSE}
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install(version = '3.14')
BiocManager::install("dada2")
BiocManager::install("ShortRead")
BiocManager::install("Biostrings")
BiocManager::install("DECIPHER")
BiocManager::install("phangorn")
BiocManager::install("ggplot2")
BiocManager::install("phyloseq")
```

This downloads all the things!

# LOAD LIBRARIES

```{r loadlibs, eval=FALSE}
library(dada2)#sequence processing
library(ShortRead)
library(Biostrings)
library(DECIPHER)#sequence alignment
library(phangorn) #phylogenetic tree generation
library(dplyr) #for %>%
library(ggplot2) #data viz and analysis
library(gridExtra) #for grid.arrange()
library(phyloseq) #data viz and analysis
```

Along with the dada2 library, we also load the ShortRead and the Biostrings package (R Bioconductor packages; can be installed from the following locations, dada2, ShortRead and Biostrings) which will help in identification and count of the primers present on the raw FASTQ sequence files.

Define the following path variable so that it points to the directory containing those files on your machine:

```{r setpath, eval=FALSE}
path <- "~/Desktop/Github/16S-rRNA-analysis/data/raw_reads" #change ME to the directory containing the fastq files after unzipping
list.files(path)
```

If the packages loaded successfully and your listed files match those here, you are ready to follow along with the ITS workflow.

Before proceeding, we will now due a bit of housekeeping, and generate matched lists of the forward and reverse read files, as well as parsing out the sample name. Here we assume forward and reverse read files are in the format SAMPLENAME\_1.fastq.gz and SAMPLENAME\_2.fastq.gz, respectively, so string parsing may have to be altered in your own data if your filenamess have a different format.

```{r sortfiles, eval=FALSE}
#sort files to ensure forward/reverse reads are in the same order
fnFs <- sort(list.files(path, pattern = "_R1_001.fastq.gz"))
fnRs <- sort(list.files(path, pattern = "_R2_001.fastq.gz"))

```

<!-- ### Identify primers -->

<!-- The <> (forward) and <> (reverse) primers were used to amplify this dataset. We record the DNA sequences, including ambiguous nucleotides, for those primers. -->

<!-- ```{r} -->

<!-- FWD <- "ACCTGCGGARGGATCA"  ## CHANGE ME to your forward primer sequence -->

<!-- REV <- "GAGATCCRTTGYTRAAAGTT"  ## CHANGE ME... -->

<!-- forward_F1 = "ACACTCTTTCCCTACACGACGCTCTTCCGATCTGAGTGYCAGCMGCCGCGGTAA" -->

<!-- forward_F2 = "ACACTCTTTCCCTACACGACGCTCTTCCGATCTTGAGTGYCAGCMGCCGCGGTAA" -->

<!-- forward_F3 = "ACACTCTTTCCCTACACGACGCTCTTCCGATCTCTGAGTGYCAGCMGCCGCGGTAA" -->

<!-- forward_F4 = "ACACTCTTTCCCTACACGACGCTCTTCCGATCTACTGAGTGYCAGCMGCCGCGGTAA" -->

<!-- forward_F5 = "ACACTCTTTCCCTACACGACGCTCTTCCGATCTGACTGAGTGYCAGCMGCCGCGGTAA" -->

<!-- forward_F6 = "ACACTCTTTCCCTACACGACGCTCTTCCGATCTTGACTGAGTGYCAGCMGCCGCGGTAA" -->

<!-- reverse_F1 = "GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCTACGGACTACNVGGGTWTCTAAT"  -->

<!-- ``` -->

<!-- In theory if you understand your amplicon sequencing setup, this is sufficient to continue. However, to ensure we have the right primers, and the correct orientation of the primers on the reads, we will verify the presence and orientation of these primers in the data. -->

<!-- ```{r} -->

<!-- allOrients <- function(primer) { -->

<!--     # Create all orientations of the input sequence -->

<!--     require(Biostrings) -->

<!--     dna <- DNAString(primer)  # The Biostrings works w/ DNAString objects rather than character vectors -->

<!--     orients <- c(Forward = dna, Complement = complement(dna), Reverse = reverse(dna),  -->

<!--         RevComp = reverseComplement(dna)) -->

<!--     return(sapply(orients, toString))  # Convert back to character vector -->

<!-- } -->

<!-- FWD.orients <- allOrients(FWD) -->

<!-- REV.orients <- allOrients(REV) -->

<!-- FWD.orients -->

<!-- ``` -->

## Plot quality scores and determine trimming cutoffs

I followed [this](https://www.youtube.com/watch?v=wV5_z7rR6yw) DADA2 pipeline/tutorial from youtube.

```{r specifypaths, eval=FALSE}
#Extract samples sames, assuming filenames have format: SAMPLENAME_XXX.fastq
sample.names <- sapply(strsplit(fnFs, "_"), `[`, 1)
#sample.names <- sapply(strsplit(fnRs, "_"), `[`, 1)

#specify the pull path to the fnFs and fnRs
fnFs <- file.path(path, fnFs)
fnRs <- file.path(path, fnRs)
```

\#\#\#Plot average quality scores for forward reads

```{r plotqualityforward, eval=FALSE}
plotQualityProfile(fnFs[1:2])
```

Overall, the quality scores of these two samples are very good. Scores never really dip below 30. We don't really need to trim these. If we want to be safe, we can trim 10 bp off the end of the reads to make sure we only have high quality data. Don't have to look at all the files before determining where to trim since there's not much variation in quality scores.

Plot average quality scores of reverse reads

```{r plotqualityreverse, eval=FALSE}
plotQualityProfile(fnRs[1:2])
```

Almost complete overlap with v4 regions. So we can be pretty aggressive with trimming.

Trim at 250bp??

```{r newfilepath, eval=FALSE}
# create a new file path to store filtered and trimmed reads
filt_path <- file.path(path, "filtered") #place filtered files in filtered/ subdirectory

#rename filtered files
filtFs <- file.path(filt_path, paste0(sample.names, "_F_filt.fastq.gz"))
filtRs <- file.path(filt_path, paste0(sample.names, "_R_filt.fastq.gz"))
```

### Quality filtering and trimming

```{r filterandtrim, eval=FALSE}
#this step may take a while... may be a good idea to run first on a a subset of samples. 
out <- filterAndTrim(fnFs, filtFs, fnRs, filtRs, truncLen=c(290, 250), trimLeft=20, 
                     maxN=0, maxEE=c(2,2), truncQ=2, rm.phix=TRUE, 
                     compress=TRUE, multithread = TRUE) #on windows set multithread=F

head(out)
```

specify where your starting reads are and specify where you want to put the forward reads. same with reverse. cut forward reads at 290, reverse reads at 250. consider how long your amplicon is and how much overlap you need. (dada2 needs 20nt by default) but for v4, you have almost complete overlap so it doesn't much matter. for longer amplicons (v4-v5) trimLeft = 20 (making sure the primers are cut off...) maxN = max number of ambiguous bases allowed. here we get rid of any ambiguous base pairs. maxEE = max amount of estimated errors. If you are losing too many reads due to poor sequence quality, you can relax this. truncQ= truncated reads at the first instance of a Q score less than or equal to the value specified (2 is \~63% chance of a base call being incorrect!) re.phix = removes reads identified as belonging to the phiX phage (adds sequence diversity to the sample) compress= should FASTQ files be gzipped (saves memory)

## Clip barcodes

If you want a more precise way to clip barcodes, you can follow [this](https://benjjneb.github.io/dada2/ITS_workflow.html) tutorial. It'll show you how to identify and remove primers from the reads + verify primer orientation and removal. We don't really need to do that here because there is sufficient overlap bc we're looking at the V4 region.

## Error rate estimation

```{r estimateerrors, eval=FALSE}
#estimate the error model for DADA2 algorithm using forward reads
errF <- learnErrors(filtFs, multithread=TRUE)
#estimate the error model for DADA2 algorithm using reverse reads
errR <- learnErrors(filtRs, multithread=TRUE)
#Notice that error rate estimation takes longer for reverse reads. (more steps bc reverse reads are worse)
```

This command creates an error model that will be used by the DADA2 algorithm downstream. Every batch of sequencing will have a different error rate (so important to rerun every time). Algorithm starts with the assumption that the error rates are the maximum possible (takes most abundant sequence and that's the only sequence that's true and all the others are wrong. Alternates error rate estimation and sample composition inference until they converge at a consistent solution that makes sense for both of those parameters.

Plot error rates

```{r ploterrors, eval=FALSE}
plotErrors(errF, nominalQ=TRUE)
plotErrors(errR, nominalQ=TRUE)
```

This plots frequency of each possible base transition as a function of quality score. The black line is the observed error rates. Red line is the expected error rate under the nominal definition of the Q value. In general, freq of errors dec as quality score increase. These look as expected so we can proceed with data analysis.

## Dereplicate reads

Now that you've estimated your error model from your reads, the next thing to do is dereplicate them

This is so that dada2 doesn't have to work on every single read you have. speeds up computation.

```{r dereplicatereads, eval=FALSE}
# dereplicate FASTQ files to speed up computation
derepFs <- derepFastq(filtFs, verbose=TRUE)
derepRs <- derepFastq(filtRs, verbose=TRUE)

#name the derep-class objects by the sample names
names(derepFs) <- sample.names
names(derepRs) <- sample.names
```

## Sample inference!

This is the heart of the DADA2 program. This step takes a long time!

```{r sampleinference, eval=FALSE}
#apply core sequence-variant inference algorithm to forward reads
dadaFs<-dada(derepFs, err=errF, multithread = TRUE)

#apply core sequence-variant inference algorithm to reverse reads
dadaRs<-dada(derepRs, err=errR, multithread = TRUE)
```

Using the error model developed earlier (and the depreplicated reads), the algorithm calculates abundance p-values for each unique sequence. Tests null hypothesis that a sequence (with a given error rate) is too abundant to be explained by sequencing errors. Low p-value - indicates that there are more reads of sequence than can be explained by sequencing errors. High p-value - a read was likely caused by errors (thrown out bc it's low quality data).

## Merge paired end reads

Now that you've denoised all of your forward and reverse reads, you can actually merge all of them.

```{r merge, eval=FALSE}
#merge denoised reads
mergers <- mergePairs(dadaFs, derepFs, dadaRs, derepRs, verbose=TRUE)
#Inspect the merger data.frame from the first sample
head(mergers[[1]])
```

This merges paired end reads only if they EXACTLY overlap. This is because both forward and reverse reads have been denoised and should be error-free. Can be changed by adding maxMismatch option (default is 0, can make it 1 or 2). By default, the program requires 20nt of overlap but you can lower it if need be with minOverlap (or can concatenate your reads). Since our reads almost completely overlap, we do not have to worry about this setting but you should keep it in mind if you sequenced larger regions like V4V5. Might have to compromise how much you trim earlier.

## Tabulate denoised and merged reads

Now that you've merged forward and reverse reads, you can create a sequence table!

```{r tabulate, eval=FALSE}
#organize ribosomal sequence variants (RSVs) into a sequence table (analogous to an OTU table) 
seqtab <- makeSequenceTable(mergers) 
dim(seqtab) 

#view the length of all total RSVs
table(nchar(getSequences(seqtab))) #length in bp of the merged sequence

# top row = size of merged read (bp)
# bottom row = frequency (# of sequences) 
# eg. N sequences are NNN bp long. 

```

The DADA2 sequence table is analogous to a traditional OTU table. Breaks down the size of these amplicons.

## Chimera checking and removal

Chimera = a PCR artifact. If you're amplifying a region of your 16S gene, your primer will start amplifying the region that it targets... but at some point it will fall off and it won't go all the way through. That oligo can be used to prime another completely unrelated 16S region. You'll get amplicons that are actually fusions of 2 (or more) parent sequences. This command takes your sequence table and performs a multiple sequence alignment. Starts with your least abundant read and for all the reads that are more abundant, will do sequence alignments with all possible combinations to see if there's any overlap.

```{r chimeras, eval=FALSE}
# perform de novo chimera sequence detection and removal
seqtab.nochim <- removeBimeraDenovo(seqtab, method="consensus", multithread=TRUE, verbose=TRUE)
dim(seqtab.nochim)

# calculate the proportion of non-chimeric RSVs (as a function of those that are)
sum(seqtab.nochim)/sum(seqtab)
#e.g. 71-96% of your reads are not chimeric. 
```

Utilizes a de novo rather than reference-based system to check for two-parent (bimeric) chimeras Performs a Needleman-Wunsch global alignment of each sequence to all more abundant sequences. Searches for all possible combinations of "left" and "right" parents to completely cover child sequence.

To illustrate... you have 2 parents and during the course of amplification, you create this child read that's half and half. Next you align the child read with all possible parents. With some sequences, you can recognize and filter chimeras from your sequence table.

Here we may see that a vast majority (64.6%) of our unique RSVs are identified as bimeras (illustrates the importance of chimera checking). This is normal... HOWEVER, it SHOULD be that most of my total reads (second line) were not flagged as chimeric (96.6%).

The first time I ran this analysis, we find that a majority of TOTAL READS were chimeric (100-41%), so we had a problem...

The problem was mostly likely that the primers were not completely removed from reads. Using primers with ambiguous bases can cause reads to be flagged as chimeras. So I re-ran filterAndTrim() command using the trimLeft (=20) argument. Now this problem has been fixed! IF that hadn't fixed it, there could have been other factors at play. We could have tried trimming more low-quality bases. Think about which hypervariable region you're sequencing (i.e. V4 vs V4V5). How complex is your community?

## Assign Taxonomy

Now that we have a sequence table with clean reads, we can assign taxonomy.

```{r taxonomy, eval=FALSE}
# assign taxonomy using Silva database (RDP and Greengenes also available)
# this is performed in two steps: this first one assigns phylum to genus

taxa <- assignTaxonomy (seqtab.nochim, "~/Desktop/Github/16S-rRNA-analysis/data/taxonomy/silva_nr99_v138.1_wSpecies_train_set.fa.gz", multithread = TRUE)
unname(head(taxa)) #this dataset assigns taxonomy up to species when possible. 
```

Utilizes RDP Naive Bayesian Classifier algorithm to assign taxonomy to chimera-removed sequence variants Even though it has RDP in the data, you can use RDP, Greengenes, or SILVA for classification of 16S sequences. DADA2 formatted databases available online for all 3. RDP is more frequently updated than Greengenes.

I found the SILVA database [here](https://benjjneb.github.io/dada2/assign.html) and downloaded from [here](https://zenodo.org/record/4587955#.Ye3lCFjMLQ0).

If you'd like to add species-level assignment to the dataset without species in it (we don't need it here but I'll keep it for future reference), you would use the "addSpecies" command as shown here.

```{r species, eval=FALSE}
## assign species (when possible) 
#system.time({taxa.plus <- addSpecies(taxa, "~/Desktop/Github/16S-rRNA-analysis/data/taxonomy/silva_species_assignment_v138.1.fa.gz", verbose=TRUE)
#colnames(taxa.plus) <- c("Kingdom", "Phylum", "Class", "Order", "Family", "Genus", "Species")
#unname(taxa.plus)})
```

As you can see, only a few (246/5852) of the taxa were assigned to the species level. Possible explanation: v4 regions is fairly small (combination of regions like v4v5 might give more resolution) v4 region doesn't have a lot of variation (other regions like v1 might be better for species level assignment bc it's more variable).

## Align sequences

```{r align, eval=FALSE}
#extract sequences from DADA2 output
sequences <- getSequences(seqtab.nochim)
names(sequences)<-sequences

#run sequence alignment (MSA) using DECIPHER
alignment <- AlignSeqs(DNAStringSet(sequences), anchor=NA)
```

If you aren't that interested in phylogeny, you could technically stop here. We currently have a sequence table but no info about possible phylogenetic relationships between them.

A tree is needed to calculate certain metrics like Unifrac. Sequence alignment is needed before tree construction.

## Construct Phylogenetic Tree

Next you pass that sequence alignment

```{r tree, eval=FALSE}
#change sequence alignment output into a phyDat structure. 
phang.align <- phyDat(as(alignment, "matrix"), type="DNA")

#create distance matrix
dm <- dist.ml(phang.align)

#perform neighbor joining
treeNJ<- NJ(dm) # note, tip order != sequence order

#internal maximum likelihood
fit = pml(treeNJ, data=phang.align)
##negative edges length changed to 0!

fitGTR <- update(fit, k=4, inv=0.2)
fitGTR <- optim.pml(fitGTR, model="GTR", optInv=TRUE, optGamma=TRUE, 
                    rearrangement = "stochastic", control = pml.control(trace=0))
# pml computes the likelihood of a given tree. 
# optim.pml() will optimize tree topology and branch length for your selected model of nucleotide evolution. 
#this last step takes a LONG time. 12 hours? more?

# Save tree to image
save(fitGTR, file = "OTU_tree.RData")
plot_tree(fitGTR$tree)
```

# Have R email you when it's done running.

```{r mailalert, eval=FALSE}
###Calculating - your wish is R's command.
#library(mail)
#Send yourself an email - specify your preferred email address, subject, and message. The password is fixed at "rmail".
#sendmail("julie.jung@utah.edu", subject="Notification from R", message="Conditions finished running!", password="SURpr!se93")
```

constructs a phylogenetic tree using neighbor joining methods. subsequently uses this tree as a starting point to create a GTR+G+I (generalized time-reversible with Gamma rate variation) maximum likelihood tree.

Fasttree also works to create trees from your data.

## Import data into [Phyloseq](https://joey711.github.io/phyloseq/)

to actually analyze and visualize your data.

```{r phyloseq, eval=FALSE}
map <- import_qiime_sample_data("~/Desktop/Github/16S-rRNA-analysis/map.txt")

ps <- phyloseq(otu_table(seqtab.nochim, taxa_are_rows=FALSE), 
               tax_table(taxa.plus), phy_tree(fitGTR$tree))
#takes all components we have and puts into a usable format. 

#merge PS object with map
ps <- merge_phyloseq(ps, map)
ps
```

Imports metadata specified in the provided mapping file. Will associate metadata with each sample. Allows for visualization and analysis.

## Root phylogenetic tree

Currently, the phylogenetic tree is not rooted. Though it is not necessary here, you will need to root the tree if you want to calculate any phylogeny based diversity metrics (like Unifrac).

```{r roottree, eval=FALSE}
set.seed(711)
phy_tree(ps) <- root(phy_tree(ps), sample(taxa_names(ps), 1), resolve.root=TRUE)
is.rooted(phy_tree(ps))
```

If you don't do this step, the program will randomly assign the root of the tree every time you calculate these metrics! Then it's possible to get different answers from the same data.

If you don't plan on using phylogenetic diversity metrics (Faith's PD, Unifrac, etc) this is not necessary. Shannon and Simpson (alpha diversity) and Bray-Curtis Dissimilarity (beta diversity) all don't need phylogeny to be calculated.

## Data summary and assessment

While there are numerous possible ways to evaluate your data, a standard starting approach would consist of the following [steps](https://github.com/shandley/Microbiome-Analysis-Using-R/blob/master/analysis/16S_analysis.Rmd):

1\) Evaluate Amplicon Sequence Variants (ASV) summary statistics 2) Detect and remove outlier samples 3) Taxon cleaning 4) Prevalence estimation and filtering

```{r data-assessment, eval=FALSE}
# Create a new data frame of the sorted row sums, a column of sorted values from 1 to the total number of individuals/counts for each ASV and a categorical variable stating these are all ASVs.
readsumsdf <- data.frame(nreads = sort(taxa_sums(ps), decreasing = TRUE), 
                        sorted = 1:ntaxa(ps),
                        type = "ASVs")
# Make a data frame with a column for the read counts of each sample for histogram production
sample_sum_df <- data.frame(sum = sample_sums(ps))
# Make plots
# Generates a bar plot with # of reads (y-axis) for each taxa. Sorted from most to least abundant
# Generates a second bar plot with # of reads (y-axis) per sample. Sorted from most to least
p.reads = ggplot(readsumsdf, aes(x = sorted, y = nreads)) +
  geom_bar(stat = "identity") +
  ggtitle("ASV Assessment") +
  scale_y_log10() +
  facet_wrap(~type, scales = "free") +
  ylab("# of Sequences")
# Histogram of the number of Samples (y-axis) at various read depths
p.reads.hist <- ggplot(sample_sum_df, aes(x = sum)) + 
  geom_histogram(color = "black", fill = "firebrick3", binwidth = 150) +
  ggtitle("Distribution of sample sequencing depth") + 
  xlab("Read counts") +
  ylab("# of Samples")
# Final plot, side-by-side
grid.arrange(p.reads, p.reads.hist, ncol = 2)
# Basic summary statistics
summary(sample_sums(ps))
```

The above data assessment is useful for getting an idea of 1) the number of sequences per taxa (left plot). This will normally be a "long tail" with some taxa being highly abundant in the data tapering off to taxa with very few reads, 2) the number of reads per sample. Note the spike at the lowest number of reads due to samples taken from mice given antibiotics. Very low read count can be indicative of a failed reaction. Both of these plots will help give an understanding of how your data are structured across taxa and samples and will vary depending on the nature of your samples.

Samples with unexpectedly low number of sequences can be considered for removal. This is an intuitive process and should be instructed by your understanding of the samples in your study. For example, if you have 5 samples from stool samples, one would expect to obtain thousands, if not several thousands of ASV. This may not be the case for other tissues, such as spinal fluid or tissue samples. Similarly, you may not expect thousands of ASV from samples obtained from antibiotic treated organisms. Following antibiotic treatment you may be left with only dozens or hundreds of ASV. So contextual awareness about the biology of your system should guide your decision to remove samples based on ASV number.

Importantly, at each stage you should document and justify your decisions. If you are concerned that sample removal will alter the interpretation of your results, you should run your analysis on the full data and the data with the sample(s) removed to see how the decision affects your interpretation.

The above plots provide overall summaries about the number of ASV found in all of your samples. However, they are not very useful for identifying and removing specific samples. One way to do this is using code from the following R chunk.

*Step 2: Detect and remove outlier samples*

Detecting and potentially removing samples outliers (those samples with underlying data that do not conform to experimental or biological expectations) can be useful for minimizing technical variance. One way to identify sample outliers is shown in the R chunk below.

```{r sample-removal-identification, eval=FALSE}
# Format a data table to combine sample summary data with sample variable data
ss <- sample_sums(ps)
sd <- as.data.frame(sample_data(ps))
ss.df <- merge(sd, data.frame("ASV" = ss), by ="row.names")

# Plot the data by the treatment variable
y = 1000 # Set a threshold for the minimum number of acceptable reads. Can start as a guess
x = "SampleName" # Set the x-axis variable you want to examine
label = "SampleName" # This is the label you want to overlay on the points

p.ss.boxplot <- ggplot(ss.df, aes_string(x, y = "ASV", color = "SampleName")) + 
  geom_boxplot(outlier.colour="NA", position = position_dodge(width = 0.8)) +
  geom_jitter(size = 2, alpha = 0.6) +
  scale_y_log10() +
  #facet_wrap(~SampleName) +
  geom_hline(yintercept = y, lty = 2) +
  geom_text(aes_string(label = label), size = 3, nudge_y = 0.05, nudge_x = 0.05)
p.ss.boxplot
```

The example data does have 1 sample with fewer than 1,000 ASV. When questionable samples arise you should take note of them so if there are samples which behave oddly in downstream analysis you can recall this information and perhaps justify their removal. In this case lets remove them for practice.

```{r sample-outlier-removal, eval=FALSE}
nsamples(ps)
ps1 <- ps %>%
  subset_samples(SampleName != "PCR_R_bc41")
nsamples(ps1)
```

Note that we created a new PhyloSeq object called ps1. This preserves all of the data in the original ps0 and creates a new data object with the offending samples removed called ps1.

Failure to detect and remove "bad" samples can make interpreting ordinations much more challenging as they typically project as "outliers" severely skewing the rest of the samples. These samples also increase variance and will impede your ability to identify diferentially abundant taxa between groups. Thus sample outlier removal should be a serious and thoughtful part of every analysis in order to obtain optimal results.

*Step 3: Taxon cleaning*

The following R chunk removes taxa not-typically part of a bacterial microbiome analysis.

```{r taxon-cleaning, eval=FALSE}
# Some examples of taxa you may not want to include in your analysis
get_taxa_unique(ps1, "Kingdom")
get_taxa_unique(ps1, "Class")
ps1 # Check the number of taxa prior to removal
ps2 <- ps1 %>%
  subset_taxa(
    Kingdom == "Bacteria" &
    Family  != "mitochondria" &
    Class   != "Chloroplast" &
    Phylum != "Cyanobacteria/Chloroplast"
  )
ps2 # Confirm that the taxa were removed
get_taxa_unique(ps2, "Kingdom")
get_taxa_unique(ps2, "Class")
```

## Prevalance assessment

Identification of taxa that are poorly represented in an unsupervised manner can identify taxa that will have little to no effect on downstream analysis. Sufficient removal of these "low prevalence" features can enhance many analysis by focusing statistical testing on taxa common throughout the data.

This approach is frequently poorly documented or justified in methods sections of manuscripts, but will typically read something like, "Taxa present in fewer than 3% of all of the samples within the study population and less than 5% relative abundance were removed from all subsequent analysis.".

While the ultimate selection criteria can still be subjective, the following plots can be useful for making your selection criteria.

```{r prevalence-assessment, eval=FALSE}
# Prevalence estimation
# Calculate feature prevalence across the data set
prevdf <- apply(X = otu_table(ps2),MARGIN = ifelse(taxa_are_rows(ps2), yes = 1, no = 2),FUN = function(x){sum(x > 0)})
# Add taxonomy and total read counts to prevdf
prevdf <- data.frame(Prevalence = prevdf, TotalAbundance = taxa_sums(ps2), tax_table(ps2))
#Prevalence plot
prevdf1 <- subset(prevdf, Phylum %in% get_taxa_unique(ps, "Phylum"))
p.prevdf1 <- ggplot(prevdf1, aes(TotalAbundance, Prevalence / nsamples(ps2),color=Family)) +
  geom_hline(yintercept = 0.05, alpha = 0.5, linetype = 2) +
  geom_point(size = 3, alpha = 0.7) +
  scale_x_log10() +
  xlab("Total Abundance") + ylab("Prevalence [Frac. Samples]") +
  facet_wrap(~Phylum) +
  theme(legend.position="none") +
  ggtitle("Phylum Prevalence in All Samples\nColored by Family")
p.prevdf1
```

This code will produce a plot of all of the Phyla present in your samples along with information about their prevalence (fraction of samples they are present in) and total abundance across all samples. In this example we drew a dashed horizontal line to cross at the 5% prevalence level (present in \> 5% of all of the samples in the study). If you set a threshold to remove taxa below that level you can visually see how many and what types of taxa will be removed. Whatever, threshold you choose to use it should be well documented within your materials and methods.

An example on how to filter low prevalent taxa is below.

```{r prevelance-filtering, eval=FALSE}
## Remove specific taxa
## Define a list with taxa to remove
#filterPhyla = c("Fusobacteria", "Tenericutes")
#filterPhyla
#get_taxa_unique(ps2, "Phylum")
#ps2.prev <- subset_taxa(ps2, !Phylum %in% filterPhyla) 
#get_taxa_unique(ps2.prev, "Phylum")
```

```{r, eval=FALSE}
# Removing taxa that fall below 5% prevelance
# Define the prevalence threshold
prevalenceThreshold = 0.05 * nsamples(ps2)
prevalenceThreshold
# Define which taxa fall within the prevalence threshold
keepTaxa <- rownames(prevdf1)[(prevdf1$Prevalence >= prevalenceThreshold)]
ntaxa(ps2)
# Remove those taxa
ps2.prev <- prune_taxa(keepTaxa, ps2)
ntaxa(ps2.prev)
```

## Data transformation

Many analysis in community ecology and hypothesis testing benefit from data transformation. Many microbiome data sets do not fit to a normal distribution, but transforming them towards normality may enable more appropriate data for specific statistical tests. The choice of transformation is not straight forward. There is literature on how frequently used transformations affect certain analysis, but every data set may require different considerations. Therefore, it is recommended that you examine the effects of several transformations on your data and explore how they alter your results and interpretation.

The R chunk below implements several commonly used transformations in microbiome research and plots their results. Similar to outlier removal and prevalence filtering, your choice should be justified, tested and documented.

```{r data-transform, eval=FALSE}
# Transform to Relative abundances
ps2.ra <- transform_sample_counts(ps2, function(OTU) OTU/sum(OTU))

# Transform to Proportional Abundance
ps2.prop <- transform_sample_counts(ps2, function(x) min(sample_sums(ps2)) * x/sum(x))

# Log transformation moves to a more normal distribution
ps2.log <- transform_sample_counts(ps2, function(x) log(1 + x))

# View how each function altered count data
par(mfrow=c(1,4))
plot(sort(sample_sums(ps2), TRUE), type = "o", main = "Native", ylab = "RSVs", xlab = "Samples")
plot(sort(sample_sums(ps2.log), TRUE), type = "o", main = "log Transformed", ylab = "RSVs", xlab = "Samples")
plot(sort(sample_sums(ps2.ra), TRUE), type = "o", main = "Relative Abundance", ylab = "RSVs", xlab = "Samples")
plot(sort(sample_sums(ps2.prop), TRUE), type = "o", main = "Proportional Abundance", ylab = "RSVs", xlab = "Samples")
par(mfrow=c(1,4))

# Histograms of the non-transformed data vs. the transformed data can address the shift to normality
p.nolog <- qplot(rowSums(otu_table(ps2))) + ggtitle("Raw Counts") +
  theme_bw() +
  xlab("Row Sum") +
  ylab("# of Samples")

p.log <- qplot(log10(rowSums(otu_table(ps2)))) +
  ggtitle("log10 transformed counts") +
  theme_bw() +
  xlab("Row Sum") +
  ylab("# of Samples")

#library(ggpubr)
ggpubr::ggarrange(p.nolog, p.log, ncol = 2, labels = c("A)", "B)"))
```

These are both pretty similar so I will *not* do any transformation here.

## Community composition plotting

Classic bar plots of bacterial phyla present per sample can be useful for communicating "high level" results. These are relatively easy to interpret when major shifts in microbial communities are present, such as in this study where antibiotics are used However, they are not effective at detecting subtle shifts in communities or taxa and do not convey any statistical significance and can be subjectively interpreted. Interpretation of these plots should always be subject to subsequent statistical analysis.

```{r community-composition-plots, eval=FALSE}
# Create a data table for ggplot
ps2_phylum <- ps2 %>%
  tax_glom(taxrank = "Phylum") %>%                     # agglomerate at phylum level
  transform_sample_counts(function(x) {x/sum(x)} ) %>% # Transform to rel. abundance (or use ps0.ra)
  psmelt() %>%                                         # Melt to long format for easy ggploting
  filter(Abundance > 0.01)                             # Filter out low abundance taxa
```

```{r}
# Plot - Phylum
p.ra.phylum <- ggplot(ps2_phylum, aes(x = SampleName, y = Abundance, fill = Phylum)) +
  geom_bar(position="fill", stat = "identity", color="white") +
    labs(x = "sample",
       y = "abundance") +
    scale_x_discrete(labels=c("1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15", "16"))+
    scale_fill_discrete(name = "Abundant Phyla (> 1%)") +
    theme_bw(base_size=9, base_family="Palatino") + 
    theme(axis.text.y=element_text(size=16, colour= "black")) +
  theme(axis.text.x=element_text(size=16, colour= "black")) +
  theme(axis.title.x=element_text(size=16, colour = "black")) +
  theme(axis.title.y=element_text(size=16, colour = "black"))
p.ra.phylum

# You can rerun the first bit of code in this chunk and change Phylum to Species, Genus, etc.
# Draw in interactive plotly plot
plotly::ggplotly(p.ra.phylum)

ggsave("~/Desktop/phyl_diversity.pdf",
 plot = p.ra.phylum, # or give ggplot object name as in myPlot,
 width = 20, height = 14,
 units = "cm", # other options c("in", "cm", "mm"),
 dpi = 300,
 useDingbats=FALSE)
```

```{r genus-composition-plots, eval=FALSE}
# Create a data table for ggplot
ps2_genus <- ps2 %>%
  tax_glom(taxrank = "Genus") %>%                     # agglomerate at phylum level
  transform_sample_counts(function(x) {x/sum(x)} ) %>% # Transform to rel. abundance (or use ps0.ra)
  psmelt() %>%                                         # Melt to long format for easy ggploting
  filter(Abundance > 0.01)                             # Filter out low abundance taxa
# Plot - Genus
p.ra.genus <- ggplot(ps2_genus, aes(x = SampleName, y = Abundance, fill = Genus)) +
  geom_bar(position="fill", stat = "identity", color="white") +
    labs(x = "sample",
       y = "abundance") +
    scale_x_discrete(labels=c("1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15", "16"))+
    scale_fill_discrete(name = "Abundant Genera (> 1%)") +
    theme_bw(base_size=9, base_family="Palatino") + 
    theme(axis.text.y=element_text(size=16, colour= "black")) +
  theme(axis.text.x=element_text(size=16, colour= "black")) +
  theme(axis.title.x=element_text(size=16, colour = "black")) +
  theme(axis.title.y=element_text(size=16, colour = "black"))
p.ra.genus

# You can rerun the first bit of code in this chunk and change Phylum to Species, Genus, etc.
# Draw in interactive plotly plot
plotly::ggplotly(p.ra.genus)

ggsave("~/Desktop/gen_diversity.pdf",
 plot = p.ra.genus, # or give ggplot object name as in myPlot,
 width = 30, height = 14,
 units = "cm", # other options c("in", "cm", "mm"),
 dpi = 300,
 useDingbats=FALSE)
```

```{r family-composition-plots, eval=FALSE}
# Create a data table for ggplot
ps2_family <- ps2 %>%
  tax_glom(taxrank = "Family") %>%                     # agglomerate at phylum level
  transform_sample_counts(function(x) {x/sum(x)} ) %>% # Transform to rel. abundance (or use ps0.ra)
  psmelt() %>%                                         # Melt to long format for easy ggploting
  filter(Abundance > 0.01)        # Filter out low abundance taxa

# Plot - Family
p.ra.family <- ggplot(ps2_family, aes(x = SampleName, y = Abundance, fill = Family)) +
  geom_bar(position="fill", stat = "identity", color="white") +
    labs(x = "sample",
       y = "abundance") +
    scale_x_discrete(labels=c("1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15", "16"))+
    scale_fill_discrete(name = "Abundant Families (> 1%)") +
    theme_bw(base_size=9, base_family="Palatino") + 
    theme(axis.text.y=element_text(size=16, colour= "black")) +
  theme(axis.text.x=element_text(size=16, colour= "black")) +
  theme(axis.title.x=element_text(size=16, colour = "black")) +
  theme(axis.title.y=element_text(size=16, colour = "black"))
p.ra.family

# You can rerun the first bit of code in this chunk and change Phylum to Species, Genus, etc.
# Draw in interactive plotly plot
plotly::ggplotly(p.ra.family)

ggsave("~/Desktop/fam_bac_diversity.pdf",
 plot = p.ra.family, # or give ggplot object name as in myPlot,
 width = 30, height = 14,
 units = "cm", # other options c("in", "cm", "mm"),
 dpi = 300,
 useDingbats=FALSE)
```

## Barplot of families

from [here](https://benjjneb.github.io/dada2/tutorial.html)

```{r, eval=FALSE}
#top20 <- names(sort(taxa_sums(ps2), decreasing=TRUE))[1:20]
#ps.top20 <- transform_sample_counts(ps2, function(OTU) OTU/sum(OTU))
#ps.top20 <- prune_taxa(top20, ps.top20)
#plot_bar(ps.top20, x="SampleName", fill="Family") 

#top50 <- names(sort(taxa_sums(ps2), decreasing=TRUE))[1:50]
#ps.top50 <- transform_sample_counts(ps2, function(OTU) OTU/sum(OTU))
#ps.top50 <- prune_taxa(top50, ps.top50)
#plot_bar(ps.top50, x="SampleName", fill="Family") 

#top100 <- names(sort(taxa_sums(ps2), decreasing=TRUE))[1:100]
#ps.top100 <- transform_sample_counts(ps2, function(OTU) OTU/sum(OTU))
#ps.top100 <- prune_taxa(top100, ps.top100)
#plot_bar(ps.top100, x="SampleName", fill="Family") 

#+ facet_wrap(~SampleName, scales="free_x")
```

```{r saveplot, eval=FALSE}
#ggsave("~/Desktop/fam_diversity.pdf",
# plot = fam_diversity, # or give ggplot object name as in myPlot,
# width = 12, height = 12,
# units = "cm", # other options c("in", "cm", "mm"),
# dpi = 300,
# useDingbats=FALSE)
```

the taxonomic distribution of the top 20, 50, and 100 sequences

# START HERE

IDK we should think about what else to plot here.

Look through this vignette for [ideas](https://bioconductor.org/packages/devel/bioc/vignettes/phyloseq/inst/doc/phyloseq-analysis.html)!

Also this F1000 paper for [ideas](https://f1000research.com/articles/5-1492/v1).

## Phyla level plots

Data with strong effects can be assessed with a variety of high-level plots. While these plots do not reveal low-level detail about bacterial species, they can provide a visual assessment of how bacterial community structure is behaving within your experimental system. As another example on how one might display the time-series data collected as part of this study one can use line plots to examine the trajectories of each major taxa following each treatment.

In order to display these plots, some work needs to be done to convert the data from a PhyloSeq object into a data table useful for non-PhyloSeq specific ggplot aesthetics. This first chunk takes care of a majority of this preparatory work.

```{r phyla-level-plots-preparation, eval=FALSE}
# agglomerate taxa to a specific taxanomic level
glom <- tax_glom(ps2, taxrank = 'Phylum')

# create dataframe from phyloseq object
dat <- tibble(psmelt(glom))

fct_reorder(dat$Phylum, dat$Abundance, .desc = TRUE)

# Select 5 most abundant Phyla for a reduced plot
dat.1 <- filter(dat, Phylum %in% c("Proteobacteria", "Bacteroidota", "Actinobacteriota", "Firmicutes", "Desulfobacterota"))
levels(dat.1$Phylum)
dat.1 <- droplevels(dat.1)
levels(dat.1$Phylum)
```

And now for the actual plot of the prepared data.

```{r phyla-level-plotting, eval=FALSE}
# Phyla plots with loess smoother 
p.smoothed.phylum <- ggplot(dat.1, aes(x = SampleName, y = Abundance, color = Phylum, group = Phylum)) +
  stat_smooth(method = "loess") +
  #facet_grid(~treatment) +
  ylab("Relative Abundance") +
  geom_point(size = 1.25, alpha = 0.4)
p.smoothed.phylum
```

## Ordination

Beta diversity enables you to view overall relationships between samples. These relationships are calculated using a distance metric calculation (of which there are many) and these multi-dimensional relationships are evaluated and viewed in the two dimensions which explain the majority of their variance. Additional dimensions can be explored to evaluate how samples are related to one another.

The UniFrac distance metric takes advantage of the phylogenetic relationships between bacterial taxa by down-weighting relationships between bacterial taxa that are phylogenetically related versus ones that are distantly related. Weighted UniFrac builds on this by also taking into account the relative abundances of each taxa. For a more detailed discussion on these distance calculations see: <https://en.wikipedia.org/wiki/UniFrac>. Ultimately, the decision on what distance metric to use should be a result of experimentation and a good understanding of the underlying properties of your data.

The following R chunks calculate UniFrac and wUniFrac on a PhyloSeq object and display the the two components of these results that explain a majority of the variance in the data using Principle Coordinates Analysis (PCoA). For a detailed explanation of how PCoA works see: <https://sites.google.com/site/mb3gustame/dissimilarity-based-methods/principal-coordinates-analysis>.

```{r ordination, eval=FALSE}
#Ordination Analysis
ord.pcoa.uni <- ordinate(ps2, method = "PCoA", distance = "unifrac")
ord.pcoa.wuni <- ordinate(ps2, method = "PCoA", distance = "wunifrac")
```

And now to plot each ordination.

```{r ordination-plots, eval=FALSE}
## Ordination plots all samples
# Unifrac
p.pcoa.uni <- plot_ordination(ps2, ord.pcoa.uni, color = "treatment", axes = c(1,2)) +
  geom_point(size = 2) +
  labs(title = "PCoA of UniFrac Distances", color = "Treatment") +
  facet_grid(~treatment_days)
p.pcoa.uni
# Weighted Unifrac
p.pcoa.wuni <- plot_ordination(ps2, ord.pcoa.wuni, color = "treatment", axes = c(1,2)) +
  geom_point(size = 2) +
  labs(title = "PCoA of wUniFrac Distances", color = "Treatment") +
  facet_grid(~treatment_days)
p.pcoa.wuni
ggarrange(p.pcoa.uni, p.pcoa.wuni, nrow = 2, labels = c("A)", "B)"))
```

## Plot tree

to actually [plot](https://joey711.github.io/phyloseq/plot_tree-examples.html) your tree:

```{r, eval=FALSE}
plot_tree(ps)
```

dots are annotated next to tips (OTUs) in the tree, one for each sample in which that OTU was observed. Some have more dots than others. Also by default, the node labels that were stored in the tree were added next to each node without any processing (although we had trimmed their length to 4 characters in the previous step).

What if we want to just see the tree with no sample points next to the tips?

```{r, eval=FALSE}
plot_tree(ps, "treeonly")
```

And what about without the node labels either?

```{r, eval=FALSE}
plot_tree(ps, "treeonly", nodeplotblank)
```

We can adjust the way branches are rotated to make it look nicer using the ladderize parameter.

```{r, eval=FALSE}
plot_tree(ps, "treeonly", nodeplotblank, ladderize="left")

plot_tree(ps, "treeonly", nodeplotblank, ladderize=TRUE)

```

And what if we want to add the OTU labels next to each tip?

```{r, eval=FALSE}
plot_tree(ps, nodelabf=nodeplotblank, label.tips="taxa_names", ladderize="left")
```

```{r, eval=FALSE}
plot_tree(ps, nodelabf=nodeplotblank, ladderize="left", color="Phylum")
```

For bacteria, trees are not actually that helpful because of recombination. Recombination is well known for disrupting phylogenetic interference, and esp to affect branch length estimates so that trees look star-like with abnormally long terminal branches. So i'm going to abandon the tree stuff for now and focus on other analyses such as barplot, alpha, and beta diversity.

## Alpha diversity metrics plotting

Alpha diversity is a standard tool researchers can use to calculate the number of bacterial taxa present in a study or study group and the relationships between relative abundance and how evenly taxa are distributed. These are classic representations of species number and diversity in a study which provide useful summary information about the numbers and relative abundances of bacterial taxa within your study.

Similar to the plot above, we can calculate several measures of alpha diversity, add them to a data frame and use ggplot2 to follow the alpha diversity trajectory over time.

```{r omit-outlier, eval=FALSE}
nsamples(ps)
ps3 <- ps %>%
  subset_samples(SampleName != "PCR_R_bc41_NA")
nsamples(ps3)
```

```{r alpha-diversity-plots, eval=FALSE}
richness<- plot_richness(ps3, x="SampleName",  measures = c("Observed", "Shannon", "Simpson")) + theme_bw(base_size=9, base_family="Palatino") +
  scale_x_discrete(labels=c("1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15", "16"))
richness
#SampleName category of the mapping file... #CHANGE THIS

ggsave("~/Desktop/richness.pdf",
 plot = richness, # or give ggplot object name as in myPlot,
 width =17, height = 9,
 units = "cm", # other options c("in", "cm", "mm"),
 dpi = 300,
 useDingbats=FALSE)


alpha.div<-estimate_richness(ps3, split=TRUE, measures=c("Observed", "Shannon", "Simpson"))
alpha.div
#alpha diversity measured
```

This command will plot the computed values for Shannon and Simpson diversity. You can also output the calculated values themselves with the following command: estimate\_richness(). Notice the error message - we never actually trimmed or rarefied our data so what gives? In DADA2, singletons always result in an abundance p-value of 1. It is incredibly difficult to differentiate true singletons from those caused by sequencing errors (can artificially.

Shannon is usually a measure of richness. Simpson is more a measure of evenness.

NOTE: The rest of this analysis is commented out because it's not relevant to our analysis here -- ie. there are no groups here to perform an ordination plot of richness or the associated stats.

ALSO, commented out is Talia's analysis.. I didn't actually end up using it here because I found a way to do it mostly in R. But if you're more into python/bash, there's a ton of useful pipelines in there.
