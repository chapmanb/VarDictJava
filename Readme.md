# VarDictJava

## Introduction

VarDictJava is a variant discovery program written in Java and Perl. It is a partial Java port of [VarDict variant caller](https://github.com/AstraZeneca-NGS/VarDict). 

The original Perl VarDict is a sensitive variant caller for both single and paired sample variant calling from BAM files. VarDict implements several novel features such as amplicon bias aware variant calling from targeted
sequencing experiments, rescue of long indels by realigning bwa soft clipped reads and better scalability
than many other Java based variant callers. The Java port is around 10x faster than the original Perl implementation.

Please cite VarDict:

Lai Z, Markovets A, Ahdesmaki M, Chapman B, Hofmann O, McEwen R, Johnson J, Dougherty B, Barrett JC, and Dry JR.  VarDict: a novel and versatile variant caller for next-generation sequencing in cancer research. Nucleic Acids Res. 2016, pii: gkw227.

The link to is article can be accessed through: https://academic.oup.com/nar/article/44/11/e108/2468301?searchresult=1

Original coded by Zhongwu Lai 2014.

VarDictJava can run in single sample (see Single sample mode section), paired sample (see Paired variant calling section), or amplicon bias aware modes. As input, VarDictJava takes reference genomes in FASTA format, aligned reads in BAM format, and target regions in BED format.

## Requirements

1. JDK 1.8 or later
2. R language (uses /usr/bin/env R)
3. Perl (uses /usr/bin/env perl)
4. Internet connection to download dependencies using gradle.

To see the help page for the program, run
```
 <path_to_vardict_folder>/build/install/VarDict/bin/VarDict -H.
```
## Getting started

### Getting source code

The VarDictJava source code is located at [https://github.com/AstraZeneca-NGS/VarDictJava](https://github.com/AstraZeneca-NGS/VarDictJava).

To load the project, execute the following command:

```
git clone --recursive https://github.com/AstraZeneca-NGS/VarDictJava.git
```

Note that the original VarDict project is placed in this repository as a submodule and its contents can be found in the sub-directory VarDict in VarDictJava working folder. So when you use `teststrandbias.R` and `var2vcf_valid.pl.` (see details and examples below), you have to add prefix VarDict: `VarDict/teststrandbias.R` and `VarDict/var2vcf_valid.pl.`

### Compiling

The project uses [Gradle](http://gradle.org/) and already includes a gradlew script.

To build the project, in the root folder of the project, run the following command:

```
./gradlew clean installDist
```

To generate Javadoc, in the build/docs/javadoc folder, run the following command:

```
./gradlew clean javadoc
```

To generate release version in the build/distributions folder as tar or zip archive, run the following command:
```
./gradlew distZip
``` 
or
```
./gradlew distTar
```

#### Distribution Package Structure
When the build command completes successfully, the `build/install/VarDict` folder contains the distribution package.

The distribution package has the following structure:
* `bin/` - contains the launch scripts
* `lib/` - has the jar file that contains the compiled project code and the jar files of the third-party libraries that the project uses.

You can move the distribution package (the content of the `build/install/VarDict` folder) to any convenient location.

Generated zip and tar releases will also contain scripts from VarDict Perl repository in `bin/` directory (`teststrandbias.R`, 
`testsomatic.R`, `var2vcf_valid.pl`, `var2vcf_paired.pl`).

You can add VarDictJava on PATH by adding this line to `.bashrc`:
```
export PATH=/path/to/VarDict/bin:$PATH
```
After that you can run VarDict by `Vardict` command instead of full path to `<path_to_vardict_folder>/build/install/VarDict/bin/VarDict`. 

#### Third-Party Libraries
Currently, the project uses the following third-party libraries:
* JRegex (http://jregex.sourceforge.net, BSD license) is a regular expressions library that is used instead of the 
standard Java library because its performance is much higher than that of the standard library.
* Commons CLI (http://commons.apache.org/proper/commons-cli, Apache License) – a library for parsing the command line.
* HTSJDK (http://smtools.github.io/htsjdk/) is an implementation of a unified Java library for accessing common file formats, such as SAM and VCF.
* PowerMock and TestNG are the testing frameworks (not included in distribution, used only in tests).

### Single sample mode

To run VarDictJava in single sample mode, use a BAM file specified without the `|` symbol and perform Steps 3 and 4 
(see the Program workflow section) using `teststrandbias.R` and `var2vcf_valid.pl.`
The following is an example command to run in single sample mode:
  
```
AF_THR="0.01" # minimum allele frequency
<path_to_vardict_folder>/build/install/VarDict/bin/VarDict -G /path/to/hg19.fa -f $AF_THR -N sample_name -b /path/to/my.bam -z -c 1 -S 2 -E 3 -g 4 /path/to/my.bed | VarDict/teststrandbias.R | VarDict/var2vcf_valid.pl -N sample_name -E -f $AF_THR
```

VarDictJava can also be invoked without a BED file if the region is specified in the command line with `-R` option.
The following is an example command to run VarDictJava for a region (chromosome 7, position from 55270300 to 55270348, EGFR gene) with `-R` option:

```
<path_to_vardict_folder>/build/install/VarDict/bin/VarDict  -G /path/to/hg19.fa -f 0.001 -N sample_name -b /path/to/sample.bam  -z -R  chr7:55270300-55270348:EGFR | VarDict/teststrandbias.R | VarDict/var2vcf_valid.pl -N sample_name -E -f 0.001 >vars.vcf
```

In single sample mode, output columns contain a description and statistical info for variants in the single sample. 
See section Output Columns for list of columns in the output. 

### Paired variant calling

To run paired variant calling, use BAM files specified as `BAM1|BAM2` and perform Steps 3 and 4 
(see the Program Workflow section) using `testsomatic.R` and `var2vcf_paired.pl`.

In this mode, the number of statistics columns in the output is doubled: one set of columns is 
for the first sample, the other - for second sample.

The following is an example command to run in paired mode:

```
AF_THR="0.01" # minimum allele frequency
<path_to_vardict_folder>/build/install/VarDict/bin/VarDict -G /path/to/hg19.fa -f $AF_THR -N tumor_sample_name -b "/path/to/tumor.bam|/path/to/normal.bam" -z -F -c 1 -S 2 -E 3 -g 4 /path/to/my.bed | VarDict/testsomatic.R | VarDict/var2vcf_paired.pl -N "tumor_sample_name|normal_sample_name" -f $AF_THR
```

### Amplicon based calling 
This mode is active if the BED file uses 8-column format and the -R option is not specified.

In this mode, only the first list of BAM files is used even if the files are specified as `BAM1|BAM2` - like for paired variant calling.

For each segment, the BED file specifies the list of positions as start and end positions (columns 7 and 8 of 
the BED file). The Amplicon based calling mode outputs a record for every position between start and end that has 
any variant other than the reference one (all positions with the `-p` option). For any of these positions, 
VarDict in amplicon based calling mode outputs the following:
* Same columns as in the single sample mode for the most frequent variant
* Good variants for this position with the prefixes `GOOD1`, `GOOD2`, etc.
* Bad variants for this position with the prefixes `BAD1`, `BAD2`, etc.

For this running mode, the `-a` option (default: `10:0.95`) specifies the criteria of discarding reads that are too 
far away from segments. A read is skipped if its start and end are more than 10 positions away from the segment 
ends and the overlap fraction between the read and the segment is less than 0.95.

### Running Tests
#### Integration testing

The list of integration test cases is stored in files in `testdata/intergationtestcases` directory.
To run all integration tests, the command is:

```$xslt
./gradlew test --tests com.astrazeneca.vardict.integrationtests.IntegrationTest 
```

The results of the tests can be viewed in the `build/reports/tests/index.html` file.

##### User extension of testcases

Each file in `testdata/intergationtestcases` directory represents a test case with input data and expected output

**1.** Create a txt file in `testdata/intergationtestcases` folder. 

The file contains testcase input (of format described in [Test cases file format](Readme.md#CSVhead)) in the first line and expected output in the remaining file part.

**2.** Extend or create [thin-FASTA](Readme.md#thFastahead) in `testdata/fastas` folder.
 
**3.** Run tests.

##### <a name="CSVhead"></a>Test cases file format 

Each input file represents one test case input description. In the input file the first line consists of the following fields separated by `,` symbol:

*Required fields:*
- test case name
- reference name
- bam file name
- chromosome name
- start of region
- end of region

*Optional fields:*
- start of region with amplicon case
- end of region with amplicon case

*Parameters field:*
- the last filed can be any other command line parameters string

Example of first line of input file:

```
Amplicon,hg19.fa,Colo829-18_S3-sort.bam,chr1,933866,934466,933866,934466,-a 10:0.95 -D
Somatic,hg19.fa,Colo829-18_S3-sort.bam|Colo829-19_S4-sort.bam,chr1,755917,756517
Simple,hg19.fa,Colo829-18_S3-sort.bam,chr1,9922,10122,-p
```

##### <a name="testCoverageReporthead"></a>Test coverage Report

To build test coverage report run the following command:

```
./gradlew test jacocoTestReport 
```

Then HTML report could be found in `build/reports/jacoco/test/html/index.html`

##### <a name="thFastahead"></a>Thin-FASTA Format

Thin fasta is needed to store only needed for tests regions of real references to decrease disk usage. Each thin-FASTA file is `.csv` file, each line of which represent part of reference data with information of: 
- chromosome name
- start position of contig
- end position of contig
- and nucleotide sequence that corresponds to region

thin-FASTA example:
```
chr1,1,15,ATGCCCCCCCCCAAA
chr1,200,205,GCCGA
chr2,10,12,AC
```

>Note: VarDict expands given regions by 1200bp to left and right (plus given value by `-x` option). 

## Program Workflow

#### The main workflow
The VarDictJava program follows the workflow:

1. Get regions of interest from a BED file or the command line.
2. For each segment:
   1. Find all variants for this segment in mapped reads:
      1. Optionally skip duplicated reads, low mapping-quality reads, and reads having a large number of mismatches.
      2. Skip unmapped reads.
      3. Skip a read if it does not overlap with the segment.
      4. Preprocess the CIGAR string for each read (section CIGAR Preprocessing).
      5. For each position, create a variant. If a variant is already present, adjust its count using the adjCnt function.
   2. Find structural variants (optionally can be disabled by option `-U`).
   3. Realign some of the variants using realignment of insertions, deletions, large insertions, and large deletions using unaligned parts of reads 
   (soft-clipped ends). This step is optional and can be disabled using the `-k 0` switch.
   4. Calculate statistics for the variant, filter out some bad ones, if any.
   5. Assign a type to each variant.
   6. Output variants in an intermediate internal format (tabular). Columns of the table are described in the Output Columns section.
	 
	 **Note**: To perform Steps 1 and 2, use Java VarDict.

3.	Perform a statistical test for strand bias using an R script.  
    **Note**: Use R scripts `teststrandbias.R` or `testsomatic.R` for this step.
4.	Transform the intermediate tabular format to VCF. Output the variants with filtering and statistical data.  
     **Note**: Use the Perl scripts `var2vcf_valid.pl` or `var2vcf_paired.pl` for this step.
     
#### CIGAR Preprocessing (Initial Realignment)
Read alignment is specified in a BAM file as a CIGAR string. VarDict modifies this string (and alignment) in the following special cases:
* Soft clipping next to insertion/deletion is replaced with longer soft-clipping. 
The same takes place if insertion/deletion is separated from soft clipping by no more than 10 matched bases.
* Short matched sequence and insertion/deletion at the beginning/end are replaced by soft-clipping.
* Two close deletions and insertions are combined into one deletion and one insertion
* Three close deletions are combined in one
* Three close insertions/deletions are combined in one deletion or in insertion+deletion
* Two close deletions are combined into one
* Two close insertions/deletions are combined into one
* Mis-clipping at the start/end are changed to matched sequences

#### Variants
Simple variants (SNV, simple insertions, and deletions) are constructed in the following way:
* Single-nucleotide variation (SNV). VarDict inserts an SNV into the variants structure for every matched or 
mismatched base in the reads. If an SNV is already present in variants, VarDict adjusts its counts and statistics.
* Simple insertion variant. If read alignment shows an insertion at the position, VarDict inserts +BASES 
string into the variants structure. If the variant is already present, VarDict adjusts its count and statistics.
* Simple Deletion variant. If read alignment shows a deletion at the position, VarDict inserts -NUMBER 
into the variants structure. If the variant is already present, VarDict adjusts its count and statistics.
VarDict also handles complex variants (for example, an insertion that is close to SNV or to deletion) 
using specialized ad-hoc methods.

Structural Variants are looked after simple variants. VarDict supported DUP, INV and DEL structural variants.

#### Variant Description String
The description string encodes a variant for VarDict internal use. 

The following table describes Variant description string encoding:

String | Description 
 ----- | ---------- 
[ATGC] | for SNPs 
+[ATGC]+ | for insertions 
-[0-9]+ | for deletions
...#[ATGC]+ | for insertion/deletion variants followed by a short matched sequence
...^[ATGC]+ | something followed by an insertion
...^[0-9]+ | something followed by a deletion
...&amp;[ATGC]+ | for insertion/deletion variants followed by a matched sequence

#### Variant Filtering
A variant appears in the output if it satisfies the following criteria (in this order):
1. Frequency of the variant exceeds the threshold set by the `-f` option (default = 1%).
2. The minimum number of high-quality reads supporting variant is larger than the threshold set by the `-r` option (default = 2).
3. The mean position of the variant in reads is less than the value set by the `-P` option (default = 5).
4. The mean base quality (phred score) for the variant is less than the threshold set by the `-q` option (default = 22.5).
5. Variant frequency is more than 25% or reference allele does not have much better mapping quality than the variant.
6. Deletion variants are not located in the regions where the reference genome is missing.
7. The ratio of high-quality reads to low-quality reads is larger than the threshold specified by `-o` option (default=1.5).
8. Variant frequency exceeds 30%.
9. Mean mapping quality exceeds the threshold set by the `-O` option (default: no filtering)
10. In the case of an MSI region, the variant size is less than 12 nucleotides for the non-monomer MSI or 15 for the monomer MSI. 
Variant frequency is more than 10% for the non-monomer MSI and 25% for the monomer MSI.
11. Variant has not "2;1" bias.
11. Variant is not SNV and variants refallele or varallele lengths are more then 3 nucleotides when variant frequency less then 20%.

## Program Options
- `-H|-?`  
    Print help page
- `-h|--header`   
    Print a header row describing columns
- `-i|--splice `
    Output splicing read counts
- `-p`   
    Do pileup regardless the frequency
- `-C`    
    Indicate the chromosome names are just numbers, such as 1, 2, not chr1, chr2 (deprecated)
- `-D|--debug`    
    Debug mode.  Will print some error messages and append full genotype at the end.
- `-y|--verbose`   
    Verbose mode.  Will output variant calling process.
- `-t|--dedup`   
    Indicate to remove duplicated reads.  Only one pair with identical start positions will be kept
- `-3`   
     Indicate to move indels to 3-prime if alternative alignment can be achieved.
- `-K`
     Include Ns in the total depth calculation.
- `-F bit`  
     The hexical to filter reads. Default: `0x500` (filter 2nd alignments and duplicates).  Use `-F 0` to turn it off.
- `-z 0/1`       
    Indicate whether the BED file contains zero-based coordinates, the same way as the Genome browser IGV does.  -z 1 indicates that coordinates in a BED file start from 0. -z 0 indicates that the coordinates start from 1. Default: `1` for a BED file or amplicon BED file.  Use `0` to turn it off. When using `-R` option, it is set to `0`
- `-a|--amplicon int:float`    
    Indicate it is amplicon based calling.  Reads that do not map to the amplicon will be skipped.  A read pair is considered to belong to the amplicon if the edges are less than int bp to the amplicon, and overlap fraction is at least float.  Default: `10:0.95`
- `-k 0/1`   
    Indicate whether to perform local realignment.  Default: `1` or yes.  Set to `0` to disable it.
- `-G Genome fasta`  
    The reference fasta.  Should be indexed (.fai).  Defaults to: `/ngs/reference_data/genomes/Hsapiens/hg19/seq/hg19.fa`
- `-R Region`  
    The region of interest.  In the format of chr:start-end.  If chr is not start-end but start (end is omitted), then it is a single position.  No BED is needed.
- `-d delimiter`  
    The delimiter for splitting `region_info`, defaults to tab `"\t"`
- `-n regular_expression`  
    The regular expression to extract sample names from bam filenames.  Defaults to: `/([^\/\._]+?)_[^\/]*.bam/`
- `-N string`   
    The sample name to be used directly.  Will overwrite `-n` option
- `-b string`   
    The indexed BAM file. Multiple BAM files can be specified with the “:” delimiter.
- `-c INT`   
    The column for chromosome
- `-S INT`   
    The column for the region start, e.g. gene start
- `-E INT`  
    The column for the region end, e.g. gene end
- `-s INT`   
    The column for a segment starts in the region, e.g. exon starts
- `-e INT`  
    The column for a segment ends in the region, e.g. exon ends
- `-g INT`     
    The column for a gene name, or segment annotation
- `-x INT`   
    The number of nucleotides to extend for each segment, default: `0`
- `-f double`   
    The threshold for allele frequency, default: `0.01` or `1%`
- `-r minimum reads`   
    The minimum # of variance reads, default: `2`
- `-B INT`  
    The minimum # of reads to determine strand bias, default: `2`
- `-Q INT`  
    If set, reads with mapping quality less than INT will be filtered and ignored
- `-q double`   
    The phred score for a base to be considered a good call.  Default: 22.5 (for Illumina). For PGM, set it to ~15, as PGM tends to underestimate base quality.
- `-m INT`   
    If set, reads with mismatches more than `INT` will be filtered and ignored.  Gaps are not counted as mismatches. Valid only for bowtie2/TopHat or BWA aln followed by sampe.  BWA mem is calculated as NM - Indels.  Default: 8, or reads with more than 8 mismatches will not be used.
- `-T|--trim INT`  
    Trim bases after `[INT]` bases in the reads
- `-X INT`   
    Extension of bp to look for mismatches after insersion or deletion.  Default to 3 bp, or only calls when they are within 3 bp.
- `-P number`  
    The read position filter.  If the mean variants position is less that specified, it is considered false positive.  Default: 5
- `-Z|--downsample double`  
    For downsampling fraction,  e.g. `0.7` means roughly `70%` downsampling.  Default: No downsampling.  Use with caution.  The downsampling will be random and non-reproducible.
- `-o Qratio`  
    The `Qratio` of `(good_quality_reads)/(bad_quality_reads+0.5)`.  The quality is defined by `-q` option.  Default: `1.5`
- `-O MapQ`  
    The reads should have at least mean `MapQ` to be considered a valid variant.  Default: no filtering
- `-V freq`  
    The lowest frequency in a normal sample allowed for a putative somatic mutations.  Defaults to `0.05`
- `-I INT`  
    The indel size.  Default: 50bp. 
    Be cautious with -I option, especially in the amplicon mode, as amplicon sequencing is not a way 
    to find large indels. Increasing the search size might slow and the false positives may appear in low 
    complexity regions. Increase it to 200-300 bp would recommend only for hybrid capture sequencing. 
- `-M INT`  
    The minimum matches for a read to be considered.  If, after soft-clipping, the matched bp is less than INT, then the 
    read is discarded.  It's meant for PCR based targeted sequencing where there's no insert and the matching is only the primers.
    Default: 0, or no filtering
- `-th [threads]`  
    If this parameter is missing, then the mode is one-thread. If you add the -th parameter, the number of threads 
    equals to the number of processor cores. The parameter -th threads sets the number of threads explicitly.
- `-VS STRICT | LENIENT | SILENT`   
    How strict to be when reading a SAM or BAM.
     `STRICT`   - throw an exception if something looks wrong.
     `LENIENT`  - Emit warnings but keep going if possible.
     `SILENT`   - Like `LENIENT`, only don't emit warning messages.
    Default: `LENIENT`
- `-u`  
    Indicate unique mode, which when mate pairs overlap, the overlapping part will be counted only once using **forward** read only.
    Default: unique mode disabled, all reads are counted.
- `-UN`  
    Indicate unique mode, which when mate pairs overlap, the overlapping part will be counted only once using **first** read only.
    Default: unique mode disabled, all reads are counted.
- `--chimeric`  
    Indicate to turn off chimeric reads filtering.  Chimeric reads are artifacts from library construction, 
    where a read can be split into two segments, each will be aligned within 1-2 read length distance,
    but in opposite direction. Default: filtering enabled
- `-U|--nosv`  
    Turn off structural variant calling
- `-L INT`   
   The minimum structural variant length to be presented using \<DEL\> \<DUP\> \<INV\> \<INS\>, etc. 
   Default: 1000. Any indel, complex variants less than this will be spelled out with exact nucleotides
- `-w|--insert-size INT` INSERT_SIZE  
   The insert size. Used for SV calling. Default: 300
- `-W|--insert-std INT` INSERT_STD  
   The insert size STD. Used for SV calling. Default: 100
- `-A INT` INSERT_STD_AMT  
   The number of STD. A pair will be considered for DEL if INSERT > INSERT_SIZE + INSERT_STD_AMT * INSERT_STD. Default: 4
- `-Y|--ref-extension INT`  
    Extension of bp of reference to build lookup table. Default to 1200 bp. Increase the number will slowdown the program. 
    The main purpose is to call large indels with 1000 bp that can be missed by discordant mate pairs. 
      
## Output columns

1. Sample - sample name
2. Gene - gene name from a BED file
3. Chr - chromosome name
4. Start - start position of the variation
5. End - end position of the variation
6. Ref - reference sequence
7. Alt - variant sequence
8. Depth - total coverage
9. AltDepth - variant coverage
10. RefFwdReads - reference forward strand coverage
11. RefRevReads - reference reverse strand coverage
12. AltFwdReads - variant forward strand coverage
13. AltRevReads - variant reverse strand coverage
14. Genotype - genotype description string
15. AF - allele frequency
16. Bias - strand bias flag
17. PMean - mean position in read
18. PStd - flag for read position standard deviation
19. QMean - mean base quality
20. QStd - flag for base quality standard deviation
21. MAPQ - mapping quality
22. QRATIO - ratio of high quality reads to low-quality reads
23. HIFREQ - variant frequency for high-quality reads
24. EXTRAFR - Adjusted AF for indels due to local realignment
25. SHIFT3 - No. of bases to be shifted to 3 prime for deletions due to alternative alignment
26. MSI - MicroSattelite. > 1 indicates MSI
27. MSINT - MicroSattelite unit length in bp
28. NM - average number of mismatches for reads containing the variant
29. HICNT - number of high-quality reads with the variant
30. HICOV - position coverage by high quality reads
31. 5pFlankSeq - neighboring reference sequence to 5' end 
32. 3pFlankSeq - neighboring reference sequence to 3' end
33. SEGMENT:CHR_START_END - position description
34. VARTYPE - variant type
35. DUPRATE - duplication rate in fraction
36. SV splits-pairs-clusters: Splits - No. of split reads supporting SV, Pairs - No. of pairs supporting SV, 
Clusters - No. of clusters supporting SV 

### Input Files

#### BED File – Regions
VarDict uses 2 types of BED files for specifying regions of interest: 4-column and 8-column. 
The 8-column file format is used for targeted DNA deep sequencing analysis (amplicon based calling), 
the 4-column file format - for single sample analysis.

All lines starting with #, browser, and track in a BED file are skipped. 
The column delimiter can be specified as the -d option (the default value is a tab “\t“).

The 8-column file format involves the following data:
* Chromosome name
* Region start position
* Region end position
* Gene name
* Score - not used by VarDict
* Strand - not used by VarDict
* Start position – VarDict starts outputting variants from this position
* End position –VarDict ends outputting variants from this position

The 4-column file format involves the following data:
* Chromosome name
* Region start position
* Region end position
* Gene name

#### FASTA File - Reference Genome
The reference genome in FASTA format is read using HTSJDK library. 
For every invocation of the toVars function (usually 1 for a region in a BED file) 
and for every BAM file, a part of the reference genome is extracted from the FASTA file. 

Region of FASTA extends and this extension can be regulate by REFEXT variable (option `-Y INT`, default 1200 bp).

# License

The code is freely available under the [MIT license](http://www.opensource.org/licenses/mit-license.html).

# Contributors

Java port of [VarDict](https://github.com/AstraZeneca-NGS/VarDict) implemented based on the original Perl version ([Zhongwu Lai](https://github.com/zhongwulai)) by:

- [Viktor Kirst](https://github.com/vkirst)

- [Zaal Lyanov](https://github.com/jabbarish)

