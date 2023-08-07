## Amplicon analysis with QIIME2

This pipeline was run in Linux (terminal on iOS).

### Conda and Miniconda
To run Qiime, we need to run Conda first

Installation of Conda can be done from the [Conda website](https://docs.conda.io/projects/conda/en/latest/user-guide/install/)

```
conda install conda-build
```

It is important to corroborate Conda is updated

```
conda update conda
conda update conda-build
```

And now, **Qiime2** 

### QIIME2

Here is the website to install [Qiime2](https://docs.qiime2.org/2020.11/)
The documentation includes detailed information about QIIME and how it works 

QIIME 2 can be installed natively or using virtual machines using the conda environment, instructions are detailed [here](https://docs.qiime2.org/2020.11/install/)

1. Open the terminal (Linux environment)

_Tip: Some iOS updates set the **zsh** shell profile by default, we need to work on the bash environment (instead of zsh)_
```
chsh -s /bin/bash
```

2. Create metadata (manifest)

The manifest file is a text document with the following format: 
sample-id,absolute-filepath,direction

_Below is an example of a manifest file, it is important to notice there are not blank spaces, not on the document nor on the file name_
_sample-id,absolute-filepath,direction_
>SampleS7,/Users/user/Documents/Postdoc/Sequencing/Southport/S7_S13_L001_R2_001.fastq,forward

3. Activate QIIME2
```
conda activate qiime2-2020.8
```

4. Import metadata
```
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path /Users/user/Documents/Southport/Sequencing/Parys_Mountain/manifest_PM.txt --output-path paired-end-demuxl.qza --input-format PairedEndFastqManifestPhred33
```
_Notice that the input path will be different for each set of samples_

5. Visualize sequences
```
qiime demux summarize --i-data paired-end-demuxl.qza --p-n 2485316 --o-visualization demuxl.qzv 
```
Notes:
This command creates a file **demuxl.qzv** 
This file was created and stored in the User folder (I set this folder by default)
The **n** represents the subsample size, that number was standardised for all the samples. 
This number is different for each set of samples based on the length of the sequences 
The file can now be dragged and visualized on [qiime2view](https://view.qiime2.org)
And then click on “Interactive Quality Plot”

This will show the quality score. Ideally, we want any sequence with higher quality than 33 (Phred) and that is why we set the commands for the next step, we will trim the edges of the sequences: 20 from the left and 220 to the right.

<img width="451" alt="image" src="https://github.com/srz11d/Amplicon_analysis_with_QIIME2/assets/135147161/2bffc28f-f7ec-4a24-bf2a-89fc51a3ebe7">

6. Denoising with Dada2
```
qiime dada2 denoise-paired --i-demultiplexed-seqs paired-end-demuxl.qza --p-trim-left-f 20 --p-trim-left-r 20 --p-trunc-len-f 220 --p-trunc-len-r 220 --o-representative-sequences rep-seqs-dada2l.qza --o-table table-dada2l.qza --o-denoising-stats stats-dada2l.qza
```

The following files are created:
>Saved FeatureTable[Frequency] to: table-dada2l.qza

>Saved FeatureData[Sequence] to: rep-seqs-dada2l.qza

>Saved SampleData[DADA2Stats] to: stats-dada2l.qza

>Saved FeatureTable[Frequency] to: table-dada2l.qza

>Saved FeatureData[Sequence] to: rep-seqs-dada2l.qza

>Saved SampleData[DADA2Stats] to: stats-dada2l.qza

_Notes_

**table-dada2.qza** This file includes the **table.biom** file (OTUs and number of reads from each one of them). This file can also be useful to determine alpha diversity on R (remember to transpose data and subsampling)

**rep-seqs-dada2.qza** List of OTUs detected on the document, these files can be used to Blast and will be used to assign taxonomy

**stats-dada2.qza** includes statistics of the sequences, number of reads, merged and chimeric reads, this information may be useful to report on the methods part


7. Obtain statistics
```
qiime metadata tabulate --m-input-file stats-dada2l.qza --o-visualization stats-dada2l.qzv
```
>Saved Visualization to: stats-dada2l.qzv

8. Assign taxonomy
To assign taxonomy we need to download the databases and store them on the folder we have set by default.
The available databases are Silva and Greengenes, both are found on the [QIIME2 documentation site](https://docs.qiime2.org/2020.11/data-resources/#taxonomy-classifiers-for-use-with-q2-feature-classifier)
<img width="451" alt="image" src="https://github.com/srz11d/Amplicon_analysis_with_QIIME2/assets/135147161/0f2446c5-bf89-42b7-a88e-60f8d7a16a9c">

Once the databases are installed, we can run the following commands (depending on the database used)
```
qiime feature-classifier classify-sklearn --i-classifier silva-138-99-nb-classifier.qza --i-reads rep-seqs-dada2l.qza --o-classification taxonomyl.qza
```
or
```
qiime feature-classifier classify-sklearn --i-gg-13-8-99-nb-classifier.qza --i-reads rep-seqs-dada2l.qza --o-classification taxonomyl.qza
```

The following file was created:
Saved FeatureData[Taxonomy] to: **taxonomyl.qza**

9. Classification by level 

The **Taxonomy** file can be difficult to handle, hence we can create files classified by taxonomic level

Level 1: Domain
```
qiime taxa collapse \
  --i-table table-dada2l.qza\
  --i-taxonomy taxonomyl.qza \
  --p-level 1 \
  --o-collapsed-table feature-table-level1l.qza
```

Level 2: phylum
```
qiime taxa collapse \
  --i-table table-dada2l.qza\
  --i-taxonomy taxonomyl.qza \
  --p-level 2 \
  --o-collapsed-table feature-table-level2l.qza
```

Level 3: class
```
qiime taxa collapse \
  --i-table table-dada2l.qza\
  --i-taxonomy taxonomyl.qza \
  --p-level 3 \
  --o-collapsed-table feature-table-level3l.qza
```

Level 4: order
```
qiime taxa collapse \
  --i-table table-dada2l.qza\
  --i-taxonomy taxonomyl.qza \
  --p-level 4 \
  --o-collapsed-table feature-table-level4l.qza
```

Level 5: family
```
qiime taxa collapse \
  --i-table table-dada2l.qza\
  --i-taxonomy taxonomyl.qza \
  --p-level 5 \
  --o-collapsed-table feature-table-level5l.qza
```

Level 6: genus
```
qiime taxa collapse \
  --i-table table-dada2l.qza\
  --i-taxonomy taxonomyl.qza \
  --p-level 6 \
  --o-collapsed-table feature-table-level6l.qza
```

Level 7: species
```
qiime taxa collapse \
  --i-table table-dada2l.qza\
  --i-taxonomy taxonomyl.qza \
  --p-level 7 \
  --o-collapsed-table feature-table-level7l.qza
```

The **qza** files can be opened with a decompressor (I used **The Unarchiver** for MacOS).
Inside those folders we can find a **.biom** file, that’s the file we need

10. Conversion to readable tables

_Note: the **biom** conversion needs to be run on the qiime2 environment_

```
biom convert -i feature-table_level1l.biom -o table.from_biom_level1l.txt --to-tsv
```
```
biom convert -i feature-table_level2l.biom -o table.from_biom_level2l.txt --to-tsv
```
```
biom convert -i feature-table_level3l.biom -o table.from_biom_level3l.txt --to-tsv
```
```
biom convert -i feature-table_level4l.biom -o table.from_biom_level4l.txt --to-tsv
```
```
biom convert -i feature-table_level5l.biom -o table.from_biom_level5l.txt --to-tsv
```
```
biom convert -i feature-table_level6l.biom -o table.from_biom_level6l.txt --to-tsv
```
```
biom convert -i feature-table_level7l.biom -o table.from_biom_level7l.txt --to-tsv
```

11. End the process
```
conda deactivate
```
