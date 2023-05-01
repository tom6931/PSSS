# PSSS

The Department of Energy Joint Genome Institute (DOE JGI) and National Center for Biotechnology Information (NCBI) are collaborating to sponsor a team at the BioIT hackathon in Boston on May 15-16.  Participants will interrogate SRA using appropriate tools (Pebblescout and SourMash) to identify reads of interest.  They can then explore these results by examining metadata of the reads, aligning pre-assembled contigs with BLAST, and identifying conserved domains on the contigs.

This documents describes the tools and steps in the workflow for this hackathon team.

# Searching the Sequence Read Archive (SRA).

SRA is big wih close to 18 PB of data.  Tools such as PebbleScout and SourMash match kmers found in a query to kmers in SRA to select SRA runs that might be of interest. These programs do not return alignments but instead a ranked list of matches that can be further explored.  

We describe how to run Pebblescout and SourMash.  Pebblescout is best at ??? and SourMash is good at ???

## Pebblescout
We will use this [page](https://pebblescout.ncbi.nlm.nih.gov/) to search SRA with Pebblescout.  Pebblescout can run in three differnet modes that are Profile, Summary, and Detailed.  Here, we discuss the Summary mode.  An input file can be uploaded to this page or it can be pasted into a text box.  Results can be viewed on this page or a TSV file downloaded.  The fields in the TSV file are described in the Pebblescout [documentation](https://pebblescout.ncbi.nlm.nih.gov/?view=doc).  The TSV file has one line per read identified and three scores are included in that line: raw score, percent coverage, and the Pebblescout score.  The percent coverage and the pebblescout score (a normalized score) are probably the most interesting for this work.  

Example output from Pebblescout is (query was NC_003715.1:1330-2928):

```
QueryID	SubjectID	RawScore	%coverage	PBSscore	BioSample	Title
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8729122	166.83	95.98	95.98	SAMN11127329	Next-generation sequencing of double stranded RNA is dramatically improved by treatment with an inexpensive reagent
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8729123	165.83	95.40	95.40	SAMN11127328	Next-generation sequencing of double stranded RNA is dramatically improved by treatment with an inexpensive reagent
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8729125	163.83	94.25	94.25	SAMN11127326	Next-generation sequencing of double stranded RNA is dramatically improved by treatment with an inexpensive reagent
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8729124	150.84	86.78	86.78	SAMN11127327	Next-generation sequencing of double stranded RNA is dramatically improved by treatment with an inexpensive reagent
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR12063609	149.84	86.21	86.21	SAMN15332863	Kettle_Holes_BIBS_Metatranscriptomics
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR12063614	134.85	77.59	77.58	SAMN15332859	Kettle_Holes_BIBS_Metatranscriptomics
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8663624	132.86	76.44	76.43	SAMN11050324	Seasonal Dynamics of DNA and RNA Viral Bioaerosol Communities in a Daycare Center
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8663609	103.88	59.77	59.76	SAMN11050330	Seasonal Dynamics of DNA and RNA Viral Bioaerosol Communities in a Daycare Center
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8663608	78.92	45.40	45.40	SAMN11050331	Seasonal Dynamics of DNA and RNA Viral Bioaerosol Communities in a Daycare Center
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8663603	76.91	44.25	44.25	SAMN11050316	Seasonal Dynamics of DNA and RNA Viral Bioaerosol Communities in a Daycare Center
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8663602	71.92	41.38	41.37	SAMN11050317	Seasonal Dynamics of DNA and RNA Viral Bioaerosol Communities in a Daycare Center
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8663628	67.93	39.08	39.08	SAMN11050328	Seasonal Dynamics of DNA and RNA Viral Bioaerosol Communities in a Daycare Center
NC_003715.1:1330-2928Pseudomonasphagephi-6segmentL,completesequence	SRR8663629	66.93	38.51	38.50	SAMN11050329	Seasonal Dynamics of DNA and RNA Viral Bioaerosol Communities in a Daycare Center
```

You can filter for matches with a Pebblescout score of 90.0 or more with:

```
cat pebblescout-meta-summary.tsv | awk '{if ($5 > 90.0) print $0}' 
```


## Sourmash

In order to run SourMash, you will use a client called Mastiff.  Follow the steps below to download the client and then run SourMash.  The client only needs to be downloaded the first time.

```
curl -o mastiff -L https://github.com/sourmash-bio/mastiff/releases/latest/download/mastiff-client-x86_64-unknown-linux-musl
chmod +x mastiff
./mastiff -o matches.csv  TOBG_NP-110.fna
```

You can check the results of the mathces with:

```
cat matches.csv | more
```

The results are a set of accessions and the corresponding containment scores. For example,

```
SRR5506584,0.9992119779353822,TOBG_NP-110.fna
SRR6056557,0.04018912529550828,TOBG_NP-110.fna
```
You can filter out results below a given containment score with a simple awk command.  The command below prints only lines that have a containment score of 0.99 or greater.

```
cat matches.csv | awk -F , '{ if ($2 > "0.99") print $0}' > extract.csv
```

# Next steps
Once you have the SRA reads you can try a few things:

1. Explore the metadata for the reads you have identified.  This can tell you about blah, blah, blah.
2. Align your query sequence(s) against the JGI contigs for your reads.
3. Align the JGI contigs for your reads against a BLAST database 
3. Find conserved domains on the JGI contigs for your reads.

For the metadata exploration, you can use AWS Athena.  You can use ElasticBLAST for other three.


# Metadata
Now that you have the matching reads from your guqery, you can explore the metadata for your reads with AWS Athena.

Much more!

# Aligning with ElasticBLAST

ElasticBLAST is a cloud native application that runs the command-line BLAST+ executables for you in the cloud.  It can bring up multiple instances to run a large nubmer of searches quickly for you, and it handles a lot of the complexity of running BLAST on the cloud for you.  This includes bringing up instances and populating them with the BLAST databases and the BLAST sofware, scheduling the seaches, and deallocating the resources when the work is done.

ElasticBLAST can be used for the alignment portions of your work, so we discuss here how to do that. 

```
elastic-blast submit --query s3://elasticblast-USERNAME/queries/{}.fa --db s3://elasticblast-USERNAME/custom_blastdb/TOBG_NP-110.fna --program blastn --num-nodes 2 --results s3://elasticblast-USERNAME/results/ -task megablast -word_size 28 -evalue 0.00001 -max_target_seqs 10000000  -perc_identity 97 -outfmt "6 std qlen slen qcovs"
```

You may monitor the results by running:
```
elastic-blast status --results 
```






