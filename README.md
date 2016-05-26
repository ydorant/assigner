# assigner
[![Travis-CI Build Status](https://travis-ci.org/thierrygosselin/assigner.svg?branch=master)](https://travis-ci.org/thierrygosselin/assigner) [![CRAN_Status_Badge](http://www.r-pkg.org/badges/version/assigner)](http://cran.r-project.org/package=assigner) [![DOI](https://zenodo.org/badge/14548/thierrygosselin/assigner.svg)](https://zenodo.org/badge/latestdoi/14548/thierrygosselin/assigner)

Next-generation sequencing techniques that reduce the size of the genome (e.g. genotype-by-sequencing (GBS) and
restriction-site-associated DNA sequencing (RADseq)) produce huge numbers of markers that hold great potential and promises for assignment analysis. After hitting the bioinformatic wall with the different workflows you'll likely end up with several folders containing whitelist and blacklist of markers and individuals, data sets with various *de novo* and/or filtering parameters and missing data. This reality of GBS/RAD data is quite hard on GUI software traditionally used for assignment analysis. The end results is usually poor data exploration, constrained by time, and poor reproducibility.

**assigner** was tailored to make it easy to conduct assignment analysis using GBS/RAD data within R. Additionally, combining the use of tools like [RStudio] (https://www.rstudio.com) and [GitHub] (https://github.com) will make effortless documenting your workflows and pipelines.

This is the development page of the **assigner** package for the R software. The name **assigner** |əˈsʌɪn| is rooted in the latin word *assignare*. It's first use in french dates back to XIIIe.

**Use assigner to:**

* **Conduct assignment analysis** and **mixture analysis** using [gsi_sim] (https://github.com/eriqande/gsi_sim), a tool developed 
by Eric C. Anderson (see Anderson et al. 2008 and Anderson 2010) or [adegenet] (https://github.com/thibautjombart/adegenet), a R package developed by Thibaul Jombart, to conduct the assignment analysis.
* The input file are:
    1. a VCF file format (Danecek et al. 2011) (*batch_x.vcf*) produced by [STACKS] (http://catchenlab.life.illinois.edu/stacks/) (Catchen et al. 2011, 2013),
    2. an haplotypes data frame file (*batch_x.haplotypes.tsv*) produced by [STACKS] (http://catchenlab.life.illinois.edu/stacks/) (Catchen et al. 2011, 2013),
    3. very large files (> 50 000 markers) can be imported in PLINK tped/tfam format (Purcell et al. 2007), and
    4. a data frame of genotypes.
    5. For genepop file, use [genepopedit](https://github.com/rystanley/genepopedit) to prepare the file for assigner.
* Individuals, populations and markers can be **filtered** and/or selected in several ways using **blacklist, 
whitelist** and other arguments
* **Map-independent imputation** of missing genotype or alleles using **Random Forest** or the most frequent category is also available to test the impact of missing data on assignment analysis
* Genotypes of poor quality (e.g. in coverage, genotype likelihood or sequencing errors) can be erased prior to imputations or assignment analysis with the use of a `blacklist.genotype` argument.
* Markers can be randomly selected for a **classic LOO (Leave-One-Out) assignment** or 
chosen based on **ranked Fst** (Weir & Cockerham, 1984) for a **THL (Training, Holdout, Leave-one-out) assignment analysis** (reviewed in Anderson 2010)
* Use `iteration.method` and/or `iteration.subsample` arguments to resample markers or individuals to get statistics!
* The impact of the minor allele frequency, MAF, (local and global) can also be easily explored with custom thresholds
* The impact of filters used in other software can be explored by using the `whitelist.markers` argument.
* Compute the **genotype likelihood ratio distance metric (Dlr)** (Paetkau's et al. 1997, 2004)
* Import and summarise the assignment results from [GenoDive] (http://www.bentleydrummer.nl/software/software/GenoDive.html) (Meirmans and Van Tienderen, 2004)
* `ggplot2`-based plotting to view assignment results and create publication-ready figures
* Fast computations with optimized codes to run in parallel!


## Installation
You can try out the dev version of **assigner**. Follow the 3 steps below:

Step 1 You will need the package **devtools**
```r
if (!require("devtools")) install.packages("devtools") # to install
library(devtools) # to load
```

Step 2 Install **assigner** and install [gsi_sim] (https://github.com/eriqande/gsi_sim) from source:
```r
install_github("thierrygosselin/assigner") # to install
library(assigner) # to load
install_gsi_sim(fromSource = TRUE) # to install gsi_sim from source
```

Step 3 For faster imputations, you need to install an OpenMP enabled **randomForestSRC package** [website](http://www.ccs.miami.edu/~hishwaran/rfsrc.html).

Option 1: From source (Linux & Mac OSX)

```r
# Terminal
cd ~/Downloads
curl -O https://cran.r-project.org/src/contrib/randomForestSRC_2.0.7.tar.gz
tar -zxvf randomForestSRC_2.0.7.tar.gz
cd randomForestSRC
autoconf
# Back in R:
install.packages(pkgs = "~/Downloads/randomForestSRC", repos = NULL, type = "source")
```
Option 2: Use a pre-compiled binary (Mac OSX) [instructions here] (http://www.ccs.miami.edu/~hishwaran/rfsrc.html) or quick copy/paste solution below:

```r
# Mac OSX
library("devtools")
install_url(url = "http://www.ccs.miami.edu/~hishwaran/rfsrc/randomForestSRC_2.0.7.tgz")
```

**Note for Windows users**:

**assigner** is currently not tested with PC, sorry. 
I currently don't have the time to do PC bioinformatics troubleshooting.

**Problems during installation:**

Sometimes you'll get warnings while installing dependencies required for **assigner** or other R packages.
```r
Warning: cannot remove prior installation of package ‘stringi’
```

To solve this problem: 

Option 1. Delete the problematic packages manually and reinstall. On MAC computers, in the **Finder**, use the shortcut **cmd+shift+g**, or in the menu bar : **GO -> Go to Folder**, copy/paste the text below:
```r
/Library/Frameworks/R.framework/Resources/library
#Delete the problematic packages.
```

Option 2. If you know your way around the terminal and understand the consequences of using **sudo rm -R** command, here something faster to remove problematic packages:
```r
sudo rm -R /Library/Frameworks/R.framework/Resources/library/package_name
# Changing 'package_name' to the problematic package.
# Reinstall the package.
```

**Dependencies**

Here the list of packages that **assigner** is depending on:
```r
if (!require("reshape2")) install.packages("reshape2")
if (!require("ggplot2")) install.packages("ggplot2")
if (!require("stringr")) install.packages("stringr")
if (!require("stringi")) install.packages("stringi")
if (!require("plyr")) install.packages("plyr")
if (!require("dplyr")) install.packages("dplyr")
if (!require("tidyr")) install.packages("tidyr")
if (!require("readr")) install.packages("readr")
if (!require("purrr")) install.packages("purrr")
if (!require("data.table")) install.packages("data.table")
if (!require("lazyeval")) install.packages("lazyeval")
if (!require("adegenet")) install.packages("adegenet")
if (!require("parallel")) install.packages("parallel")
```
If you don't have them, no worries, it's intalled automatically during **assigner** installation. If you have them, it's your job to update them, because i'm usually using the latest versions...

## Parallel computation and Cloud Computing

Most of the function in **assigner** were designed to be as fast as possible. 
Using computer with 16GB RAM is recommended. 
With more CPU and Memory comes faster computation time. 
If you decide to keep intermediate files during assignment analysis, 
you will need a large external drive (disk space is cheap). 
Solid State Drive and thunderbolt cables will provide fast input/output. 

If disk space and computer power is an issue, cloud computing with [Google Cloud Compute Engine] (https://cloud.google.com/compute/) and 
[Amazon Elastic Cloud Compute] (https://aws.amazon.com/ec2/) is cheap and can be used easily. 

A tutorial and pipeline along an Amazon Machine Image (AMI) are available 
in our [tutorial-workflow] (http://gbs-cloud-tutorial.readthedocs.org/en/latest/). 

The AMI is preloaded with **gsi_sim** and the required **R packages**. 
Following a few steps: [link] (http://gbs-cloud-tutorial.readthedocs.org/en/latest/10_use_rstudio.html), 
you can have [RStudio server] (https://www.rstudio.com/) running and used through your web browser!

The Amazon image can be imported into Google Cloud Compute Engine to start a new compute engine virtual machine: [link] (https://cloud.google.com/compute/docs/creating-custom-image#import_an_ami_image). 


## New features
Version, new feature and bug history now lives in the [NEWS.md file] (https://github.com/thierrygosselin/assigner/blob/master/NEWS.md)

**v.0.2.7**
* you need to update [stackr] (https://github.com/thierrygosselin/stackr) to v.0.2.7
to appreciate this new version of assigner.

* updated `assignment_ngs` with the separate stackr modules to simplify the function.

* new data file available for `assignment_ngs`: `genepop` and `genind` object.

* `assignment_ngs` now accept any vcf input file! i.e. it’s no longer limited to stacks vcf. 

* new arguments in `assignment_ngs`. The assignment using dapc can now use 
the optimized alpha score `adegenet.dapc.opt == "optim.a.score"` or 
the cross-validation `adegenet.dapc.opt == "xval"`. This is useful for fine tuning 
the trade-off between power of discrimination and over-fitting 
(for stability of group membership probabilities).
Cross validation with `adegenet.dapc.opt == "xval"` doesn't work with 
missing data, so it's only available with *imputed data* 
(i.e. `imputation.method == "rf" or "max"`). 
With non imputed data or the default: the optimized alpha-score is used 
(`adegenet.dapc.opt == "optim.a.score"`).
When using `adegenet.dapc.opt == "xval"`, 2 new arguments are available: 
(1) `adegenet.n.rep` and (2) `adegenet.training`. See documentation for details.

* removed arguments in `assignment_ngs`. Removed the `pop.id.start` and `pop.id.end`
arguments that were confusing people. For those used to these arguments, 
they are now recycled in the new function `individuals2strata`
in [stackr] (https://github.com/thierrygosselin/stackr). 
The strata file created by this function can be used with the `strata` argument in
`assignment_ngs`.

* 2 modified arguments in `assignment_ngs`: (1) `gsi_sim.filename` is now `filename`; and 
(2) if you didn't use the imputation argument, replace `imputation.method = FALSE`
to `imputation.method = NULL` or leave the argument missing.

* simplified sections of codes in `assignment_ngs` that dealt with `strata`, 
`pop.levels` and `pop.labels`.

* new function: `write_gsi_sim`. Write a gsi_sim file from a data frame (wide or long/tidy). 
Used internally in [assigner] (https://github.com/thierrygosselin/assigner)
and might be of interest for users.

**v.0.2.6**
* `fst_WC84` is now a separate and very fast function that can compute: the overall and pairwise Weir and Cockerham 1984 Theta/Fst. Bootstrap resampling of markers is avalaible to build Confidence Intervals (For Louis Bernatchez and his students;). The estimates are available as a data frame and a matrix with upper diagonal filled with Fst values and lower diagonal filled with the confidence intervals.

For previous news:
[NEWS.md file] (https://github.com/thierrygosselin/assigner/blob/master/NEWS.md)


## Roadmap of future developments:

* The ability to provide the ranking of markers based on something else than Fst (Weir and Cockerham, 1984) currently used in the function. e.g. Informativeness for Assignment Measure (In, Rosenberg et al. 2003), the Absolute Allele Frequency Differences (delta, δ, Shriver et al., 1997).
* Provide ranking from other software: e.g. Toolbox for Ranking and Evaluation of SNPs [TRES](http://mlkd.csd.auth.gr/bio/tres/), [BayeScan](http://cmpg.unibe.ch/software/BayeScan/) and [OutFLANK](https://github.com/whitlock/OutFLANK).
* Would be very cool to use genotype likelihood information to get more accurate assignment.
* Documentation and vignettes
* Workflow tutorial
* Use Shiny and ggvis when subplots or facets are available
* CRAN
* ...suggestions ?

## Contributions:

This package has been developed in the open, and it wouldn’t be nearly as good without your contributions. There are a number of ways you can help me make this package even better:  
* If you don’t understand something, please let me know. 
* Your feedback on what is confusing or hard to understand is valuable. 
* If you spot a typo, feel free to edit the underlying page and send a pull request.

New to pull request on github ? The process is very easy:  
* Click the edit this page on the sidebar.
* Make the changes using github’s in-page editor and save.
* Submit a pull request and include a brief description of your changes. 
* “Fixing typos” is perfectly adequate.


## Example 
Under construction

Here an example on how to read the GenoDive assignment output file in order to 
use the arguments in the `dlr` and the `plot_assignment_dlr` functions correctly.

![](vignettes/dlr.example.png)


Vignettes are in development, check periodically for updates.


## References

Anderson EC, Waples RS, Kalinowski ST (2008) An improved method for predicting the accuracy of genetic stock identification. Canadian Journal of Fisheries and Aquatic Sciences, 65, 1475–1486.

Anderson EC (2010) Assessing the power of informative subsets of loci for population assignment: standard methods are upwardly biased. Molecular Ecology Resources, 10, 701–710.

Catchen JM, Amores A, Hohenlohe PA et al. (2011) Stacks: Building and Genotyping Loci De Novo From Short-Read Sequences. G3, 1, 171–182.

Catchen JM, Hohenlohe PA, Bassham S, Amores A, Cresko WA (2013) Stacks: an analysis tool set for population genomics. Molecular Ecology, 22, 3124–3140.

Danecek P, Auton A, Abecasis G et al. (2011) The variant call format and VCFtools. Bioinformatics, 27, 2156–2158.

Foll M, Gaggiotti O (2008) A Genome-Scan Method to Identify Selected Loci Appropriate for Both Dominant and Codominant Markers: A Bayesian Perspective. Genetics, 180, 977–993.

Ishwaran H. and Kogalur U.B. (2015). Random Forests for Survival, Regression and Classification (RF-SRC), R package version 1.6.1.

Jombart T (2008) adegenet: a R package for the multivariate analysis of genetic markers. Bioinformatics, 24, 1403–1405.

Jombart T, Ahmed I (2011) adegenet 1.3-1: new tools for the analysis of genome-wide SNP data. Bioinformatics, 27, 3070–3071.

Kavakiotis I, Triantafyllidis A, Ntelidou D et al. (2015) TRES: Identification of Discriminatory and Informative SNPs from Population Genomic Data. Journal of Heredity, 106, 672–676.

Meirmans PG, Van Tienderen PH (2004) genotype and genodive: two programs for the analysis of genetic diversity of asexual organisms. Molecular Ecology Notes, 4, 792-794.

Paetkau D, Slade R, Burden M, Estoup A (2004) Genetic assignment methods for the direct, real-time estimation of migration rate: a simulation-based exploration of accuracy and power. Molecular Ecology, 13, 55-65.

Paetkau D, Waits LP, Clarkson PL, Craighead L, Strobeck C (1997) An empirical evaluation of genetic distance statistics using microsatellite data from bear (Ursidae) populations. Genetics, 147, 1943-1957.

Purcell S, Neale B, Todd-Brown K, Thomas L, Ferreira MAR, Bender D, et al. 
PLINK: a tool set for whole-genome association and population-based linkage analyses. 
American Journal of Human Genetics. 2007; 81: 559–575. doi:10.1086/519795

Rosenberg NA, Li LM, Ward R, Pritchard JK (2003) Informativeness of genetic markers for inference of ancestry. American Journal of Human Genetics, 73, 1402–1422.

Shriver MD, Smith MW, Jin L et al. (1997) Ethnic-affiliation estimation by use of population-specific DNA markers. American Journal of Human Genetics, 60, 957.

Weir BS, Cockerham CC (1984) Estimating F-Statistics for the Analysis of Population Structure. Evolution, 38, 1358–1370.

Whitlock MC, Lotterhos KE (2015) Reliable Detection of Loci Responsible for Local Adaptation: Inference of a Null Model through Trimming the Distribution of FST*. The American Naturalist, S000–S000.

## Citation:
To get the citation for **assigner**, inside R:
```r
citation("assigner")
```

