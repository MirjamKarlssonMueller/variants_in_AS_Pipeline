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
--wrapper:
-p 16: parallel threads, here 16
-q: Query input files are fastq
--fr: The upstream/downstream mate orientations for a valid paired-end alignment against forward referenced strand. Here: There is a candidate paired-end alignment where mate 1 appears upstream of the reverse complement of mate 2 and if the fragment length constraints are met, that alignment is valid.
--phred33: Qualities are Phred+33
-t: Output the time required.
--dta: Report alignments tailored for transcript assemblers. Hisat2 will require longer anchor lengths for de novo discovery of splice sites. This leads to fewer alignments with short-anchors, which helps transcript assemblers improve significantly in computation and memory usage.
--dta-cufflinks: Alignments tailord for cufflinks. Looks for novel splice sites with three signals. 
--new-sumary: result printing style.
--non-deterministic: HISAT2 will not necessarily report same alignment for two identical reads.
--RF: paired-end reads, every alignment has an XS attribute tag: + means the read belongs to transcript on + strand of genome, - means the read belongs to a transcript on the - strand of the genome.
--summary-file: print alignment summary to this file.
--rg: specify field names for SAM output file. requires rg-id to be set, as all fields are associated with the ID. 
--passthrough
--novel-splicesite-outfile: reports a list of splicesites in the specified tsv file.
-x specify index file.
<p>
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
