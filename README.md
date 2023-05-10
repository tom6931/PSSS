# PSSS

The Department of Energy Joint Genome Institute ([DOE JGI](https://jgi.doe.gov/)) and National Center for Biotechnology Information ([NCBI](https://www.ncbi.nlm.nih.gov/)) are collaborating to sponsor a team at the [BioIT hackathon](https://www.bio-itworldexpo.com/fair-data-hackathon) in Boston on May 15-16.  Participants will interrogate SRA using appropriate tools (Pebblescout and SourMash) to identify reads of interest.  They can then explore these results by examining metadata of the reads, aligning pre-assembled contigs with BLAST, and identifying conserved domains on the contigs.  Read an overview of this project [here](https://github.com/ncbi/PSSS-Bytes2Biology/wiki).


This document describes the tools and steps in the workflow for this hackathon team.

# Workflow Overview

This is a high-level outline of the workflow for this hackathon team.  The exact steps will depend upon discussions with the team and the interest of the participants.  Bullet points with the same indentation are independent of each other.  (FIXME: replace with figure)


* Search sequence data against SRA with Pebblescout or SourMash
  * Investigate metadata for the SRA reads identified
    * Using AWS Athena
  * Identify JGI contigs that correspond to your SRA runs
    * Align contigs with ElasticBLAST against a database
    * Find conserved domains on those contigs with RPSTBLASTN using ElasticBLAST
    * Download the FASTA for the SRA runs and align that back against the contigs
    * Examine JGI contig metadata (FIXME: does it exist and fits in hackathon etc)
    * Use the GFF file for a contig to explore features on contigs and neighboring genes to the target gene.
  


# Computing and Storage

For this hackathon we’ve set up an EC2 instance in AWS for each participant.

Each user’s instance has 194G of storage in /home/ubuntu

Each user’s instance also has 217GB of SSD in /home/ubuntu/nvme_ssd

You will also have an S3 bucket set up for you. These buckets can persist once the computing instances are shut down.

# Searching the Sequence Read Archive (SRA).

SRA is big with more than 36 PB of data.  Tools such as PebbleScout and SourMash use kmers in a query to select SRA runs that might be of interest. These programs do not return alignments but instead a ranked list of matches that can be further explored.  

We describe how to run Pebblescout and SourMash.  Pebblescout requires a query that is at least 42 bases long.  SourMash requires one or more genomes of at least 10,000 bases as a query (https://dib-lab.github.io/2022-paper-branchwater-software/)

## Pebblescout
We will use this [page](https://pebblescout.ncbi.nlm.nih.gov/) to search SRA with Pebblescout.  Pebblescout can run in three different modes that are Profile, Summary, and Detailed.  Here, we discuss the Summary mode.  An input file can be uploaded to this page or it can be pasted into a text box.  Results can be viewed on this page or a TSV file downloaded.  The fields in the TSV file are described in the Pebblescout [documentation](https://pebblescout.ncbi.nlm.nih.gov/?view=doc).  The TSV file has one line per accession identified and three scores are included in that line: raw score, percent coverage, and the Pebblescout score.  The percent coverage and the Pebblescout score (a normalized score) are probably the most interesting for this work.  

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

You will use Mastiff for these searches.  Mastiff is a tool in the sourmash suite developed in the lab of C. Titus Brown at UC Davis..  Follow the steps below to download and then run mastiff.  Mastiff only needs to be downloaded the first time.

```
curl -o mastiff -L https://github.com/sourmash-bio/mastiff/releases/latest/download/mastiff-client-x86_64-unknown-linux-musl
chmod +x mastiff
./mastiff -o matches.csv  TOBG_NP-110.fna
```

You can check the results of the matches with:

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
The NCBI has deposited metadata for SRA entries into AWS Athena, making it available to the bioinformatics community.  
You can read about this at https://www.ncbi.nlm.nih.gov/sra/docs/sra-athena/ and https://www.ncbi.nlm.nih.gov/sra/docs/sra-athena-examples/ (FIXME)

JGI contig metadata? (FIXME)

# Obtaining the contigs for a SRA accessions

Pebblescout and Sourmash return a list of accessions and scores for how well those accessions matched the query.  The JGI has assembled contigs based on SRA reads.  These contigs can be used to better understand the contents of the SRA runs through alignments.  

You can find the contigs at : s3://elasticblast-jgiworkshop-394212713216  To list this directory, use:

```
aws s3 ls s3://elasticblast-jgiworkshop-394212713216
```
You should look for the contigs in directories called IMG_DATA, e.g. s3://elasticblast-jgiworkshop-394212713216/R219601/NitDOMTargeteWGA_16/IMG_Data/

To copy contigs from a bucket to your instance, you can use the AWS s3 command.  An example command is:

```
aws s3 cp s3://elasticblast-jgiworkshop-394212713216/R219601/NitDOMTargeteWGA_16/IMG_Data/ . --recursive 
```

# Aligning with ElasticBLAST

ElasticBLAST is a cloud native application that runs the command-line BLAST+ executables for you in the cloud.  It can bring up multiple instances to run a large number of searches quickly, and it handles much of the complexity of running BLAST on the cloud.  This includes selecting instance types, starting them, populating them with the BLAST databases and sofware, scheduling the seaches, and deallocating the resources when the work is done.

ElasticBLAST can search an NCBI provided database or one that you provide.  

We describe how to run ElasticBLAST in a couple of different modes.  The full ElasticBLAST documentation is [here](https://blast.ncbi.nlm.nih.gov/doc/elastic-blast/elasticblast.html). 

First, you need to install and enable ElasticBLAST:

```
[ -d .elb-venv ] && rm -fr .elb-venv
python3 -m venv .elb-venv
source .elb-venv/bin/activate
pip install wheel
pip install elastic-blast==1.0.0
```

## Running BLASTN (DNA-DNA) 

Below is a command that runs an ElasticBLAST search of your query against a database.  The query is already in an S3 bucket and an NCBI database is used.  You will need to substitute your real results bucket name and replace REPLACEME with your own token (which should be unique for each search).  This search should take less than 10 minutes.  This command customizes the BLAST output with the -outfmt flag and some custom fields.  You can read more about the -outfmt flag [here](https://www.ncbi.nlm.nih.gov/books/NBK569862/)

```
elastic-blast submit --query s3://elasticblast-jgiworkshop-394212713216/R219596/ETNvirmetaSPAdes_5/IMG_Data/184068.assembled.fna --db ref_viruses_rep_genomes --program blastn --num-nodes 2 --results s3://elasticblast-USERNAME/results/REPLACEME/ -- -evalue 0.00001 -max_target_seqs 500  -perc_identity 95 -outfmt "6 std qlen slen staxid ssciname"
```

This command will take a few minutes to return while it submits.  After it returns, you may monitor the results by running:
```
elastic-blast status --results s3://elasticblast-USERNAME/results/REPLACEME
```

When elastic-blast reports that all batches have finished, you can retrieve results with the command below.

```
export YOUR_RESULTS_BUCKET=s3://elasticblast-USERNAME/results/REPLACEME/
aws s3 cp ${YOUR_RESULTS_BUCKET}/ . --exclude "*" --include "*.out.gz" --recursive
```
These results are a tabular report with the fields (FIXME)

You can filter these results for alignments longer than 100 bases with:

```
gunzip -c batch*virus*.gz | awk '{if($4 > 100) print $0}'
```

The search above used an NCBI provided database.  You can get a listing of all the NCBI provided databases by running:

```
update_blastdb.pl --source aws --showall pretty
```

You can also upload your own database to a cloud bucket and use that.  Instructions are [here](https://blast.ncbi.nlm.nih.gov/doc/elastic-blast/configuration.html#blast-database).

## Running RPSTBLASTN (protein domains with DNA query)

ElasticBLAST can also run the RPSTBLASTN program which translates a query in six frames and performs a profile search against the Conserved Domain Database ([CDD](https://www.ncbi.nlm.nih.gov/Structure/cdd/cdd.shtml)).  The command below performs this search on the same set of contigs as above.  This search uses 20 instances and should take about 50 minutes.

```
elastic-blast submit --query s3://elasticblast-jgiworkshop-394212713216/R219596/ETNvirmetaSPAdes_5/IMG_Data/184068.assembled.fna --db s3://elasticblast-test/db/CDD_TAX/cdd_tax --program rpstblastn --num-nodes 20 --results s3://elasticblast-USERNAME/results/REPLACEME/ -- -evalue 0.00001 -max_target_seqs 500 -outfmt "6 std qlen slen staxid sskingdoms ssciname stitle"
```
When elastic-blast reports that all batches have finished, you can retrieve results with a command similar to the one given above for the BLASTN search.  These results are a tabular report with a field that include taxonomy and title information for the domain.  The taxonomy information here describes where the conserved domain is found taxonomically.  That is, if the sskingdoms is Bacteria and the ssciname is Enterobacteriaceae, then it is found in Enterobacteriaceae.  In some cases, a broad taxonomic range is returned (e.g., the conserved domain is found in bacteria).  A very broad range may also be returned.    

## Aligning reads to the JGI contigs

You can also use ElasticBLAST to align the SRA reads corresponding to a contig back to the contig to see how well they align (e.g., how well the contig represents the SRA accession).  This will allow you to answer questions like

* How much of the contig is covered by the reads in the SRA accession.
* What percentage of the reads are aligned. 

There are a few steps here.  First, you need to download the contigs to your instance and make a BLAST database out of them. Second, you upload that database to a cloud bucket. Next, you can run ElasticBLAST using the pre-formatted FASTA for the SRA accession matching the contigs. 

The examples below use the ETNvirmetaSPAdes_5 contigs and the corresponding SRA accession.  You will need to substitute the contigs and SRA accession you want to align as well as the path to your own S3 bucket.

To download the contigs, make a BLAST database, and upload them to a cloud bucket, you should run these steps:

```
aws s3 cp s3://elasticblast-jgiworkshop-394212713216/R219596/ETNvirmetaSPAdes_5/IMG_Data/184068.assembled.fna .
makeblastdb -in 184068.assembled.fna -parse_seqids -title "ETNvirmetaSPAdes_5 contigs (184068)" -dbtype nucl -out 184068.assembled
aws s3 cp 184068.assembled.n* s3://elasticblast-USERNAME/DB/ETNvirmetaSPAdes_5/
```

You can then align the SRA reads against the contigs with this command:

```
elastic-blast submit --query s3://FIXME --db s3://elasticblast-USERNAME/DB/ETNvirmetaSPAdes_5/184068.assembled --program blastn --num-nodes 2 --results s3://elasticblast-USERNAME/results/REPLACEME/ -- -evalue 0.00001 -max_target_seqs 500  -perc_identity 95 -outfmt "6 std qlen slen"
```

Copy the results from the bucket to your instance using the same procedure as above.

## Aligning reads to your query

You can also use ElasticBLAST to align the SRA reads back to the query (that was used with SourMash or Pebblescout).  This will allow you to calculate the query coverage from the reads as well as calculate what percentage of the reads aligned to the query.

There are a few steps here.  First, you need to make a BLAST database out of your query. Second, you upload that database to a cloud bucket. Next, you can run ElasticBLAST using the pre-formatted FASTA for the SRA accession that your query identified. 

You can make a database from your query and upload it to a cloud bucket using the commands below.  You will need to substitute your real query name for MYQUERY as well as use your real bucket name.

```
makeblastdb -in MYQUERY.fna -parse_seqids -title "MYQUERY" -dbtype nucl -out MYQUERY
aws s3 cp MYQUERY.n* s3://elasticblast-USERNAME/DB/MYQUERY
```

You can then align the SRA reads against your query with this command (again replacing MYQUERY with your actual query name).

```
elastic-blast submit --query s3://FIXME --db s3://elasticblast-USERNAME/DB/MYQUERY --program blastn --num-nodes 2 --results s3://elasticblast-USERNAME/results/REPLACEME/ -- -evalue 0.00001 -max_target_seqs 500  -perc_identity 95 -outfmt "6 std qlen slen"
```

Copy the results from the bucket to your instance using the same procedure as above.





