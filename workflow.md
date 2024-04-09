# Prelude

This is a potential workflow for CMSC, BJE, MK and TP to build a chromosome scale reference for the allotetraploid X. mellotropicalis. During my PhD, I tried different approach with overall limited success. The assemblies using part or all the data available at the time (short reads of different insert size: 150bp x 2 - 2nd being of low quality, 400bp, 1kb, 6kb, 10kb, Pacbio long reads) were very fragmented though allowed us to identified the sex determining system and sex chromosome.
Since then BJE got even more data. Collaborators in Germany obtained an assembly using linked reads with Supernova. Supernova is a diploid assembly, which means it tries to assemble maternal and paternal haplotypes separately. In our case it means for a given gene, we might get "4 copies" (2 alleles for both subgenomes). The assembly size is ~2.86Gb which is fairly closed to the expected size (~3.1Gb). BJE attempted to improve this assembly with a lot of different ways but the genome size always end up closer to the diploid size. BJE mapped the Supernova assembly against tropicalis and some regions seem duplicated which is a good sign. We will start with this Supernova assembly and try to scaffold it more.

# Scaffolding with Pacbio and MATE

[LINKS](https://github.com/bcgsc/LINKS) v1.8.5 since it is the recommended version for long reads scaffolding. Also starting at v.2.0.0 MPET are no long supported.
Scaffolding should start with the PacBio reads first (cannot be done at same time as mate pairs), followed by the mate pairs (6kb, 10kb libraries).

Here we are looking to increase the quality of the genome a little. I am hoping we will get some longer scaffolds which should help with the following step. I do not expect a crazy improvement.

# Scaffolding with family GBS 

This is I think the most important step and should hopefully allow us to improve the assembly a lot. We will use [allmaps](https://github.com/tanghaibao/jcvi/wiki/ALLMAPS). The installation might require some helps from server administrators (was needed previously for strawberries).

Before playing with this sofware we need to make multiple genetic maps. Previously I did paternal, maternal and combined. I think the minimal is maternal and paternal. BJE also has RNAseq data though I don't know who are the parents and if they were sequenced in any ways - if so then maybe we could call genotypes and make independant linkage maps using those data too.

To create the linkape maps, first map the GBS data using bwa, then call and filter using GATK. The obtained vcf file can directly be loaded into R::onemap. Some markers are directly dropped because not useful for genetic mapping. The raw onemap file can be saved. I usually use a python script to split the file into maternal and paternal markers depending on the marker type. Then I load each new file (maternal or paternal) separately (no need to load at the same time if big). If a lot of markers, drop markers that have missing data. Redundant markers might be dropped if file really too big though be careful because markers on different scaffolds might be redundant so in this case we lose useful info. Making genetic maps with onemap might require a lot of trying, changing parameters (see [tutorial for outcross](https://statgen-esalq.github.io/tutorials/onemap/Outcrossing_Populations.html). We should obtained a number of linkage groups similar to the number of expected chromosomes (20). If the number is a bit bigger it is fine but not smaller.

Once happy with the linkage maps, they should all (maternal and paternal maps togeter) be loaded along with the assembly as inputs to allmaps.

To check if the scaffolding has been successfull, we need to then map the obtained assembly against tropicalis using minimap. In a best case scenario, 2 chromosome scale scffolds will map to each trop chromosome.

# Scaffolding and identifying subgenomes with diploid reference X. tropicalis

Using the output of minimap, we can try to separate the scaffolds into potential subgenome with the identity column.

# Gap closing

That would be if we are happy. But multiple options can be considered. Having a lot of short read data we can use those at least to try to close the gaps. OPtions: for pacbio - [TGS-GapCloser](https://github.com/BGI-Qingdao/TGS-GapCloser), [pbjelly](https://sourceforge.net/projects/pb-jelly/), short reads - GMcloser, gapcloser. Will update when we know if the GBS scaffolding step worked well.
