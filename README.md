# PSSS

The Department of Energy Joint Genome Institute (DOE JGI) and National Center for Biotechnology Information (NCBI) are collaborating to sponsor a team at the BioIT hackathon in Boston on May 15-16.  Participants will interrogate SRA using appropriate tools (Pebblescout and SourMash) to identify reads of interest.  They can then explore these results by examining metadata of the reads, aligning pre-assembled contigs with BLAST, and identifying conserved domains on the contigs.  Read an overview of this project [here](https://github.com/ncbi/PSSS-Bytes2Biology/wiki).

This document describes the tools and steps in the workflow for this hackathon team.

# Searching the Sequence Read Archive (SRA).

SRA is big with more than 36 PB of data.  Tools such as PebbleScout and SourMash use kmers in a query to select SRA runs that might be of interest. These programs do not return alignments but instead a ranked list of matches that can be further explored.  

We describe how to run Pebblescout and SourMash.  Pebblescout is best at ??? and SourMash is good at ???

## Pebblescout
We will use this [page](https://pebblescout.ncbi.nlm.nih.gov/) to search SRA with Pebblescout.  Pebblescout can run in three differnet modes that are Profile, Summary, and Detailed.  Here, we discuss the Summary mode.  An input file can be uploaded to this page or it can be pasted into a text box.  Results can be viewed on this page or a TSV file downloaded.  The fields in the TSV file are described in the Pebblescout [documentation](https://pebblescout.ncbi.nlm.nih.gov/?view=doc).  The TSV file has one line per read identified and three scores are included in that line: raw score, percent coverage, and the Pebblescout score.  The percent coverage and the Pebblescout score (a normalized score) are probably the most interesting for this work.  

Pebblescout output (truncated) for NC_003715.1:1330-2928:

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
Now that you have the matching reads from your query, you can explore the metadata for your reads with AWS Athena.

Much more!

# Aligning with ElasticBLAST

ElasticBLAST is a cloud native application that runs the command-line BLAST+ executables for you in the cloud.  It can bring up multiple instances to run a large number of searches quickly, and it handles much of the complexity of running BLAST on the cloud.  This includes selecting instance types, starting them, populating them with the BLAST databases and sofware, scheduling the seaches, and deallocating the resources when the work is done.

ElasticBLAST can search an NCBI provided database or one that you provide.  

The section below presents a quick overview of ElasticBLAST that is required for the hackathon.  The full ElasticBLAST documentation is [here](https://blast.ncbi.nlm.nih.gov/doc/elastic-blast/elasticblast.html). 

First, you need to install and enable ElasticBLAST:

```
[ -d .elb-venv ] && rm -fr .elb-venv
python3 -m venv .elb-venv
source .elb-venv/bin/activate
pip install wheel
pip install elastic-blast==1.0.0
```
Below is a command that runs an ElasticBLAST search of your query against a database.  The query is already in an S3 bucket and a database provided by the NCBI is used.  You will need to substitute your real results bucket name and replace REPLACEME with your own token (which should be unique for each search).  This command should take less than 10 minutes to run.

```
elastic-blast submit --query s3://elasticblast-jgiworkshop-394212713216/R219596/ETNvirmetaSPAdes_5/IMG_Data/184068.assembled.fna --db ref_viruses_rep_genomes --program blastn --num-nodes 2 --results s3://elasticblast-USERNAME/results/REPLACEME/ -- -evalue 0.00001 -max_target_seqs 500  -perc_identity 95 -outfmt "6 std qlen slen staxid ssciname"
```

This may take a few minutes to return while it submits.  After the submission, you may monitor the results by running:
```
elastic-blast status --results s3://elasticblast-USERNAME/results/REPLACEME
```

When elastic-blast reports that all batches have finished, you can retrieve resutls with the command below.

```
export YOUR_RESULTS_BUCKET=s3://elasticblast-USERNAME/results/REPLACEME/
aws s3 cp ${YOUR_RESULTS_BUCKET}/ . --exclude "*" --include "*.out.gz" --recursive
```

You can filter these results for alignments longer than 100 bases with:

```
gunzip -c batch*virus*.gz | awk '{if($4 > 100) print $0}'
```

The search above used an NCBI provided database.  You can also upload your own database to a cloud bucket and use that.  Instructions are [here](https://blast.ncbi.nlm.nih.gov/doc/elastic-blast/configuration.html#blast-database).







