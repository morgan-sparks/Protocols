# Christie Lab Guide to SNP filtering

By: Morgan Sparks and Peter Euclide, March 2022

The goal of this document is to provide a basic overview for SNP filtering and an outline for general best practices for SNP filtering.

The Genome Analysis Toolkit, is the general tool--though not exclsusive--most of us are currently using for alignment and variant calling. GATK and other genotype callers often result in a VCF (Variant Call Fortmat) file that contains all of the variants for every individual along with a header describing metadata within the file.

A helpful discussion of the VCF [format]{https://en.wikipedia.org/wiki/Variant_Call_Format} is available on wikipedia.

### Why filter?
Before working with your data it is important to filter it to make sure that there are not any serious issues that could influence your results. these include:

1. sequencing errors - errors arising from sequencing. Illumina's current error rate is about 0.1% or ~1 in 1000.
2. missing data - some missing data is ok. But markers or individuals that are missing a lot of genotype calls can be a problem. Frequent cut-offs include dropping samples and markers with < 70 to 90 % Genotype rate. This value will differ slightly among datasets. However, a good starting place for the Christie lab is 80%
3. low minor allele frequency - many SNP variants will be contain low MAF. While these can represent true SNPs that are rare in a population, they can also often be sequencing or genotyping errors. This is because the probability of the same error occurring frequently is low. So most sequencing errors occur only once or twice in a dataset. Removing variants with low MAF will therefore control for these issues while not significantly reducing the power of the dataset. Be aware, this filter can greatly reduce the number of SNPs, sometimes by 80 to 90%. Additionally, while you may be losing many SNPs keep in mind that for our purposes we're often looking for loci of large effect and these would not show up in Fst or related tests.

### What types of filters?
There are different types of filtering that you might consider based on what you know about your data.

#### 1. Hard filters
Hard filters are filters you apply to every sample in you dataset. A good discussion of what GATK recommends for germline short variants is available [here]{https://gatk.broadinstitute.org/hc/en-us/articles/360035890471-Hard-filtering-germline-short-variants}.

These hard filters tend to have to do with many of the factors discussed above like the error rate of the platform you are using or missing data.

#### 2. Soft filters
Soft filters are often applied to all your data but you may use multiple thresholds or try filtering population in multiple ways.

Soft filters will often have to do with what you know about the population genetics and ecology/evolutionary history of the organism you are working with.

### Christie Lab Filtering Best Practices

For hard filtering, we've provided examples above but it's best to consult whatever pipeline you are using.

GATK provides software to apply the first hard filter on your dataset but it's common to use another software such as VCF tools to do the following:

1. Missing data per locus
2. Missing data per individual 
3. Minor allele frequency 
    * Do for all pops at once
    * Do separately by pop
4. Hardy-weinberg equilibrium
    * Particularly heterozygous excess individuals



