# Intro to RNA-Seq

Suggested read/based off: [hbctraining](https://github.com/hbctraining/Intro-to-rnaseq-hpc-salmon-flipped/blob/main/lessons/01_intro-to-RNAseq.md)  

# **How is RNA-Seq performed?** *i.e. how is my data created?*
RNA-Seq is used to quantify gene expression.
Genes coded in DNA are transcribed to RNA that might also get translated to proteins. Genes might be transcribed in different levels, or not expressed at all, in different cells.  

To be translated into proteins, the RNA must undergo processing to generate the mRNA. Genes in the DNA are comprised of the untranslated regions (UTRs) and the open read frame. Genes are transcribed into pre-mRNA, which still contains the intronic sequences. After post-transciptional processing, the introns are spliced out and a polyA tail and 5' cap are added to yield mature mRNA transcripts, which can be translated into proteins.  

## Illumina
### Library preparation
When starting an RNA-seq experiment, the RNA needs to be isolated and turned into a cDNA library for sequencing for every sample.
1. Briefly, the RNA is isolated from the sample and contaminating DNA is removed with DNase.
2. The RNA sample then undergoes either selection of the mRNA (polyA selection) or depletion of the rRNA. The resulting RNA is fragmented.  
Generally, ribosomal RNA represents the majority of the RNAs present in a cell, while messenger RNAs represent a small percentage of total RNA, ~2% in humans. Therefore, if we want to study the protein-coding genes, we need to enrich for mRNA or deplete the rRNA. For differential gene expression analysis, it is best to enrich for Poly(A)+, unless you are aiming to obtain information about long non-coding RNAs, then do a ribosomal RNA depletion.  
3. The RNA is then reverse transcribed into double-stranded cDNA and sequence adapters are then added to the ends of the fragments.  
    The cDNA libraries can be generated in a way to retain information about which strand of DNA the RNA was transcribed from. Libraries that retain this information are called stranded libraries, which are now standard with Illumina’s TruSeq stranded RNA-Seq kits. Stranded libraries should not be any more expensive than unstranded, so there is not really any reason not to acquire this additional information.  
    There are 3 types of cDNA libraries available:

    - Forward (secondstrand) – reads resemble the gene sequence or the secondstrand cDNA sequence
    - Reverse (firststrand) – reads resemble the complement of the gene sequence or firststrand cDNA sequence (TruSeq)
    - Unstranded
4. Finally, the fragments are PCR amplified if needed, and the fragments are size selected (usually ~300-500bp) to finish the library.  

**EXPAND ON THIS:** The size of the target fragments in the final library is a key parameter for library construction. DNA fragmentation is typically done by physical methods (i.e., acoustic shearing and sonication) or enzymatic methods (i.e., non-specific endonuclease cocktails and transposase tagmentation reactions.  


### Sequencing  

There are a few factors to considers: **single-/paired-end**, **sequencing platform** and **multiplexing**.  

#### Single-end, paired-end  
During sequencing, only the end of the fragment is read. Sequencing can be performed in only one end of the cDNA fragments (single-end reads), or in both (paired-end reads).
In SE you will have only read 1 (`r1`), and in PE you will have read1 and read2 (`r1` and `r2`), that can be 2 separate FastQ files or just one with interleaved pairs.

**USE CASE**  
Generally single-end sequencing is sufficient unless it is expected that the reads will match multiple locations on the genome (e.g. organisms with many paralogous genes), assemblies are being performed, or for splice isoform differentiation. Be aware that paired-end reads are generally 2x more expensive.


#### Sequencing platform  

The platform of choice (benchtop: MiniSeq, MiSeq, NextSeq; production-scale: HiSeq, HiSeq X, NovaSeq) will influence the sequencing results:
**EXPAND ON THIS**
- Quality of reads
- Read length
- Number of reads sequenced
The different platforms each use a different flow cell, which is a glass surface coated with an arrangement of paired oligos that are complementary to the adapters added to your template molecules. The flow cell is where the sequencing reactions take place.


#### Sequencing protocol  

The DNA fragments in the cDNA library are denatured and applied to the glass flow cell. These denatured fragments bind to the complementary oligos that are already covalently bound to the flow cell lanes, resulting in attachment. **Once the fragments have attached, a phase called cluster generation begins**.

During this step, single fragments are clonally amplified to create a cluster (fragments in close proximity) of identical fragments. This is necessary so that the **fluorescence can be readily captured from each cluster** (during nucleotide incorporation), instead of just a single fragment. The process of cluster generation is described in the four steps below, using a single fragment as an example.

Synthesize the complement with polymerase: The dsDNA is denatured, and the original DNA is washed away leaving the synthesized strand covalently bound to the flow cell. The single strand hybridises with an adjacent adapter to form a ‘bridge’. dsDNA is extended by polymerase. Now each strand is covalently bound to a different adapter on the flow cell.  
The process above is repeated many times to clonally amplify all unique fragments on flow cell to form clusters of identical sequence. This is happening for millions of fragments in parallel on the flow cell.  
After cluster generation, fluorescently-tagged nucleotides are incorporated one at a time (cyclically) and fluorescence images are captured to identify which nucleotide gets incorporated into each cluster in each cycle.

1. Denature clusters and then block 3’ ends to prevent unwanted priming. 
2. Hybridize sequencing primers to the adapter sequence at the loose ends.
3. Cycle four NTPs with fluorescent markers, terminator sequence and polymerases.
4. Once an NTP is incorporated, the cluster is excited by a light source and a characteristic fluorescent signal is emitted.
5. The color is recorded by the camera, then the terminator on dye is cleaved and washed. The process repeats for a specified number of cycles.


#### Base Calling
Illumina has a proprietary software that goes through all the images captured in the previous stage and generates text files with sequence information about each cluster based on the levels of fluorescence observed. In addition to calling the bases, this software assigns a probablity score to indicate how certain it was about the calling something an "A", a "T", a "G" or a "C".



If there are any ambiguities, e.g. at a certain cycle the image for a cluster does not have a distinct color that can be associated with a specific nucleotide, the base calling software will have a low probability associated with it and would assign an "N" instead of "A", "T", "G" or "C".

In closing,
- Number of clusters ~= Number of reads
- Number of sequencing cycles = Length of reads

The number of cycles (length of the reads) will depend on sequencing platform used as well as your preferences.

NOTE. If you want to explore sequencing by synthesis in more depth, we recommend this really nice animation available on Illumina's YouTube channel.

### Multiplexing
Depending on the Illumina platform (MiSeq, HiSeq, NextSeq), the number of lanes per flow cell, and the number of reads that can be obtained per lane varies widely. You will need to decide on how many reads you would like per sample (i.e. the sequencning depth) and then based on the platform you choose calculate how many total lanes you will require for your set of samples. We will talk more about considerations when making this decision in the next lesson on Experimental Considerations

Typically, charges for sequencing are per lane of the flow cell and you will be able to run multiple samples per lane. Illumina has therefore devised a nice multiplexing method which allows libraries from several samples to be pooled and sequenced simultaneously in the same lane of a flow cell. This method requires the addition of indices (within the Illumina adapter) or special barcodes (outside the Illumina adapter) as described in the schematic below.



NOTE: The workflow presented in this lesson is specific to Illumina sequencing, which is currently the most utilized sequencing method. But there are other long-read sequencing methods worth noting, such as:

Pacific Biosciences: http://www.pacb.com/
Oxford Nanopore (MinION): https://nanoporetech.com/
10X Genomics: https://www.10xgenomics.com/


# **Experimental planning considerations** *i.e. don't spend thousands in something that won't work*
Important factors:  

- Number and type of replicates
- Avoiding confounding
- Addressing batch effects

## Replicates vs depth
Experimental replicates can be performed as **technical replicates or biological replicates**.

- Technical replicates: same biological sample to repeat the technical or experimental steps: multiple measurements of the same sample. Helps reducing technical variation during analysis.
- Biological replicates use different biological samples of the same condition to measure the biological variation between samples. Same condition, multiple samples.

In the days of microarrays, technical replicates were considered a necessity; however, with the current RNA-Seq technologies, technical variation is much lower than biological variation and technical replicates are unneccessary.

In contrast, biological replicates are absolutely essential for differential expression analysis. 

For differential expression analysis, the more biological replicates, the better the estimates of biological variation and the more precise our estimates of the mean expression levels. This leads to more accurate modeling of our data and identification of more differentially expressed genes.


**Biological replicates are of greater importance than sequencing depth**, which is the total number of reads sequenced per sample. Higher in the number of replicates tends to return more DE genes than increasing the sequencing depth. Therefore, generally more replicates are better than higher sequencing depth, with the caveat that higher depth is required for detection of lowly expressed DE genes and for performing isoform-level differential expression.

**Sample pooling**: Try to avoid pooling of individuals/experiments, if possible; however, if absolutely necessary, then each pooled set of samples would count as a single replicate. To ensure similar amounts of variation between replicates, you would want to pool the same number of individuals for each pooled set of samples.
For example, if you need at least 3 individuals to get enough material for your control replicate and at least 5 individuals to get enough material for your treatment replicate, you would pool 5 individuals for the control and 5 individuals for the treatment conditions. You would also make sure that the individuals that are pooled in both conditions are similar in sex, age, etc.

### Guidelines

1. General gene-level **differential expression**:
- ENCODE guidelines suggest 30 million SE reads per sample (stranded).
- 15 million reads per sample is often sufficient, if there are a good number of replicates (>3).
- Spend money on more biological replicates, if possible.
- Generally recommended to have read length >= 50 bp

2. Gene-level differential expression with detection of **lowly-expressed genes**:
- Similarly benefits from replicates more than sequencing depth.
- Sequence deeper with at least 30-60 million reads depending on level of expression (start with 30 million with a good number of replicates).
- Generally recommended to have read length >= 50 bp  

3. **Isoform-level** differential expression:  
    1. Of **known isoforms**, suggested to have a depth of at least 30 million reads per sample and paired-end reads.  
    2. Of **novel isoforms** should have more depth (> 60 million reads per sample).  
Choose biological replicates over paired/deeper sequencing.
Generally recommended to have read length >= 50 bp, but longer is better as the reads will be more likely to cross exon junctions
Perform careful QC of RNA quality. Be careful to use high quality preparation methods and restrict analysis to high quality RIN # samples.  
  
4. Other types of RNA analyses (intron retention, small RNA-Seq, etc.):

Different recommendations depending on the analysis.

Almost always more biological replicates are better!

NOTE: The factor used to estimate the depth of sequencing for genomes is "coverage" - how many times do the number of nucleotides sequenced "cover" the genome. This metric is not exact for genomes (whole genome sequencing), but it is good enough and is used extensively. However, the metric does not work for transcriptomes because even though you may know what % of the genome has trancriptional activity, the expression of the genes is highly variable.

## Confounding
A confounded RNA-Seq experiment is one where you cannot distinguish the separate effects of two different sources of variation in the data.

For example, we know that sex has large effects on gene expression, and if all of our control mice were female and all of the treatment mice were male, then our treatment effect would be confounded by sex. We could not differentiate the effect of treatment from the effect of sex.

To AVOID confounding:
- Ensure animals in each condition are all the same sex, age, litter, and batch, if possible.

If not possible, then ensure to split the animals equally between conditions



## Batch effects
Batch effects are a significant issue for RNA-Seq analyses. This is illustrated in the figure below taken from Hicks SC, et al., bioRxiv (2015). On the right-hand side the experimental design is depicted, demonstrating a good use of batches by having samples from both sample groups present in each batch. On the far right hand side, an example PCA is plotted with the samples segregating by batch. The effect of batches on gene expression can often be larger than the effect from the experimental variable of interest, and it is therefore important to design your experiments such that we are able to account for this in our statistical model. We discuss this in more detail below.

How to know whether you have batches?
- Were all RNA isolations performed on the same day?
- Were all library preparations performed on the same day?
- Did the same person perform the RNA isolation/library preparation for all samples?
- Did you use the same reagents for all samples?
- Did you perform the RNA isolation/library preparation in the same location?

If any of the answers is ‘No’, then you have batches.

Best practices regarding batches:
- Design the experiment in a way to **avoid batches**, if possible.

If unable to avoid batches:
- **Do NOT confound your experiment by batch**.
- **DO split replicates of the different sample groups across batches**. The more replicates the better (definitely more than 2).
- DO include batch information in your experimental metadata. During the analysis, **we can regress out the variation due to batch if not confounded** so it doesn’t affect our results if we have that information.

NOTE: The sample preparation of cell line "biological" replicates "should be performed as independently as possible" (as batches), "meaning that cell culture media should be prepared freshly for each experiment, different frozen cell stocks and growth factor batches, etc. should be used." However, preparation across all conditions should be performed at the same time.