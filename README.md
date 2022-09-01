# Annotate_variants
Annotate variants from a VCF file using VEP API and generates a TSV output with annotations
## Description

The script annotate variants from a single sample VCF (generated by PLATYPUS) using VEP API
### Assumptions

--annotates VCF generated by platypus tool with a single sample

--script is tested for VCF version 4.0 (output from platypus)

--human genome version used is GRCh37

--All the variants to be annotated in the VCF file should be prefiltered for quality (no quality check is in place in the current version)

--cyvcf2, the package used for reading VCF, checks for the sanity of VCF and 
	issues a warning on missing chromosome information in the VCF header
	
--prints the output as a TSV file

--cyvcf2 warns " bcftools [W::bcf_hdr_check_sanity] GL should be declared as Number=G"
 	see the thread regarding the fix: https://github.com/samtools/bcftools/issues/1003
 	likely due to multi-allelic variants for which GL is not determined


### Key strength/Features

--if needed to include additional annotations, it can be added with minimal effort

--reports minor allele frequency only if the minor allele match with the alternate allele

--multiple alternate alleles reported for the same reference variant are also annotated

--prints a log file of IDs failed to get annotated by VEP

--if failed to get annotated from VEP, information collected from VCF will be in the final TSV out file

--most severe consequence also reported in addition individual transcript consequence

--Raises error if the requested files are not found

--cyvcf2 package checks for VCF sanity (reports missing chromosome information in VCF header)

--if generated duplicate hgvs ID, those are captured

--API request is made in bulk (max permitted)

### Limitations
--Time consuming, however can be improved choosing the fields to be returned with API instead of requesting the whole data available for the variant to be returned.

### pre-requisites
--cyvcf2

```
pip install cyvcf2
```	

Ref: http://brentp.github.io/cyvcf2/index.html

https://academic.oup.com/bioinformatics/article/33/12/1867/2971439

--requests

```
pip install requests
```
Ref: https://pypi.org/project/requests/

### Usage instructions

```
$ python Annotate_Variants_VEP.py -h will display help options

usage: Annotate_Variants_VEP.py [-h] -v VCFFILE [-o OUTFILE]
                                                    
optional arguments:
  -h, --help            show this help message and exit
  -o OUTFILE, --outfile OUTFILE
                        output TSV file with annotation

Required arguments:
 
  -v VCFFILE, --vcffile VCFFILE
                        vcffile to be annotated                      
```
#### Sample Command Line below
```             
python Translate_transcript_to_genomic_cord_v1.0.py --vcffile test-data.vcf
```
#### sample VCF data (not including headers)
```
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	sample
1	1158631	.	A	G	2965	PASS	BRF=0.16;FR=1.0000;HP=1;HapScore=1;MGOF=3;MMLQ=33;MQ=59.75;NF=89;NR=67;PP=2965;QD=20;SC=CACTTTCCTCATCCACTTTGA;SbPval=0.58;Source=Platypus;TC=160;TCF=90;TCR=70;TR=156;WE=1158639;WS=1158621	GT:GL:GOF:GQ:NR:NV	1/1:-300.0,-43.88,0.0:3:99:160:156
1	1246004	.	A	G	2965	PASS	BRF=0.09;FR=1.0000;HP=6;HapScore=1;MGOF=5;MMLQ=32;MQ=59.5;NF=101;NR=47;PP=2965;QD=20;SC=ACAGGTACGTATTTTTCCAGG;SbPval=0.62;Source=Platypus;TC=152;TCF=101;TCR=51;TR=148;WE=1246012;WS=1245994	GT:GL:GOF:GQ:NR:NV	1/1:-300.0,-41.24,0.0:5:99:152:148
1	1249187	.	G	A	2965	PASS	BRF=0.16;FR=1.0000;HP=3;HapScore=1;MGOF=3;MMLQ=34;MQ=59.72;NF=78;NR=57;PP=2965;QD=20;SC=TCCTCTGCACGAAAGTCTTGC;SbPval=0.53;Source=Platypus;TC=137;TCF=79;TCR=58;TR=135;WE=1249195;WS=1249177	GT:GL:GOF:GQ:NR:NV	1/1:-300.0,-37.63,0.0:3:99:137:135
```
#### Sample output
```
hgvsID	Chrom	position	Ref	Alt	Type_of_Variation	variant_depth	Total_read_depth	perc_variant_depth	perc_ref_depth	most_severe_consequence	minor_allele_frequency	genes	genes_effect
1:g.1158631A>G	1	1158630	A	G	substitution	156	160	97.5	2.5	synonymous_variant	NULL	SDF4	protein_coding:SDF4:synonymous_variant;processed_transcript:SDF4:downstream_gene_variant;nonsense_mediated_decay:SDF4:synonymous_variant NMD_transcript_variant;retained_intron:SDF4:upstream_gene_variant
1:g.1246004A>G	1	1246003	A	G	substitution	148	152	97.37	2.63	splice_polypyrimidine_tract_variant	NULL	CPSF3L;ACAP3;PUSL1	retained_intron:CPSF3L:downstream_gene_variant;protein_coding:ACAP3:upstream_gene_variant;nonsense_mediated_decay:ACAP3:upstream_gene_variant;protein_coding:PUSL1:splice_polypyrimidine_tract_variant intron_variant;protein_coding:CPSF3L:downstream_gene_variant;nonsense_mediated_decay:CPSF3L:downstream_gene_variant;processed_transcript:ACAP3:upstream_gene_variant;processed_transcript:CPSF3L:downstream_gene_variant;retained_intron:PUSL1:downstream_gene_variant;processed_transcript:PUSL1:splice_polypyrimidine_tract_variant intron_variant non_coding_transcript_variant;retained_intron:ACAP3:upstream_gene_variant;retained_intron:PUSL1:non_coding_transcript_exon_variant
1:g.1249187G>A	1	1249186	G	A	substitution	135	137	98.54	1.46	missense_variant	0.3101	CPSF3L;PUSL1;RP5-890O3.9;ACAP3	retained_intron:CPSF3L:non_coding_transcript_exon_variant;protein_coding:PUSL1:downstream_gene_variant;protein_coding:CPSF3L:synonymous_variant;retained_intron:CPSF3L:downstream_gene_variant;nonsense_mediated_decay:CPSF3L:downstream_gene_variant;sense_intronic:RP5-890O3.9:downstream_gene_variant;nonsense_mediated_decay:CPSF3L:3_prime_UTR_variant NMD_transcript_variant;retained_intron:CPSF3L:upstream_gene_variant;processed_transcript:CPSF3L:non_coding_transcript_exon_variant;retained_intron:PUSL1:downstream_gene_variant;processed_transcript:PUSL1:downstream_gene_variant;retained_intron:ACAP3:upstream_gene_variant;protein_coding:CPSF3L:downstream_gene_variant;nonsense_mediated_decay:CPSF3L:missense_variant NMD_transcript_variant
1:g.1261824G>C	1	1261823	G	C	substitution	134	136	98.53	1.47	intron_variant	NULL	CPSF3L;TAS1R3;GLTPD1	retained_intron:CPSF3L:upstream_gene_variant;protein_coding:TAS1R3:upstream_gene_variant;protein_coding:GLTPD1:intron_variant;protein_coding:CPSF3L:upstream_gene_variant;nonsense_mediated_decay:CPSF3L:upstream_gene_variant;processed_transcript:GLTPD1:intron_variant non_coding_transcript_variant;processed_transcript:CPSF3L:upstream_gene_variant
```
#### Sample log file
```
1:g.6579521G>C
1:g.6586070C>T
```

	
#### Test Configuration
* macbookpro(Monterey)
* Python: Python 3.8.9
* requests 2.28.1
* cyvcf2 0.30.16

--Tested with test VCF file (provided) took 1058.3 +/- 23 secs and 55.5 +/- 2 MByte memory (based on three runs)

--Time consuming step is sending and gathering API request as can be done in only 200 at a time
	one way to improve the performance is to request only the fields interested to be 
	returned instead of requesting the whole data available for the variant to be returned.

