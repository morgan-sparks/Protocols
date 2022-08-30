# Christie Lab Guide to SNP filtering

By: Peter Euclide and Morgan Sparks, March 2022

The goal of this document is to provide a basic overview for SNP filtering and an outline for general best practices for SNP filtering.

The Genome Analysis Toolkit, is the general tool--though not exclsusive--most of us are currently using for alignment and variant calling. GATK and other genotype callers often result in a VCF (Variant Call Fortmat) file that contains all of the variants for every individual along with a header describing metadata within the file.

A helpful discussion of the VCF [format]{https://en.wikipedia.org/wiki/Variant_Call_Format} is available on wikipedia.

---

### Why filter?
Before working with your data it is important to filter it to make sure that there are not any serious issues that could influence your results. these include:

1. sequencing errors - errors arising from sequencing. Illumina's current error rate is about 0.1% or ~1 in 1000.
2. missing data - some missing data is ok. But markers or individuals that are missing a lot of genotype calls can be a problem. Frequent cut-offs include dropping samples and markers with < 70 to 90 % Genotype rate. This value will differ slightly among datasets. However, a good starting place for the Christie lab is 80%
3. low minor allele frequency - many SNP variants will be contain low MAF. While these can represent true SNPs that are rare in a population, they can also often be sequencing or genotyping errors. This is because the probability of the same error occurring frequently is low. So most sequencing errors occur only once or twice in a dataset. Removing variants with low MAF will therefore control for these issues while not significantly reducing the power of the dataset. Be aware, this filter can greatly reduce the number of SNPs, sometimes by 80 to 90%. Additionally, while you may be losing many SNPs keep in mind that for our purposes we're often looking for loci of large effect and these would not show up in Fst or related tests.

---

### What types of filters?
There are different types of filtering that you might consider based on what you know about your data.

#### 1. Hard filters
Hard filters are filters you apply to every sample in you dataset. A good discussion of what GATK recommends for germline short variants is available [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035890471-Hard-filtering-germline-short-variants).

These hard filters tend to have to do with many of the factors discussed above like the error rate of the platform you are using or missing data.

#### 2. Soft filters
Soft filters are often applied to all your data but you may use multiple thresholds or try filtering population in multiple ways.

Soft filters will often have to do with what you know about the population genetics and ecology/evolutionary history of the organism you are working with.

---

### Christie Lab Filtering Best Practices

For hard filtering, we've provided examples above but it's best to consult whatever pipeline you are using.

GATK provides software to apply the first hard filter on your dataset based on sequence and alignment quality but it's common to use another software such as VCF tools to conducting additional filtering based on genotype data:

1. Missing data per locus
2. Missing data per individual 
3. Minor allele frequency 
    * Do for all pops at once
    * Do separately by pop
4. Hardy-weinberg equilibrium
    * Particularly heterozygous excess individuals

---

### Example pipeline for a 'good' genetic dataset

This pipeline can be used as a default settings pipeline that should provide a moderate to good quality dataset. However, filters need to be re-assessed and evaluated for each new project. Therefore this represents only a starting point, and should not be used as the final set of filters for your data.

This pipeline begins just after a VCF file containing all of the variants and individuals has been generated using GATK or similar genotyping protocol. Both gzipped (vcf.gz) and unzipped (.vcf) files are acceptable for VCFtools. For the purpose of this pipeline our starting file will be named `FILE.vcf`.

1. GATK does not remove hard filtered variants. It just flags them. You can remove all variants that have been flagged in VCFtools.

`vcftools --vcf FILE.vcf --remove-filtered-all --recode --recode-INFO-all --out FILE.filt`

2. Remove all non-biallelic SNPs

`vcftools --vcf FILE.filt.recode.vcf --min-alleles 2 --max-alleles 2  --recode --recode-INFO-all --out FILE.filt.biallelic`

3. Check individual missingness (i.e., the number of missing genotypes present in each individual)

`vcftools --vcf FILE.filt.biallelic.recode.vcf --missing-indv --out FILE.filt.biallelic`
 
 **results are written to a .iMiss file containing counts of % missing data by individual**

4. Check locus missingness (i.e., the number of missing genotypes present for each locus/SNP/varient)

`vcftools --vcf FILE.filt.biallelic.recode.vcf --missing-site --out FILE.filt.biallelic`
 
 **results are written to a .lMiss file containing counts of % missing data by locus**

5. Remove individuals missing >20% of genotypes - do this by identifying all individuals with a proportion missing > 20% from the iMiss file and writing names to `indiv80.txt`.

`vcftools --vcf FILE.filt.biallelic.recode.vcf --keep indiv80.txt --recode --recode-INFO-all --out FILE.filt.biallelic.Ind80`

6. Remove loci missing >20% of genotypes (Use --max-missing-count. Do this by calculating the number of _alleles_ missing (N) that result in a 20% missingness. N = 0.2*(2*number of samples)

`vcftools --vcf FILE.filt.biallelic.Ind80.recode.vcf --max-missing-count N --recode --recode-INFO-all --out FILE.filt.biallelic.Ind80.MMCN`

7. Remove loci with a minor allele frequency (MAF) < 0.05.

`vcftools --vcf FILE.filt.biallelic.Ind80.MMCN.recode.vcf --maf .05 --recode --recode-INFO-all --out FILE.filt.biallelic.Ind80.MMCN.MAF5`

8. Re-check individual and locus missingness.

`vcftools --vcf FILE.filt.biallelic.Ind80.MMCN.MAF5.recode.vcf --missing-indv --out FILE.filt.biallelic.Ind80.MMCN.MAF5` 
`vcftools --vcf FILE.filt.biallelic.Ind80.MMCN.MAF5.recode.vcf --missing-site --out FILE.filt.biallelic.Ind80.MMCN.MAF5`

9. Calculate 'soft filter' metrics.

`vcftools --vcf FILE.filt.biallelic.Ind80.MMCN.MAF5.recode.vcf --hardy --out FILE.filt.biallelic.Ind80.MMCN.MAF5`
`vcftools --vcf FILE.filt.biallelic.Ind80.MMCN.MAF5.recode.vcf --freq --out FILE.filt.biallelic.Ind80.MMCN.MAF5`
`vcftools --vcf FILE.filt.biallelic.Ind80.MMCN.MAF5.recode.vcf --site-mean-depth --out FILE.filt.biallelic.Ind80.MMCN.MAF5`

10. Values from step 3, 4, 8, and 9 should be pulled into R or Excel and summarized by plotting the densities and distributions of resulting values. The goal is to make sure that 1) The filters worked and there is not high-missingness. and 2) there are no weird patterns that could indicate genotype errors.



