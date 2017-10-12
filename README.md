# ACE Amplicon Pipeline Development

# QIIME2 for feature (OTU) table & taxonomy

## TO MAKE CACHED MEMORY FREE:
sysctl -w vm.drop_caches=3

## TO ACTIVATE QIIME2:

module load miniconda3
source activate /srv/sw/qiime2/
source activate /srv/sw/qiime2/qiime2-2017.7/


## Make .qza file all paired-end reads
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path paired_reads/ --source-format CasavaOneEightSingleLanePerSampleDirFmt --output-path paired-end.qza

## make feature-table (OTU-Table)
qiime dada2 denoise-paired --i-demultiplexed-seqs paired-end.qza --o-table table.qza --o-representative-sequences rep-seqs.qza --p-trunc-len-f 280 --p-trunc-len-r 280

## make table visualization file
qiime feature-table summarize --i-table test-table.qza --o-visualization test-table-seqs.qzv

## make table sequences file
qiime feature-table tabulate-seqs   --i-data test-rep-seqs.qza   --o-visualization test-rep-seqs.qzv

## export data
qiime tools export test-rep-seqs.qzv --output-dir test-output-data

## taxonomy assignment
### https://docs.qiime2.org/2017.9/tutorials/feature-classifier/

### make .qza file format

qiime tools import --type 'FeatureData[Sequence]' --input-path ref_db.fasta --output-path ref_db.qza
qiime tools import --type 'FeatureData[Taxonomy]' --source-format HeaderlessTSVTaxonomyFormat --input-path ref_db_taxonomy.txt --output-path ref-taxonomy.qza

### Extract reference reads

qiime feature-classifier extract-reads \
  --i-sequences ref_db.qza \
  --p-f-primer GTGCCAGCMGCCGCGGTAA \
  --p-r-primer GGACTACHVGGGTWTCTAAT \
  --p-trunc-len 100 \
  --o-reads ref-seqs.qza
  
### Train the classifier

qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads ref-seqs.qza \
  --i-reference-taxonomy ref-taxonomy.qza \
  --o-classifier classifier.qza
  
### Test the classifier

qiime feature-classifier classify-sklearn \
  --i-classifier classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv

## export data
qiime tools export taxonomy.qzv --output-dir test-output-taxonomy


