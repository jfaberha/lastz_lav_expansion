LASTZ LAV EXPANSION

About
-----
There are two pipelines contained in this repository. The first, "lav_expansion.txt”, is a script that takes lav format output files from the LASTZ program created by the Miller Lab at Penn State University and expands reported blocks to base-pair resolution sequence identity in list format. The LASTZ program is designed to align one or more query sequences to a single target, and it reports back blocks of local alignments in-between gaps.  It is a versatile program and is suitable for aligning entire chromosomes and whole genomes between distantly related taxa. Admittedly, the output lav file is not very useful on its own since it is not very readable by humans.  The “lav_expansion.txt” pipeline, however, takes the lav output and expands each gap-free block in the alignment, compares all block alignments to each base-pair in the target sequence, then reports back the highest aligned sequence identity for each target basepair.
For example, it will take a lav block that looks like this:

      s 25
      b 1130 4510
      e 1155 4538
      l 1130 4510 1140 4520 88
      l 1141 4524 1155 4538 95

And expand it out to look like this (first column contains base-pair loci in target sequence, second column is sequence identity at that locus):

	1130	88
	1131	88
	1132	88
	1133	88
	1134	88
	1135	88
	1136	88
	1137	88
	1138	88
	1139	88
	1140	88
	1141	95
	1142	95
	1143	95
	1144	95
	1145	95
	1146	95
	1147	95
	1148	95
	1149	95
	1150	95
	1151	95
	1152	95
	1153	95
	1154	95
	1155	95

Possible uses for the expanded lav output include plotting best alignments for figures, comparing alignments to the same target from different query sequences, and filtering alignments based on a minimum best sequence identity. Originally, this pipeline was conceived in order to detect horizontal gene transfer (HGT) in soil bacteria species by comparing whole genome alignments.  The pipeline we used in this process is available as “hgt_detection.txt”.  In our study we had three categories of bacterial strains: 1) focal bacteria with HGT content; 2) focal bacteria from same clade without HGT content; 3) sympatric bacteria that is genetically distant from focal bacteria but shares HGT content. The experimental design for identifying HGT content was aligning each strain from category 1 to all strains in categories 2 and 3.  Category 1 vs. 3 alignments highlighted HGT content since it was much more highly conserved that the overall genome due to recent conjugal transfer.  Category 1 vs. 2 alignments served as negative controls, since the strains were from the same bacterial clade and all genomic content was highly conserved except for the HGT content, which was absent.  We then isolated HGT content that was highly conserved from category 1 vs. 3 alignments while masking anything that was highly conserved in the 1 vs. 2 alignments.  

Our bacterial strains were perfectly suited for this analysis due to their genetic distances, genomic architecture, and overall biology. While this type of approach is powerful when the aforementioned preconditions are met, it should be used with caution. We recommend the pipeline be used for exploratory purposes and we cannot guarantee the accuracy of results.  Its use requires a priori knowledge of genetic distances of both HGT and overall genomic content, and knowledge of the genetic content of strains/species used in alignments.  It is best suited for alignments where your category 1 and 3 genomes substantially diverged (ex - average of ~10% sequence divergence) to avoid detection of false positive regions of HGT which are actually highly conserved elements by descent.

For any additional details on how the pipeline was originally implemented, please refer to the following paper.  If you use either pipeline in your published research, please cite this paper as well.

********************************

Getting Started
---------------
These pipelines first require you to run LASTZ and generate lav output files for your alignments. LASTZ is available from the following website:
https://www.bx.psu.edu/miller_lab/

The script “lav_expansion.txt” can be used for any LASTZ alignment, however if you plan to proceed to the “hgt_detection.txt” pipeline you will require two alignments.  One alignment between distantly related genomes that share HGT elements, and another alignment between closely related genomes that where one lacks the HGT elements of interest. LASTZ requires that your target be a single sequence in fast format, while your query can be a multi-fasta file.  For higher-throughput processing, we recommend that if your target is actually a whole genome in multiple contig or chromosome format, you can concatenate all sequences to form a large pseudomolecule. You can refer back to the correct genomic location of an alignment retroactively by making a key of contig order and size.  You can also concatenate all query genomes from a single genome category into a single multi-fasta file for less post-processing.

These pipelines were written for and tested on a UNIX OS and uses a combination of basic UNIX commands and automatically installed programs such as awk, sed, uniq, sort, and paste. Please ensure that your version of the sed program is compatible with the parameter “-i” prior to use, otherwise the pipeline will fail. Apart from that, move the pipeline scripts to your working directory and they will be ready to run.

Running the pipelines
---------------------
Once you have the scripts and your LASTZ lav output files in the same working directory
For “lav_expansion.txt”, enter the following command

	bash lav_expansion.txt

You will then be prompted to enter the lav file prefix (ex - for file 'foo.lav' type 'foo’), and the minimum sequence identity to report in the output (ex - enter '80' for scores >=80% similarity in output).  Entering a lower score for the latter option with make the pipeline run slower, but will return a wider range of alignments.  The final output will contain the highest sequence identity among alignment blocks for each base-pair in your target sequence above the threshold you enetered, and will be named with the suffix ‘_expanded.txt’ (ex - for file 'foo.lav' input, output will be type 'foo_expanded.txt’).

For “hgt_detection.txt”, enter the following command

	bash hgt_detection.txt

You will promted to enter the name of expaned lav output for the genome category 1 vs. 3 alignment (with HGT content), then for the 1 vs. 2 alignment (negative control/masking alignment), the minimum sequence identity accepted for HGT content, the sequence divergence allowed for masking at a particular locus, and a unique name for the project.

When entering the minimum sequence identity accepted for HGT content, entering '95' will provide output that shows predicted HGT sequence intervals with >=95% sequence similarity in HGT alignment after masking. Please adjust this value based on your own emprical observations of genetic similarity of target HGT sequence.

When entering the sequence divergence allowed for masking at a particular locus, entering '10' will mask predicted HGT alignments only if masking alignment is within 10% identity of HGT alignment. So if a certain locus in your HGT alignment file shows 98% sequence identity, that locus will be masked if it shows >=88% sequence identity in your masking alignment. This parameter is used to filter out masking sequence that is not directly related by descent to the predicted HGT sequence of interest. For example, if your HGT alignment shows a highly conserved lav block from your genome category 1 vs. 3 alignment of 98%, you don’t want to mask this sequence if your 1 vs. 2 alignment only shows sequence similarity of 55% at that same locus. Genomes from categories 1 and 2 should be more similar on average than 1 and 3, so filtering out based on the 55% similar block would lead to a false negative HGT detection.  In general, larger values for this parameter will results in more type II errors and smaller values will result in more type I errors regarding detection of HGT blocks.  However, assuming your initial strain/species choice has genome categories 1 and 2 more genetically similar than categories 1 and 3, lower values for this parameter are preferred.  

The output for this pipeline will have the suffix ‘_masked_lav.txt’ (ex - for project name 'foo', output will be type 'foo_masked_lav.txt’) and will be found in a new directory with your given project name. It will be in the same format as the ‘lav_expansion.txt’ pipline output, although it will be filtered according to your parameters to include predicted HGT content. Please use output with caution, and we recommend you confirm predicted HGT content through other bioinformatic or empirical means.

Example data
------------
In the repository, you will find a folder with example data. These data are fasta files from the three genome categories required to run the either the lav expansion pipeline or entire HGT detection pipeline.  The files are in fasta format, where ‘category1.fasta’ contains a single contig with HGT content and ‘category2.fasta’ and ‘category3.fasta’ contain entire genomes in contig format. Try running the pipeline with LASTZ lav output created using these files and play around with the parameters to get a feel for how they affect your final output.

Final notes
-----------
Thank you for using the LASTZ LAV expansion pipelines.  I hope you found them useful. For any questions, comments, or suggestions, feel free to reach out.

Josh Faber-Hammond
jfaberha@gmail.com
