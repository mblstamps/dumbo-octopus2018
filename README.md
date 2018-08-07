# dumbo-octopus2018
# Analysis of the DiabImmune T1D data
The data can be found on the diabImmune website [https://pubs.broadinstitute.org/diabimmune/t1d-cohort]

We conducted analyses of the metagenomic data and subsets of the 16S data.
The code used for these analyses can be found below

## Code Used
### Metagenomics
Mike and Natasha 

### 16S
#import the data
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path fastq_manifest.csv \
  --output-path diabimmune-demux.qza \
  --source-format PairedEndFastqManifestPhred33

#use dada2 for denoising

qiime dada2 denoise-paired \
  --i-demultiplexed-seqs paired-end-demux.qza \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  --p-trunc-len-f 0 \
  --p-trunc-len-r 0 \
  --o-table diabimmune_table.qza \
  --o-representative-sequences diabimmune_rep-seqs.qza \
  --o-denoising-stats denoising-stats.qza
 
#taxonomy analysis

qiime feature-classifier classify-sklearn \
  --i-classifier gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads wanted_samples_rep_seqs.qza \
  --o-classification taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
  
qiime taxa barplot \
  --i-table wanted_samples_table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file diabimmune_t1d_16s_metadata.txt \
  --o-visualization taxa-bar-plots.qzv
  
#generate a fasttree (we wouldn't use this in real life, but we were short on time at this point)

qiime alignment mafft \
  --i-sequences wanted_samples_rep_seqs.qza \
  --o-alignment aligned-rep-seqs.qza
  
qiime alignment mask \
  --i-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza

qiime phylogeny fasttree \
  --i-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza
  
qiime phylogeny midpoint-root \
  --i-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
