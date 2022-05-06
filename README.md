# Lab Journal 

## 28.03.22

Conda environment for project set up: researchProject
Samtools installed.
Example data copied from hard drive: S000001, S000002
into documents folder on windows compartment.

Notes for tomorrow:
- Hand in Fire safety sheet
- Clean Keyboard
- Learn about SAM and BAM
- Check RefSeq and GENCODE
- Finish review

### Big Goal for the week:
Understand how to recognize splicing sites

## 29.03.22

Inspecting the BAM file for sample S000001:
1. Turn them into SAM file.

```shell
samtools view -h -o alignment.sam alignment.bam
```

3. Check command line it was generated with:

```shell
/usr/local/packages/hisat/2.1.0/bin/hisat2-align-s --wrapper basic-0 -p 16 -q --fr --phred33 -t --dta --dta-cufflink --new-summary --non-deterministic --novel-splicesite-outfile aligned/splicesites.tsv --rna-strandness RF --summary-file aligned/summary.txt --rg PL:Illumina --rg CN:SCANB-prim --rg-id S000001.l.r.m.c.lib.g --rg PU:C1PCYACXXD1U1NACXX --rg SM:S000001 --rg LB:S000001.l.r.m.c.lib -x /reference/hg38/hg38.analysisSet_gencode27_snp150/genome_snp_tran --passthrough -1 /tmp/9982.inpipe1 -2 /tmp/9982.inpipe2"
```

<p>
Understanding it:
hisat2-align: executable alignment script.
<br>--wrapper:
<br>-p 16: parallel threads, here 16
<br>-q: Query input files are fastq
<br>--fr: The upstream/downstream mate orientations for a valid paired-end alignment against forward referenced strand. Here: There is a candidate paired-end alignment where mate 1 appears upstream of the reverse complement of mate 2 and if the fragment length constraints are met, that alignment is valid.
<br>--phred33: Qualities are Phred+33
<br>-t: Output the time required.
<br>--dta: Report alignments tailored for transcript assemblers. Hisat2 will require longer anchor lengths for de novo discovery of splice sites. This leads to fewer alignments with short-anchors, which helps transcript assemblers improve significantly in computation and memory usage.
<br>--dta-cufflinks: Alignments tailord for cufflinks. Looks for novel splice sites with three signals.
<br>--new-sumary: result printing style.
<br>--non-deterministic: HISAT2 will not necessarily report same alignment for two identical reads.
<br>--RF: paired-end reads, every alignment has an XS attribute tag: + means the read belongs to transcript on + strand of genome, - means the read belongs to a transcript on the - strand of the genome.
<br>--summary-file: print alignment summary to this file.
<br>--rg: specify field names for SAM output file. requires rg-id to be set, as all fields are associated with the ID.
<br>--passthrough
<br>--novel-splicesite-outfile: reports a list of splicesites in the specified tsv file.
<br>-x specify index file.
<\p>

```shell
@PG     ID:MarkDuplicates       PN:MarkDuplicates       VN:1.128-4(154e938885f271b2744d8d24fa22810a54752c8e_1424761755) CL:picard.sam.markduplicates.MarkDuplicates MAX_FILE_HANDLES_FOR_READ_ENDS_MAP=2000 INPUT=[aligned/alignment.bam] OUTPUT=aligned/alignment.bam.tmp_picard METRICS_FILE=aligned/alignment_picardmetrics.csv REMOVE_DUPLICATES=false ASSUME_SORTED=true VERBOSITY=WARNING QUIET=true    MAX_SEQUENCES_FOR_DISK_READ_ENDS_MAP=50000 SORTING_COLLECTION_SIZE_RATIO=0.25 PROGRAM_RECORD_ID=MarkDuplicates PROGRAM_GROUP_NAME=MarkDuplicates DUPLICATE_SCORING_STRATEGY=SUM_OF_BASE_QUALITIES READ_NAME_REGEX=[a-zA-Z0-9]+:[0-9]:([0-9]+):([0-9]+):([0-9]+).* OPTICAL_DUPLICATE_PIXEL_DISTANCE=100 VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false
```

MarkDuplicates belongs to the picard software environment. It is used with paired-end reads to remove duplicates, as it is unlikely that the same fragment is sequenced twice.

The first 195 lines are telling us what chromosomes/scaffolds the data was compared to.
To figure out what they are, you used:
https://genome.ucsc.edu/cgi-bin/hgTracks?db=hg38&chromInfoPage= (scaffold list for hg38)
https://www.gencodegenes.org/human/ (To download corresponding gff3 file, used PRI due to scaffolds used for alignment)
http://gmod.org/wiki/GFF3 (more on the gff3 format, that we just downloaded the annotation file in.)

  Extract one gene to try out a few things:

  ```shell
  samtools view -h -o gene_DDX11L1.sam alignment.bam "chr1:11869-14409"
  ```

  Made a first python script to attempt extracting splicing sites corresponding to this gene and compare them to the exons of this gene found in the gff file.

 ## 30.03.22
  Continued working on script.

## 31.03.22
  Same but for estrogen recepter: ESR1 gene. This way we can compare it to helenas output.
  ```shell
    samtools view -h -o gene_ESR1.sam alignment.bam "chr6:151690496-152103274"
  ```

## 01.04.22
```shell
  python splicesites_in_bam.py -b alignment.bam -o ESR1_h.bed
```

The script now detects the splice junctions of the estrogen receptor, using a BAM file as input. It produces a BED file, though some columns have no value yet, as the comparison to known splice junctions is not done yet.
What it lacks is generalization for other genes, chromosomes, genomes as well as the comparison to known splice junctions using the ucsc bed file containing gene predictions.
So the next step is to implement the comparison.
Which i tried, see file submission from 1.4. afternoon, but it is not working well yet.

## 04.04.22
The splicesites_in_bam.py is now working for extracting the splicesites for ESR1 from the bam file, as well as extracting the splicesites predicted from ucsc and comparing the two. The block size for the novel splice sites is set to 1,1, as we cannot make an accurate prediction based on the alignment data.
  
Red bed entries are novel, green are only predicted not found, and blue are predicted and found. A table with 26 entries was made this way.

Still to do:
  - Implement for reverse strand
  - allow coordinate input (so it works for other genes than ESR1 but also for the whole genome.
  - Document
  - title of BED file
  - document code
  - Read up on PSI (monikas report)

## 11.04.22
The splicesites_in bam has been rewritten to splice_junctions.py. It now works for genes (given by chromosome+coordinates) as well as for the whole genome and produces a comparison bed file. An attempt has been made in the same script to classify splice junctions found in the gencode bed file, but since this part is currently not in use, it is commented out.
  
The script is run with:
```shell
python splice_junctions.py -b alignment.bam -db hg38_GENCODE39.bed -o test_out.txt -c "chr6:151690496-152103274"
```
  
## 22.04.22
New script that finds alternative splicing events in the gencode bed file and then finds how many reads they have in the bam file and calculates psi scores for each event. Rn this is only implemented for Casette exons. The Psi scores dont work yet.

It is again runable on a region, or on the total genome.
runs with:
  
```shell
python events_gencode.py -b alignment.bam -gc geneID_hg38_GENCODE39.tsv -db hg38_GENCODE39.bed -o test_out.txt
```

## 27.04.22
opening the bam file in the events_gencode script is now done with pysam instead of a subprocess, as running the linux subprocess (samtools) got killed when not inputing coordinates. It should also be replaced in the splice_junctions script. 
  
Still no reasonable PSI scores. Problems with the iteration.
  
Update: The splice_junction script now also uses pysam instead of subprocess.
  
Update 2: The events_gencode script now excludes second reads of read pairs mapping to the same junction from the counts for PSI scores.
  
## 6.5.22
After managing to obtain reasonable PSI scores for ESR1 on the plus strand, this week was spent on trying to implement the same for genes on the reverse strand. So far not successfully. But in the process of trying to figure out why, the script was reworked. 
Command line for plus strand:

```shell
python events_gencode.py -b alignment.bam -gc geneID_hg38_GENCODE39.tsv -db hg38_GENCODE39.bed -o CE_ESR1.txt -c "chr6:151600000-152150000" 
```

Command line for minus strand:

```shell
python events_gencode.py -b alignment.bam -gc geneID_hg38_GENCODE39.tsv -db hg38_GENCODE39.bed -o SYNE1_event_out.txt -c "chr6:152380546-152381564" 
```

