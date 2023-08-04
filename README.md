# Hognose

## Before Beginning
- This program assumes you have the following installed:
	- Python 3.9.
	- BioPython: http://biopython.org/DIST/docs/tutorial/Tutorial.html
	- Some normal python libraries (gzip, shutil, os, pandas). These all come standard with Python 3 or Anaconda, or are downloadable using pip.
- If you have everything installed and want to run without changing anything:
	- Make sure you have many GB of space on your hard drive. You will be generating several multi-GB files for every sample you are analyzing. If you don't have space, the program will throw an error and may not clearly explain why it stopped working.
	- Make sure your computer has sufficient memory. I do most analysis on a 2018 Windows 10 laptop with 8 GB of RAM, and that has no problem with multiple files containing 1-2 million reads each. 
	- Download your NGS data (.gz files) and place in a single folder.
	- Download 'Template_sequences.csv' and open in Excel.
	- Add the names of the samples you submitted to NGS to the column: 'sample_name'.
	- Add the template sequences of the samples you submitted to NGS to the column: 'template_sequence'.
	- Write "yes" in the column 'blank' for any templates that represent negative controls against which you want to compare enrichment levels.
	  
	  **IMPORTANT**: Make sure of the following for the template sequences in Template_sequences.csv:
		- You replace the bases at the positions you want to monitor with "N" in the template.
		- You include the sequences to the primer binding sites, but NOT the adaptor sites for the NGS facility.
		  
	- Save 'Template_sequences.csv' in the same folder as your NGS data files.
	- Make sure the third block of code (user inputs) is NOT commented out.
	- Run the program and follow the prompts.
- If you want to mess around with the code to test things:
	- Talk to Eric about generating tester files with fewer reads.
	- Read through this document and the comments at the left side of the program. 
	- Be prepared to look up anything you don't understand on Stack Overflow.


## This program WILL do the following:
- Guide the user through short, step-by-step instructions for operating the program.
- Automatically extract data from .fastq.gz or .fastq files directly from the illumina sequencer.
- Match template sequences to sample names to file names based on the contents of 'Template_sequences.csv'.
- Detect regions of interest in the template sequences and slice them from the NGS reads.
- Quantitatively rank how common particular regions of interest are in each sample (i.e. enrichment of randomized positions or barcodes).
- Filter out reads by quality and discard sequences with mismatched alignments in the regions of interest.
- Take a single batch set of user inputs at the beginning of the program and process all datafiles in the background for hands-off operation.
- Accommodate sequences with different numbers and locations of 'randomized' bases.
- Separately quality screen/align Fwd and Rev reads, then integrate them for the final tally of randomized region enrichment.
- Provide periodic updates on the status of the analysis at each stage for each data file.


## This program MIGHT be able to do the following (off-label; use at your own risk):
- Process enrichment data from traditional SELEX (i.e. 4^40-member libraries). These templates have lots of randomized positions, which could lead to unforeseen artifacts during the alignment steps. Nevertheless, I've run some very quick tests to suggest that this program should work fine as long as your winning sequences have high enough levels of enrichment. 
- Process data from RNA sequencing. I've never worked with RNA, and I don't know if there are any differences in the sequencing data format/workflow for processing these data. That said, I added "U" to the list of acceptable bases ("bases") that can can be recognized and compared against in Template_sequences.csv. My assumption is that everything should work fine, but—again—no guarantees. 


## This program WILL NOT do the following:
- Tolerate errors in 'Template_sequences.csv'.
	- NOTE: This will be the most likely source of problems, the hardest to fix, and the easiest to go unnoticed. Everything will be fine as long as you follow directions for this. 
- Process protein sequences. This is probably configurable with major modifications to the code; however, 1) the use cases for randomized protein sequences (to my knowledge at the time of writing) aren't the same as those for nucleic acid aptamers, so I don't think there is demand for including these types of sequences, 2) the under-the-hood aspects of processing NA and protein sequence data are very different (e.g. protein seq data doesn't come in the same format as NA seq data, protein seq data doesn't usually use quality scores, alignment programs for NAs and proteins aren't necessarily compatible, etc.).
- Compensate for poor-quality selections, PCR, PCR purification, or NGS sample preparation.
- Tolerate extremely short reads (the alignment step won't work as well). I don't know what the lower limit on this is, but I recommend not trimming primers from your templates or reads. 
- Make judgement calls about the chemical/biological effects of different mutations (e.g. T → G increasing MW, Tm., etc.); they're interpreted as strings.
- Align different reads to each other (multiple sequence alignment.)
- Interpret importance of particular mutations at individual positions; it assesses the summed sequences of randomized bases (known as a 'slice') for each NGS sample and provides the relative ranking against other sequences. 
- Estimate how long the operations will take. My best answer is "at least several hours" for any actual data files. The speed at which this program runs is complex and affected by 1) the computer the program is being run on, 2) any other operations running in parallel with this program, 3) the size and number of data files being processed, and 4) probably a lot of other factors. The best I'm able to do is update the console/RunLog file with milestones as they happen so the user can gauge for themselves how quickly things are progressing. 


## Troubleshooting errors:
- "I got the error 'It looks like you forgot to include "Template_sequences.csv" in your file directory (or maybe you renamed it?). Please fix this, then press ENTER to continue. I'll wait.' What do I do?"
	- Make sure Template_sequences.csv is in the same file directory you entered as the working directory.
	- Make sure Template_sequences.csv is still named 'Template_sequences.csv'.
- "FileNotFoundError: [Errno 2] No such file or directory: *some file location* *file name*"
	- Are your NGS data files in that directory? (They should be).
	- Is 'Template_sequences.csv' in that file directory? (It should be).
	- Did you rename it? (Don't).
	- Maybe you didn't paste the file directory correctly. Try it again.
- "The program finished immediately. Is that bad?"
	- Are you working with a tester data set from Eric? If so, it's supposed to be very fast.
	- Are your .fastq.gz or .fastq datafiles in your input directory? (They should be).
	- Maybe you didn't paste the file directory correctly. Try it again.
- "I accidentally hit the wrong setting at the beginning."
	- Stop the program and restart it. It'll be fine.
- "I have some information in Template_sequences.csv that are for data files that aren't in the working directory. Do I need to change the file/restart the program?"
	- You should be fine. The program will anything in Template_sequences.csv that doesn't have a corresponding data file in the working directory. If you intended to analyze these other data files, you'll need to either move them into the input file shortly after the program starts or rerun the program afterwards.
- "I have some extra files that I don't need to be analyzed in the folder I specified as the working directory. Will this be okay?"
	- It should be, yes. The program will skip over any files that don't have the expected extensions—.fastq.gz, .fastq, .fasta, .txt, .csv—i.e. your family photos won't be affected. Files with one of those extensions will also be ignored unless they follow the naming conventions of the files the program expects, so "grocery list.txt" and "budget.csv" will be safely ignored. Files with the following name structures will NOT be ignored (SampleName = any sample name in Template_sequences.csv):
		- Any .fastq.gz file (will be unzipped).
		- .fastq files following the Illumina naming conventions (see below), (will be analyzed)
		- SampleName_Fwd-hq.fasta (will be analyzed).
		- SampleName_Rev-hq.fasta (will be analyzed).
		- SampleName_Fwd-aligned.txt (will be analyzed).
		- SampleName_Rev-aligned.txt (will be analyzed).
		- SampleName_sliced.txt (will be deleted, then rewritten).
		- Winners-subtracted_SampleName.txt (will be overwritten).
		- Hognose RunLog.txt (the run log prompts from the current session will be appended to the end).
	- NOTE: Items with the following names can be major "files" in the ointment if care is not taken:
		- Template_sequences.csv (this is the lynchpin of the program; don't mess it up. The program will probably fail to run or *worse*).
		- Hognose Results.csv (will be overwritten).
- "I omitted some samples from Template_sequences.csv. What can I do?"
	- Add them to Template_sequences.csv, save, and run again. You'll overwrite any other data files that are still in that directory, so watch out.
- "I made a mistake in Template_sequences.csv. What should I do?"
	- Fix the mistakes, double-check that they're fixed, and run the program again. If There's an error in Template_sequences.csv, it's probably something subtle and fundamental to the design of your experiment that this program won't be able to catch. Fixing the mistake and running the program again will regenerate the output files from scratch.
- "The program seems to be running alright, but it's extremely slow."
	- This program will always take a long time because of how much data it needs to go through and how many different operations need to be performed. Analyzing additional files or larger files (i.e. containing more reads) will make the program take longer. This is why I haven't attempted to estimate how long the program will take. 
	- I'd recommend running this program on a newer computer (older ones without enough memory might crash) and use a local file path as the input directory: Network directories can often work, but I've found they take much longer. 
- "An unusually low number of reads passed through *screen*":
	- Quality:
		- Check Template_sequences.csv again.
		- Check the quality of your PCR products.
		- Check the quality of your PCR purifications.
	- Alignment:
		- Check Template_sequences.csv again.
		- If you really need to, advanced users can fiddle with the alignment parameters.
	- Slicing (NOTE: Any reads that made it this far probably don't have an obvious cause for being dropped *en masse*): 
		- Open up the -aligned.txt file for that sample. Visually inspect reads that are listed as giving an error.
		- Check the predicted 2D or 3D structure of your DNA. Is there something that is making PCR difficult at that particular position?
		- Consider optimizing your PCR protocol.
		- Consult with the facility that performed NGS for you. 
- "I got the error 'Non-ATCG base (N) in *file name* at position X of read Y'"
	- "What does that mean?"
		- It means that read Y had a base at position X with a quality score that was too low for the Illumina sequencer to accurately assign a base identity to it. This is similar to bases showing up as "N" in Sanger sequencing data.
	- "What happens to this read?"
		- It gets ignored. This alert should only come up if the low-quality base appears in one of the positions of interest, and you don't want to contaminate your analysis downstream by having erroneous bases.
	- "But the odd base occurs outside of the randomized positions I indicated in the template for that sequence."
		- It may appear that way, but that's only because the error is counting the base position AFTER the read is aligned, i.e. there was probably an InDel upstream of that position for that particular read. There's nothing inherently wrong with this, and if you want to see for yourself, feel free to check out that particular read in the -aligned.txt file for that sample.
	- "I'm getting this message for so many reads! Is my data terrible?"
		- Wait for the final message that says "*file name*-aligned.txt: X passed reads out of Y total.". If X is >95% of Y, you should be fine. It may seem like there are a lot of reads where this issue occurs, but that's in part because there are a LOT of reads in an NGS data file. If a few thousand need to get thrown out, you should hopefully still have at least 1 million more.
		- If X is closer to ~50% of Y, then your samples probably have a more fundamental issue that needs to be addressed (see above).
	- "I'm getting a very similar error, except instead of 'N', it's *other character* in parentheses. What do I do?"
		- Assuming the other character isn't 'A', 'T', 'C', or 'G', you should reach out to the facility that performed NGS for you. They will have more expertise on why a read would have a character other than 'A', 'T', 'C', or 'G' (the expected bases) or 'N' (for unassignable bases). 


## FAQ:
- "Does this run on Linux/Windows/Mac?"
	- Yes/yes/I'm pretty sure. If you need to run this on a niche system (e.g. JavaOS), feel free to reach out: I'd be happy to help you with this... by connecting you to a therapist.
- "Can I move my files to a different folder once they've been processed?"
	- Yes, you can move them anywhere once the program is finished running.
- "Does it matter what folder I use as the 'file path'?"
	- No, although I haven't tried it with all edge cases. As long as you follow the other guidelines in this guide, you'll probably be fine.
- "Can I rename 'Template_sequences.csv' or any of my .gz files?"
	- NO! You'd need to modify the code and/or Template_sequences.csv for this, and that will be a pain to change (unless you're already digging in the code; in that case, YOLO, but it's on you to fix).
- "Can you modify this script to.../write a new script to.../help me troubleshoot.../help me with.../explain..."
	- Reach out to me at emkohn3@wisc.edu. I'll help for $50 or co-authorship on your paper.

  
## For scientists, Breakdown of filtering steps:
- Filter for read quality. Reads pass this filter if at least 10% of bases in the region corresponding to the template have a Phred score of >= 20 (>=99% chance of correct read at that position). There is no minimun cutoff where a single base or any pattern of poor-quality scores (e.g. 3 bad bases in a row) will cause a read to get thrown out.
- Aligning. Alignments are performed using the 'pairwise2.align.globalms' function. 'pairwise2.align' means it aligns two sequences (a particular read and its corresponding template); 'global; means it goes along the whole length of the template sequence (and corresponding length of the read sequence); 'm' means pairs of identical characters are considered a match, and anything else is considered a mismatch; and 's' means the pentalties for opening/extending gaps are the same for the template and read sequences. The function tries to generate alignments that maximize the score based on the following default values (I arrived at these empirically): Match, +2; mismatch, -0.1; add a gap, -1.5; extend a gap, -0.5. https://biopython.org/docs/1.75/api/Bio.pairwise2.html
- Alignments that create a gap, '-', at the randomized position are discarded.
  

## For programmers, Background on NGS data and assumptions for this program:
- This program was meant to integrate into the workflow of the NGS facility at UW-Madison (through the Biotechnology Center), 2x250bp Illumina NovaSeq. Data files are downloaded from the NGS facility's website as .gz files. I believe this is standard for NGS centers as .gz is good at compressing large files and easy to generate from the Unix command line (most bioinformatics cores run Linux).
- Illumina naming conventions: https://support.illumina.com/help/BaseSpace_OLH_009008/Content/Source/Informatics/BS/NamingConvention_FASTQ-files-swBS.htm
- In 'SampleName_S1_L001_R1_001.fastq.gz': 
	- S1, L001, and 001 are for internal records keeping only, and the user doesn't need to worry about them. 
	- SampleName is the name the user submitted (needed for this program).
	- R1 identifies if the file is for a forward (R1) or reverse (R2) read.
	- .fastq is a file containing sequences with quality information about each read
	- .gz is the compressed form of that file.
- Fastq file structures: https://support.illumina.com/bulletins/2016/04/fastq-files-explained.html
- Decoding Phred (quality) scores: https://www.drive5.com/usearch/manual/quality_score.html


## Citation:
DOI: 10.1021/acschembio.3c00183
Authors: Eric Kohn


  
## Closing remarks
Good luck and have fun!

https://cheezburger.com/7033484544/zubon-the-happy-hognose
