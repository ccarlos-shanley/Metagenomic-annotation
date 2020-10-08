# Metagenomic-annotation worklow for co-assembling and MAG analysis
#### When running in the TXST LEAP cluster load the python module
```
module load opt-python
```

### 1. Quality filtering/trimming raw reads using fastp 0.19.4

```
fastp -i SAMPLE1_R1_001.fastq.gz -I SAMPLE1_R2_001.fastq.gz -o SAMPLE1_R1_trimmed.fastq -O SAMPLE1_R2_trimmed.fastq
```

### 2. taxonomic classification of the reads with Kaiju 1.7.3
```
kaiju -a mem -z 14 -t ~/databases/kaiju/nodes.dmp -f ~/databases/kaiju/kaiju_db_nr_euk.fmi -i SAMPLE1_R1_trimmed.fastq -j SAMPLE1_R2_trimmed.fastq -o SAMPLE1_kaiju

kaiju2table -t ~/databases/kaiju/nodes.dmp -n ~/databases/kaiju/names.dmp SAMPLE1_kaiju SAMPLE2_kaiju SAMPLE3_kaiju ... -r genus -o kaiju_table_genus
```
### 3. co-assembling using MEGAHIT v1.1.3 (use output from 1)
```
cat SAMPLE1_R1_trimmed.fastq SAMPLE2_R1_trimmed.fastq SAMPLE3_R1_trimmed.fastq ... > combined_R1.fastq
cat SAMPLE1_R2_trimmed.fastq SAMPLE2_R2_trimmed.fastq SAMPLE3_R2_trimmed.fastq ... > combined_R2.fastq


megahit -t 28 -1 combined_R1.fastq -2 combined_R2.fastq -o coassembly
```

### 4. gene calling on coassembled contigs with prokka 1.12
```
prokka --metagenome --compliant --fast --norrna --notrna --noanno --mincontiglen 1000 --cpus 28 coassembly/final.contigs.fasta --outdir Scoassembly_prokka
```

### 5. ko annotation of coassembled contigs with kofamscan
```
exec_annotation -f mapper -c config.yml -o coassembly.ko coassembly_prokka/PROKKA_XXXXXXXX.faa
```

### 6. Taxonomic annoations of coassembled contigs using CAT 5.1.2
```
python3.6 ~/CAT-5.1.2/CAT_pack/CAT contigs -d ~/databases/CAT_prepare_20200618/2020-06-18_CAT_database/ -t ~/databases/CAT_prepare_20200618/2020-06-18_taxonomy/ -c coassembly/final.contigs.fa -o coassembly

python3.6 ~/CAT-5.1.2/CAT_pack/CAT add_names -t /home/c_c947/databases/CAT_prepare_20200618/2020-06-18_taxonomy/ --only_official -i coassembly.CAT.contig2classification.txt -o coassembly.CAT.contig2classification.official_names.txt 

python3.6 ~/CAT-5.1.2/CAT_pack/CAT summarise -c beetle_coassemble_metagenome/final.contigs.fa -i coassembly.CAT.contig2classification.official_names.txt -o CAT_coassembly.summary.txt
```

### 7. binning coassembled contigs using concoct 1.1.0
```
bowtie2-build -f coassembly/final.contigs.fasta contigs

bowtie2 -p 24 -1 combined.R1.fastq -2 combined.R2.fastq -x contigs -S mapped.sam

samtools sort -O bam mapped.sam -o mapped_sorted.bam 

samtools index mapped_sorted.bam

cut_up_fasta.py coassembly/final.contigs.fasta -c 10000 -o 0 --merge_last -b contigs_10K.bed > contigs_10K.fa

concoct_coverage_table.py contigs_10K.bed mapped_sorted.bam > coverage_table.tsv

concoct --composition_file contigs_10K.fa -t 24 --coverage_file coverage_table.tsv -b concoct_output/

merge_cutup_clustering.py concoct_output/clustering_gt1000.csv > concoct_output/clustering_merged.csv

mkdir concoct_output/fasta_bins
extract_fasta_bins.py coassembly/final.contigs.fasta concoct_output/clustering_merged.csv --output_path concoct_output/fasta_bins
```

### 8. checking quality of bins with checkM
```
checkm lineage_wf -t 28 concoct_output/fasta_bins/ coassembled_bins_checkm -x fa
```

### 9. mapping and quantifying sample reads to coassembled contigs
```
bowtie2 -p 24 -1 SAMPLE1.fastq -2 SAMPLE1.R2.fastq -x contigs -S SAMPLE1.mapped.sam

samtools sort -O bam SAMPLE1.mapped.sam -o SAMPLE1.mapped_sorted.bam 

samtools index SAMPLE1.mapped_sorted.bam

samtools idxstats SAMPLE1.mapped_sorted.bam > SAMPLE1.mapped_sorted.tab
```

### 10. Taxonomic annoations of coassembled BINS using CAT 5.1.2
```
python3.6 ~/CAT-5.1.2/CAT_pack/CAT bins -d ~/databases/CAT_prepare_20200618/2020-06-18_CAT_database/ -t ~/databases/CAT_prepare_20200618/2020-06-18_taxonomy/ -c concoct_output/fasta_bins/ -o bins

python3.6 ~/CAT-5.1.2/CAT_pack/CAT add_names -t /home/c_c947/databases/CAT_prepare_20200618/2020-06-18_taxonomy/ --only_official -i bins.CAT.contig2classification.txt -o bins.CAT.contig2classification.official_names.txt 
```

### 11. For Single Nucleotide Variants (SNVs) analyses --> follow Anvio' tutorial: http://merenlab.org/2015/07/20/analyzing-variability/
