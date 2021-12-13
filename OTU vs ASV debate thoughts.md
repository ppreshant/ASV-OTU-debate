---


---

<h1 id="to-cluster-or-not-to-cluster">To cluster or not to cluster</h1>
<p><strong>Context</strong> - You have sequenced the DNA of a microbial community using next generation sequencing (~ Illumina Miseq NGS) with your custom primer set. How do you figure out what species or types of organisms are in your dataset? The two prevailing paradigms are using OTUs and ASVs and there is a lot of debate out there on each of them and the contexts where they are most useful.</p>
<p>I compiled my thoughts from readings on this debate of</p>
<ul>
<li>clustering 97% identity (= OTUs) vs</li>
<li>using raw denoised amplicon sequences (ASV)</li>
</ul>
<p>To figure out scenarios where each is more relevant, with links for further reading.<br>
Look at my <a href="https://docs.google.com/presentation/d/17aro50YRmPq0sMHzjGNwyIXOuvBc0BHzv-PNpaTxmTY/edit#slide=id.p">google slide ppt</a> for detailed explanation and introduction to the NGS sequencing</p>
<h2 id="synthesis">Synthesis</h2>
<p>ASVs are good when data quality is higher (lower sequencing noise), you want higher taxonomic resolution (species -&gt; strain level?). Most importantly the analysis using ASVs should not hinge on diversity metrics of individual organisms in the sample, and you must be aware that you could be looking at different ASVs covering intra-genomic variation in your data (multiple distinct 16s copies in the same organism, ex: <em>E. coli</em> has 7 copies)</p>
<p>If you choose to go with a higher taxonomic level of data and are willing to trade off with ease of re-usability/reproducibility of data without reanalysis and fresh OTU formation then do OTUs with the best method. Among methods to generate OTUs, swarm like clustering methods are better than either open or closed references as Pat Schloss (<em>Mothur’s creator</em>) points out - <em>closed reference ones are more stable, independent of data (though dependent of reference databases, which also update frequently)</em></p>
<p>If you want more stable OTUs, and it is very important to compare across a huge number of independently analyzed samples, considering using ASVs instead if</p>
<ul>
<li>you are dealing with environments that are rarely represented in databases (everything except gut bacteria?). Since closed ref OTUs will throw away these sequences not in the database. <a href="https://www.nature.com/articles/nature24621">Earth microbiome project</a></li>
</ul>
<h2 id="chain-of-thoughts">Chain of thoughts</h2>
<ol>
<li><a href="https://www.nature.com/articles/ismej2017119">Exact sequence variants should replace operational taxonomic units in marker-gene data analysis, ISMEj 2017</a>. Podcast covering the major arguments in this paper <a href="https://bioinformatics.chat/amplicon-sequence-variants">bioinformatics.chat</a></li>
</ol>
<p>Claims supremacy of ASVs over OTUs in <strong>all scenarios</strong> – these claims have been rebutted by Pat Schloss (creator of <em>Mothur</em>) and frierlab making the debate more nuanced.</p>
<h3 id="key-advantages"><em>Key advantages</em></h3>
<ul>
<li>ASVs are consistent labels with biological meaning, independent of datasets or reference database. This enables comparison of these ASVs across studies, reproducibility in future datasets and meta-analysis across labs.
<ul>
<li>The biological meaning still holds even if intra-genomic variation (multiple copies of 16s in same organism) splits an organism into multiple ASVs – given that data is interpreted not just by numbers through diversity, richness etc. but by presence/absence of particular ASVs across datasets.</li>
<li>
<blockquote>
<p>For analysis to be reproducible the fundamental units must be reproducible, and <em>de novo</em> OTUs are not. For analysis to be comprehensive the fundamental units must be comprehensive, and closed-reference OTUs are not. Replacing OTUs with ASVs makes marker-gene sequencing more precise, reusable, reproducible and comprehensive</p>
</blockquote>
</li>
</ul>
</li>
<li>ASVs have the <strong>potential to have higher resolution</strong> (single nucleotide), given high quality error free data. Given full length 16s reads (<em>Pacbio, nanopore</em> etc) can even distinguish strains within species by differentiating intra-genomic variation of the multiple copies of the gene within the same organism.
<ul>
<li>A criticism here</li>
</ul>
<blockquote>
<p>It is <strong>not biologically plausible</strong> to entertain the possibility that different <em>rrn</em> operons from the same genome would have different ecologies.  <a href="https://journals.asm.org/doi/10.1128/mSphere.00191-21">Amplicon Sequence Variants Artificially Split Bacterial Genomes into Separate Clusters, ASM, 2021</a></p>
</blockquote>
<ul>
<li>One important point is that if the goal was higher,sub-species resolution in the first place, one should not be doing a 16s subregion analysis at all and should probably choose a different (functional) marker gene for the species of interest <a href="s" title="find the tweet and cite">cite?</a></li>
</ul>
</li>
<li>ASVs don’t impose arbitrary dissimilarity thresholds, hence should be easier to apply to other marker genes?</li>
</ul>
<h3 id="major-criticism-of-asvs"><em>Major criticism of ASVs</em></h3>
<ul>
<li>While removing sequencing noise, very low abundance sequences (~ 1-5?) are discarded considering them likely to be sequencing artifacts/contamination etc. If you are interested in rare organisms, these methods will remove the singletons. This is also known to contribute to a much lower estimates of alpha diversity (richness, by 10x) compared to OTU methods
<ul>
<li>
<p>DADA2 defends itself by giving an option to pool samples prior to error estimations. That way if there is a real rare variant that occurs in multiple samples at low abundances, the pooled sample will have enough of these sequences to be able to use them meaningfully and not discarded. An interesting hybrid pseudo pooling approach seems to be the most recommended for speed <a href="https://benjjneb.github.io/dada2/pseudo.html#pseudo-pooling">Pseudo-pooling documentation</a></p>
</li>
<li>
<blockquote>
<p>their primary gain for noise is that they remove singletons. and there are huge problems with doing that. also note that if ASVs do what they claim, you could end up putting operons from the same chromosome into separate bins. <a href="https://twitter.com/PatSchloss/status/1111274111748227072?s=20">Pat Schloss tweet</a></p>
</blockquote>
<ul>
<li>Is it really the primary gain? They are also ironing out sequencing noise when sequences are abundant <em>‘enough’</em> – this I will argue is a way of clustering the noise out of sequences (<em>which originated from the same parent</em>) but in a statistically sound way using a NULL distribution rather than arbitrary similarity cutoffs. I think you are putting too much trust into the singletons…</li>
</ul>
</li>
<li>
<p>DADA2 proponents are also more prone to believing that such singletons are more likely to be false positives, and OTU based methods which take these into their <em>‘inflated’</em> diversity estimates don’t have any sound basis for believing that these are real sequences. <a href="https://twitter.com/AstrobioMike/status/1163742917166374912?s=20">Mike Lee tweet</a></p>
<ul>
<li>Some believe that this is a <strong>no man’s land</strong> where one has to choose between retaining false positives vs incurring false negatives and the decision depends on the research question <em>(see the closing twitter comment)</em></li>
</ul>
</li>
</ul>
</li>
<li>ASVs are sensitive to intra-genomic and intra-species variation (multiple copies of 16s in same organism) which could split an organism into multiple ASVs.
<ul>
<li>So in this case each ASV does not correspond to each species but a higher resolution. The rest of the analysis is still valid as long as the researchers keep this in mind, because these sequences are <strong>real</strong>.</li>
</ul>
</li>
</ul>
<h3 id="caveats-of-otus">Caveats of OTUs</h3>
<p>Only the ones relevant to the context set above<br>
Pat Scholss writes this -</p>
<blockquote>
<p>Furthermore, the 16S rRNA gene does not evolve at the same rate across all bacterial lineages (15), which limits the biological interpretation of a common OTU definition. A distance-based definition of a taxonomic unit based on 16S rRNA gene or full-genome sequences is operational and not necessarily grounded in biological theory  <a href="https://journals.asm.org/doi/10.1128/mSphere.00191-21">Amplicon Sequence Variants Artificially Split Bacterial Genomes into Separate Clusters, ASM, 2021</a></p>
</blockquote>
<ol start="2">
<li><a href="http://fiererlab.org/2017/05/02/lumping-versus-splitting-is-it-time-for-microbial-ecologists-to-abandon-otus/">http://fiererlab.org/2017/05/02/lumping-versus-splitting-is-it-time-for-microbial-ecologists-to-abandon-otus/</a></li>
</ol>
<blockquote>
<p>Clearly ESV approach is not inherently better than traditional OTU based approach. If your data is <strong>high quality</strong>, you want <strong>improved taxonomic resolution</strong>, and you are <em>not concerned</em> about the <strong>intra-genomic heterogeneity</strong> in the targeted marker genes, an <strong>ESV-based approach could be advantageous</strong>. Otherwise, a more standard OTU-based approach might be your best bet.</p>
</blockquote>
<p>This does not address the reproducibility aspect of de-novo OTUs though…</p>
<ol start="3">
<li>I found a OTU vs ASV debate in Mothur community with some nice links - <a href="https://forum.mothur.org/t/otus-vs-asvs/3506">https://forum.mothur.org/t/otus-vs-asvs/3506</a>. This highly linked reply is useful</li>
</ol>
<blockquote>
<p>I would like to revive this topic, after reading the ISME publication by the DADA2 people (<a href="https://www.nature.com/articles/ismej2017119">https://www.nature.com/articles/ismej2017119 73</a>). It is my opinion that this does merit some more conceptual attention on the mothur forum. Although the discussion has been started before on this forum (e.g. <a href="https://forum.mothur.org/t/otus-or-sequences/2975/6">OTUs or sequences? 17</a> on ASVs and oligotyping, <a href="https://forum.mothur.org/t/otu-classification-and-minimum-entropy-decomposition/2867">OTU classification and minimum entropy decomposition 17</a> on MED), I feel that there is no consensus on when to use the one or the other.<br>
In the past my experience with ESV’s has been limited to oligotyping to a specific taxon (on relatively abundant OTU representing it) among different conditions and sometimes seeing different oligotypes popping up between conditions (i.e. the use as suggested by <a href="https://forum.mothur.org/u/dwaite">@dwaite</a> in <a href="https://forum.mothur.org/t/otu-classification-and-minimum-entropy-decomposition/2867">OTU classification and minimum entropy decomposition 17</a>). …<br>
For instance: what is the prevailing opinion on <strong><em>ecological</em> consistency/robustness</strong> (e.g. <a href="https://www.ncbi.nlm.nih.gov/pubmed/24763141">https://www.ncbi.nlm.nih.gov/pubmed/24763141 27</a> and <a href="https://msphere.asm.org/content/2/2/e00073-17">https://msphere.asm.org/content/2/2/e00073-17 29</a>) and <strong><em>biological</em> conistency/interpretation</strong> of OTUs vs. ASVs (e.g. <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5812548/">https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5812548/ 42</a> and the discussion between Schloss and Edgar stated above)? For instance: How well does the assumption that “biological sequences are more likely to be repeatedly observed than are error-containing sequences” (Callahan <em>et al</em>. (2017) ISME) hold, e.g. in the case of the divergence in rRNA operons as stated in the comment by Fierer <em>et al</em>.?<br>
I agree that it probably doesn’t make sense to use a marker gene study to distinguish pathogenic species/strains from others in the same genus based on ASV/ESV’s (given the resolution of most single-marker genes and the read length of MiSeq). But I do think that the argument that is being made about interoperability/re-usability of ASV’s is one that deserves a second look.<br>
<em>As a side-note: with the advent of full-length 16S NGS amplicon seq (e.g. on the PacBio platform, <a href="https://peerj.com/articles/1869/">https://peerj.com/articles/1869/ 12</a>), with decreasing error rates, would ASV’s become more relevant ?</em></p>
</blockquote>
<ol start="4">
<li>Discussion of observed richness using DADA2 (with and without pooling) vs OTU and zOTUs in [qiime2 forums]. Has some nice twitter threads in the end (<a href="https://forum.qiime2.org/t/method-comparison-differences-in-observed-richness/8788/2">https://forum.qiime2.org/t/method-comparison-differences-in-observed-richness/8788/2</a>)</li>
</ol>
<hr>
<ol start="5">
<li>
<p><a href="https://academic.oup.com/bioinformatics/article/34/14/2371/4913809">Updating the 97% identity threshold for 16S ribosomal RNA OTUs, OUP, 2018</a><br>
Claims a higher threshold is required to preserve the correspondence between species and OTUs. Rebuttal below</p>
</li>
<li>
<p>Pat Schloss’ rebuttal and defense of the 97% OTU - <a href="http://www.academichermit.com/2017/10/11/Review-of-Updating-the-97-identity-threshold.html">Oct 11, 2017</a>; comments section in <a href="https://www.biorxiv.org/content/10.1101/192211v1#comment-3562391836">bioarxiv</a><br>
Key arguments :</p>
<ul>
<li>higher than 97% =&gt; splitting intraspecies and <a href="http://fiererlab.org/2017/10/09/intragenomic-heterogeneity-and-its-implications-for-esvs/">intragenomic</a> variations into different OTUs</li>
<li>Some species (ex: within <em>Bacillus</em> genus) are more finely split than others (<em>ex: Pseudomonas putida</em>) hence any constant clustering threshold is sub-optimal (is a pitch to the latest <strong>swarm</strong> kinds of algorithms?)</li>
<li>Denoisers (DADA2, … ) are aggressive in removing rare sequences that <em>may</em> be true</li>
</ul>
<blockquote>
<p>As several people have mentioned, DADA2 is happy to identify and report singletons. Just tell it to operate in pool=TRUE or pool=“pseudo” mode. <a href="https://twitter.com/bejcal/status/1163970976935268357?s=20">Callahan tweet</a></p>
</blockquote>
</li>
</ol>
<p>Pat Schloss mentions that Mothur was built for fully overlapping 2x250 bp Miseq reads, where errors are corrected by overlapping. Benjamin Callahan (DADA2 guy) <a href="https://twitter.com/bejcal/status/1163971599122538496?s=20">says</a> that Deblur was meant for 1x ~150 read hence the denoising becomes more relevant</p>
<ol start="6">
<li>
<p>Edgar’s <a href="https://drive5.com/usearch/manual/schloss_97_rebuttal.html">rebuttal</a> to Pat Schloss</p>
</li>
<li>
<p>Pat Schloss’ <a href="https://threadreaderapp.com/thread/1110509388664619009.html">tweets</a> promoting de-novo clustering</p>
</li>
</ol>
<blockquote>
<p>Use de-novo clustering (ex: <a href="https://github.com/torognes/swarm#refine_OTUs">Swarm</a>) instead of open (closed) clustering of OTUs - which do closed ref first and open later and have different OTU meanings within each of them. <a href="https://peerj.com/articles/1487/">Peerj, 2015</a></p>
</blockquote>
<p>Not sure if he means a fixed threshold 97% de novo clustering or dynamic methods like swarm</p>
<blockquote>
<p>Open/closed reference methods were developed because people have the type of crappy data that you get when reads don’t fully overlap. High error rates increase the number of uniques making downstream processing more difficult.</p>
</blockquote>
<p>Is this a case for de-noising when overlapping regions are not significant enough to correct for errors?</p>
<ol start="8">
<li>The Earth microbiome sequencing project endorses ASVs as opposed to OTUs. <a href="https://www.nature.com/articles/nature24621">A communal catalogue reveals Earth’s multiscale microbial diversity, Nature, 2017</a></li>
</ol>
<blockquote>
<p>Sequence analysis and taxonomic profiling were done initially using the common approach of assigning sequences to operational taxonomic units (OTUs) clustered by sequence similarity to existing rRNA databases. While this approach was useful for certain analyses, for many sample types, <em>especially plant-associated and free-living communities</em>, <strong>one-third of reads or more could not be mapped to existing rRNA databases</strong>. We therefore used a reference-free method, Deblur <a href="s" title="Amir, A. et al. Deblur rapidly resolves single-nucleotide community sequence patterns. mSystems 2, e00191–16 (2017)">21</a>, to remove suspected error sequences and provide single-nucleotide resolution ‘sub-OTUs’, also known as ‘amplicon sequence variants’</p>
</blockquote>
<blockquote>
<p>particularly the use of exact sequences instead of clustered operational taxonomic units, enable bacterial and archaeal ribosomal RNA gene sequences to be followed across multiple studies and allow us to explore patterns of diversity at an unprecedented scale</p>
</blockquote>
<blockquote>
<p>Because exact sequences are stable identifiers, unlike OTUs, they can be compared to any 16S rRNA or genomic database now and in the future, thereby promoting reusability</p>
</blockquote>
<ol start="9">
<li>Nice  <a href="https://twitter.com/AstrobioMike/status/1163742917166374912">twitter</a> debate about the best way to remove spurious ‘OTUs’</li>
</ol>
<p>There is an assertion that singletons are most likely spurious since there was some small mock community (20 members) which gave 100s of OTUs. <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5631090/">RC Edgar, peerj, 2014</a></p>
<p>Two approaches:</p>
<ul>
<li>Pat Schloss’ rarefying to a common number of sequences from a mock community vs</li>
<li>Dada2’s use of low abundance as a key to discard spurious ‘ASVs’ caused by sequencing errors which cannot be corrected since low abundance does not provide enough information to fit to their distribution</li>
</ul>
<p>People questioning rarefaction: <a href="https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003531">Waste Not, Want Not: Why Rarefying Microbiome Data Is Inadmissible, Plosone, 2014</a></p>
<ol start="10">
<li><a href="https://journals.asm.org/doi/full/10.1128/mSphere.00148-18">Broadscale Ecological Patterns Are Robust to Use of Exact Sequence Variants versus Operational Taxonomic Units</a></li>
</ol>
<blockquote>
<p>Despite quantitative differences in microbial richness, we found that all α and β diversity metrics were highly positively correlated (<em>r</em> &gt; 0.90) between samples analyzed with both approaches. Moreover, the community composition of the dominant taxa did not vary between approaches. Consequently, statistical inferences were nearly indistinguishable.</p>
</blockquote>
<blockquote>
<p>OTU classifications remain biologically useful for comparing diversity across large data sets (<a href="https://journals.asm.org/doi/full/10.1128/mSphere.00148-18#B7">7</a>) or identifying clades that share traits (<a href="https://journals.asm.org/doi/full/10.1128/mSphere.00148-18#B16">16</a>).</p>
</blockquote>
<blockquote>
<p>Thus, while there are good reasons to employ ESVs, we need not question the validity of results based on OTUs.</p>
</blockquote>
<p>I  guess the main criticism is not the validity of OTU based results within a experiment, it is the loss of reproducibility and of comparibility to other datasets by the community when OTUs are used as opposed to ASVs. So the knowledge produced by this analysis effort is limited in some way</p>
<ol start="12">
<li>Closing comment! <a href="https://twitter.com/_Faecal_Matters/status/1366512645835284483?s=20">twitter, March 1, 2021</a></li>
</ol>
<blockquote>
<p>I see we’re still having this ASV vs OTU debate then… People, come together, both approaches are wrong. It just depends in which direction you prefer your wrongness to be in. Can we please move on now, and let people use whichever they want.</p>
</blockquote>
<hr>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

