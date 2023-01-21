\--- ---

To cluster or not to cluster
============================

**Context** - You have sequenced the DNA of a microbial community using next generation sequencing (~ Illumina Miseq NGS) with your custom primer set. How do you figure out what species or types of organisms are in your dataset? You start by reducing the dataset to unique sequences/clusters by removing duplicates/errors before subsequently matching to 16S databases. The two prevailing paradigms to de-clutter the sequences are using OTUs and ASVs and there is a lot of debate out there on each of them and the contexts where they are most useful.

I compiled my thoughts from readings on this debate of

*   clustering 97% identity (= OTUs) vs
*   using raw denoised amplicon sequences (ASV)

To figure out scenarios where each is more relevant, with links for further reading.  
Look at my [google slide ppt](https://docs.google.com/presentation/d/17aro50YRmPq0sMHzjGNwyIXOuvBc0BHzv-PNpaTxmTY/edit#slide=id.p) for detailed explanation and introduction to the NGS sequencing

**Opinion** - _Personally, I'm sold with the ASV crowd without any reservations. I believe the OTU crowd is sticking to their ways because of inertia :P
But do read all the sources here and make up your own mind :)_
- Prashant Kalvapalle, Jan 2023

Synthesis
---------

ASVs are good when data quality is higher (lower sequencing noise), you want higher taxonomic resolution (species -> strain level?). Most importantly the analysis using ASVs should not hinge on diversity metrics of individual organisms in the sample, and you must be aware that you could be looking at different ASVs covering intra-genomic variation in your data (multiple distinct 16s copies in the same organism, ex: _E. coli_ has 7 copies)

If you choose to go with a higher taxonomic level of data and are willing to trade off with ease of re-usability/reproducibility of data without reanalysis and fresh OTU formation then do OTUs with the best method. Among methods to generate OTUs, swarm like clustering methods are better than either open or closed references as Pat Schloss (_Mothur’s creator_) points out - _closed reference ones are more stable, independent of data (though dependent of reference databases, which also update frequently)_

If you want more stable OTUs, and it is very important to compare across a huge number of independently analyzed samples, considering using ASVs instead if

*   you are dealing with environments that are rarely represented in databases (everything except gut bacteria?). Since closed ref OTUs will throw away these sequences not in the database. [Earth microbiome project](https://www.nature.com/articles/nature24621)

Chain of thoughts
-----------------

1.  [Exact sequence variants should replace operational taxonomic units in marker-gene data analysis, ISMEj 2017](https://www.nature.com/articles/ismej2017119). Podcast covering the major arguments in this paper [bioinformatics.chat](https://bioinformatics.chat/amplicon-sequence-variants)

Claims supremacy of ASVs over OTUs in **all scenarios** – these claims have been rebutted by Pat Schloss (creator of _Mothur_) and frierlab making the debate more nuanced.

### _Key advantages_

*   ASVs are consistent labels with biological meaning, independent of datasets or reference database. This enables comparison of these ASVs across studies, reproducibility in future datasets and meta-analysis across labs.
    *   The biological meaning still holds even if intra-genomic variation (multiple copies of 16s in same organism) splits an organism into multiple ASVs – given that data is interpreted not just by numbers through diversity, richness etc. but by presence/absence of particular ASVs across datasets.
    *   > For analysis to be reproducible the fundamental units must be reproducible, and _de novo_ OTUs are not. For analysis to be comprehensive the fundamental units must be comprehensive, and closed-reference OTUs are not. Replacing OTUs with ASVs makes marker-gene sequencing more precise, reusable, reproducible and comprehensive
        
    *   To me, this point by itself makes ASVs way superior for reproducibility sake.
*   ASVs have the **potential to have higher resolution** (single nucleotide), given high quality error free data. Given full length 16s reads (_Pacbio, nanopore_ etc) can even distinguish strains within species by differentiating intra-genomic variation of the multiple copies of the gene within the same organism.
    
    *   A criticism here
    
    > It is **not biologically plausible** to entertain the possibility that different _rrn_ operons from the same genome would have different ecologies. [Amplicon Sequence Variants Artificially Split Bacterial Genomes into Separate Clusters, ASM, 2021](https://journals.asm.org/doi/10.1128/mSphere.00191-21)
    
    *   One important point is that if the goal was higher,sub-species resolution in the first place, one should not be doing a 16s subregion analysis at all and should probably choose a different (functional) marker gene for the species of interest [cite?](s "find the tweet and cite")
*   ASVs don’t impose arbitrary dissimilarity thresholds, hence should be easier to apply to other marker genes?

### _Major criticism of ASVs_

*   While removing sequencing noise, very low abundance sequences (~ 1-5?) are discarded considering them likely to be sequencing artifacts/contamination etc. If you are interested in rare organisms, these methods will remove the singletons. This is also known to contribute to a much lower estimates of alpha diversity (richness, by 10x) compared to OTU methods
    *   DADA2 defends itself by giving an option to pool samples prior to error estimations. That way if there is a real rare variant that occurs in multiple samples at low abundances, the pooled sample will have enough of these sequences to be able to use them meaningfully and not discarded. An interesting hybrid pseudo pooling approach seems to be the most recommended for speed [Pseudo-pooling documentation](https://benjjneb.github.io/dada2/pseudo.html#pseudo-pooling)
        
    *   > their primary gain for noise is that they remove singletons. and there are huge problems with doing that. also note that if ASVs do what they claim, you could end up putting operons from the same chromosome into separate bins. [Pat Schloss tweet](https://twitter.com/PatSchloss/status/1111274111748227072?s=20)
        
        *   Is it really the primary gain? They are also ironing out sequencing noise when sequences are abundant _‘enough’_ – this I will argue is a way of clustering the noise out of sequences (_which originated from the same parent_) but in a statistically sound way using a NULL distribution rather than arbitrary similarity cutoffs. I think you are putting too much trust into the singletons…
    *   DADA2 proponents are also more prone to believing that such singletons are more likely to be false positives, and OTU based methods which take these into their _‘inflated’_ diversity estimates don’t have any sound basis for believing that these are real sequences. [Mike Lee tweet](https://twitter.com/AstrobioMike/status/1163742917166374912?s=20)
        
        *   Some believe that this is a **no man’s land** where one has to choose between retaining false positives vs incurring false negatives and the decision depends on the research question _(see the closing twitter comment)_
*   ASVs are sensitive to intra-genomic and intra-species variation (multiple copies of 16s in same organism) which could split an organism into multiple ASVs.
    *   So in this case each ASV does not correspond to each species but a higher resolution - which I believe works for everything except estimating the number of species (_which you should not be doing anyway due to untrustworthy singletons!_). The rest of the analysis is still valid as long as the researchers keep this in mind, because these sequences are **real**. The same argument is echoed in R.C. Edgar's [response] to Pat Schloss' criticism.
    *   > Splitting is a benign problem for most purposes because the biological conclusions of typical studies would be at least as good, and perhaps better, than lumping at 97%. For example, a random forest classifier would identity all three OTUs as predictive, or not, of the metadata; a Wilcoxon signed rank test would find, or not, that alpha diversity varies significantly between different healthy and diseased samples, and so on. However, if OTUs often lump distinct phenotypes, as will often be the case with the 97% OTUs advocated by Dr. Schloss, then strains with biological significance may be lumped with strains that are not significant and the signal could be diluted or lost. Thus, lumping can be harmful but splitting is generally benign.

### Caveats of OTUs

Only the ones relevant to the context set above  
Pat Scholss writes this -

> Furthermore, the 16S rRNA gene does not evolve at the same rate across all bacterial lineages (15), which limits the biological interpretation of a common OTU definition. A distance-based definition of a taxonomic unit based on 16S rRNA gene or full-genome sequences is operational and not necessarily grounded in biological theory [Amplicon Sequence Variants Artificially Split Bacterial Genomes into Separate Clusters, ASM, 2021](https://journals.asm.org/doi/10.1128/mSphere.00191-21)

2.  [http://fiererlab.org/2017/05/02/lumping-versus-splitting-is-it-time-for-microbial-ecologists-to-abandon-otus/](http://fiererlab.org/2017/05/02/lumping-versus-splitting-is-it-time-for-microbial-ecologists-to-abandon-otus/)

> Clearly ESV approach is not inherently better than traditional OTU based approach. If your data is **high quality**, you want **improved taxonomic resolution**, and you are _not concerned_ about the **intra-genomic heterogeneity** in the targeted marker genes, an **ESV-based approach could be advantageous**. Otherwise, a more standard OTU-based approach might be your best bet.

This does not address the reproducibility aspect of de-novo OTUs though…

3.  I found a OTU vs ASV debate in Mothur community with some nice links - [https://forum.mothur.org/t/otus-vs-asvs/3506](https://forum.mothur.org/t/otus-vs-asvs/3506). This highly linked reply is useful

> I would like to revive this topic, after reading the ISME publication by the DADA2 people ([https://www.nature.com/articles/ismej2017119 73](https://www.nature.com/articles/ismej2017119)). It is my opinion that this does merit some more conceptual attention on the mothur forum. Although the discussion has been started before on this forum (e.g. [OTUs or sequences? 17](https://forum.mothur.org/t/otus-or-sequences/2975/6) on ASVs and oligotyping, [OTU classification and minimum entropy decomposition 17](https://forum.mothur.org/t/otu-classification-and-minimum-entropy-decomposition/2867) on MED), I feel that there is no consensus on when to use the one or the other.  
> In the past my experience with ESV’s has been limited to oligotyping to a specific taxon (on relatively abundant OTU representing it) among different conditions and sometimes seeing different oligotypes popping up between conditions (i.e. the use as suggested by [@dwaite](https://forum.mothur.org/u/dwaite) in [OTU classification and minimum entropy decomposition 17](https://forum.mothur.org/t/otu-classification-and-minimum-entropy-decomposition/2867)). …  
> For instance: what is the prevailing opinion on **_ecological_ consistency/robustness** (e.g. [https://www.ncbi.nlm.nih.gov/pubmed/24763141 27](https://www.ncbi.nlm.nih.gov/pubmed/24763141) and [https://msphere.asm.org/content/2/2/e00073-17 29](https://msphere.asm.org/content/2/2/e00073-17)) and **_biological_ conistency/interpretation** of OTUs vs. ASVs (e.g. [https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5812548/ 42](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5812548/) and the discussion between Schloss and Edgar stated above)? For instance: How well does the assumption that “biological sequences are more likely to be repeatedly observed than are error-containing sequences” (Callahan _et al_. (2017) ISME) hold, e.g. in the case of the divergence in rRNA operons as stated in the comment by Fierer _et al_.?  
> I agree that it probably doesn’t make sense to use a marker gene study to distinguish pathogenic species/strains from others in the same genus based on ASV/ESV’s (given the resolution of most single-marker genes and the read length of MiSeq). But I do think that the argument that is being made about interoperability/re-usability of ASV’s is one that deserves a second look.  
> _As a side-note: with the advent of full-length 16S NGS amplicon seq (e.g. on the PacBio platform, [https://peerj.com/articles/1869/ 12](https://peerj.com/articles/1869/)), with decreasing error rates, would ASV’s become more relevant ?_

4.  Discussion of observed richness using DADA2 (with and without pooling) vs OTU and zOTUs in \[qiime2 forums\]. Has some nice twitter threads in the end ([https://forum.qiime2.org/t/method-comparison-differences-in-observed-richness/8788/2](https://forum.qiime2.org/t/method-comparison-differences-in-observed-richness/8788/2))

* * *

5.  [Updating the 97% identity threshold for 16S ribosomal RNA OTUs, OUP, 2018](https://academic.oup.com/bioinformatics/article/34/14/2371/4913809)  
    Claims a higher threshold is required to preserve the correspondence between species and OTUs. Rebuttal below
    
6.  Pat Schloss’ rebuttal and defense of the 97% OTU - [Oct 11, 2017](http://www.academichermit.com/2017/10/11/Review-of-Updating-the-97-identity-threshold.html); comments section in [bioarxiv](https://www.biorxiv.org/content/10.1101/192211v1#comment-3562391836)  
    Key arguments :
    
    *   higher than 97% => splitting intraspecies and [intragenomic](http://fiererlab.org/2017/10/09/intragenomic-heterogeneity-and-its-implications-for-esvs/) variations into different OTUs
    *   Some species (ex: within _Bacillus_ genus) are more finely split than others (_ex: Pseudomonas putida_) hence any constant clustering threshold is sub-optimal (is a pitch to the latest **swarm** kinds of algorithms?)
    *   Denoisers (DADA2, … ) are aggressive in removing rare sequences that _may_ be true
    
    > As several people have mentioned, DADA2 is happy to identify and report singletons. Just tell it to operate in pool=TRUE or pool=“pseudo” mode. [Callahan tweet](https://twitter.com/bejcal/status/1163970976935268357?s=20)
    

Pat Schloss mentions that Mothur was built for fully overlapping 2x250 bp Miseq reads, where errors are corrected by overlapping. Benjamin Callahan (DADA2 guy) [says](https://twitter.com/bejcal/status/1163971599122538496?s=20) that Deblur was meant for 1x ~150 read hence the denoising becomes more relevant

6.  Edgar’s [rebuttal](https://drive5.com/usearch/manual/schloss_97_rebuttal.html) to Pat Schloss
    
7.  Pat Schloss’ [tweets](https://threadreaderapp.com/thread/1110509388664619009.html) promoting de-novo clustering
    

> Use de-novo clustering (ex: [Swarm](https://github.com/torognes/swarm#refine_OTUs)) instead of open (closed) clustering of OTUs - which do closed ref first and open later and have different OTU meanings within each of them. [Peerj, 2015](https://peerj.com/articles/1487/)

Not sure if he means a fixed threshold 97% de novo clustering or dynamic methods like swarm

> Open/closed reference methods were developed because people have the type of crappy data that you get when reads don’t fully overlap. High error rates increase the number of uniques making downstream processing more difficult.

Is this a case for de-noising when overlapping regions are not significant enough to correct for errors?

8.  The Earth microbiome sequencing project endorses ASVs as opposed to OTUs. [A communal catalogue reveals Earth’s multiscale microbial diversity, Nature, 2017](https://www.nature.com/articles/nature24621)

> Sequence analysis and taxonomic profiling were done initially using the common approach of assigning sequences to operational taxonomic units (OTUs) clustered by sequence similarity to existing rRNA databases. While this approach was useful for certain analyses, for many sample types, _especially plant-associated and free-living communities_, **one-third of reads or more could not be mapped to existing rRNA databases**. We therefore used a reference-free method, Deblur [21](s "Amir, A. et al. Deblur rapidly resolves single-nucleotide community sequence patterns. mSystems 2, e00191–16 (2017)"), to remove suspected error sequences and provide single-nucleotide resolution ‘sub-OTUs’, also known as ‘amplicon sequence variants’

> particularly the use of exact sequences instead of clustered operational taxonomic units, enable bacterial and archaeal ribosomal RNA gene sequences to be followed across multiple studies and allow us to explore patterns of diversity at an unprecedented scale

> Because exact sequences are stable identifiers, unlike OTUs, they can be compared to any 16S rRNA or genomic database now and in the future, thereby promoting reusability

9.  Nice [twitter](https://twitter.com/AstrobioMike/status/1163742917166374912) debate about the best way to remove spurious ‘OTUs’

There is an assertion that singletons are most likely spurious since there was some small mock community (20 members) which gave 100s of OTUs. [RC Edgar, peerj, 2014](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5631090/)

Two approaches:

*   Pat Schloss’ rarefying to a common number of sequences from a mock community vs
*   Dada2’s use of low abundance as a key to discard spurious ‘ASVs’ caused by sequencing errors which cannot be corrected since low abundance does not provide enough information to fit to their distribution

People questioning rarefaction: [Waste Not, Want Not: Why Rarefying Microbiome Data Is Inadmissible, Plosone, 2014](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003531)

10.  [Broadscale Ecological Patterns Are Robust to Use of Exact Sequence Variants versus Operational Taxonomic Units](https://journals.asm.org/doi/full/10.1128/mSphere.00148-18)

> Despite quantitative differences in microbial richness, we found that all α and β diversity metrics were highly positively correlated (_r_ > 0.90) between samples analyzed with both approaches. Moreover, the community composition of the dominant taxa did not vary between approaches. Consequently, statistical inferences were nearly indistinguishable.

> OTU classifications remain biologically useful for comparing diversity across large data sets ([7](https://journals.asm.org/doi/full/10.1128/mSphere.00148-18#B7)) or identifying clades that share traits ([16](https://journals.asm.org/doi/full/10.1128/mSphere.00148-18#B16)).

> Thus, while there are good reasons to employ ESVs, we need not question the validity of results based on OTUs.

I guess the main criticism is not the validity of OTU based results within a experiment, it is the loss of reproducibility and of comparibility to other datasets by the community when OTUs are used as opposed to ASVs. So the knowledge produced by this analysis effort is limited in some way

12.  Closing comment! [twitter, March 1, 2021](https://twitter.com/_Faecal_Matters/status/1366512645835284483?s=20)

> I see we’re still having this ASV vs OTU debate then… People, come together, both approaches are wrong. It just depends in which direction you prefer your wrongness to be in. Can we please move on now, and let people use whichever they want.

* * *

> Written with [StackEdit](https://stackedit.io/).
