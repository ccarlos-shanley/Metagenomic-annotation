# Metagenomic-annotation worklow for co-assembling and MAG analysis

### 1. Quality filtering/trimming raw reads using fastp 0.19.4

```
fastp -i SAMPLE1_R1_001.fastq.gz -I SAMPLE1_R2_001.fastq.gz -o SAMPLE1_R1_trimmed.fastq -O SAMPLE2_R2_trimmed.fastq
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

### 4. gene calling with prokka 1.12
```
prokka --metagenome --compliant --fast --norrna --notrna --noanno --mincontiglen 1000 --cpus 28 coassembly/final.contigs.fasta --outdir Scoassembly_prokka
```

### 5. ko annotation with kofamscan
```
exec_annotation -f mapper -c config.yml -o coassembly.ko coassembly_prokka/PROKKA_XXXXXXXX.faa
```

### 6. binning using concoct
```
bowtie2-build -f coassembly/final.contigs.fasta contigs

bowtie2 -p 24 -1 combined.R1.fastq -2 combined.R2.fastq -x HID1973K_combined_bin01 -S mapped.sam

samtools sort -O bam mapped.sam -o mapped_sorted.bam 

samtools index mapped_sorted.bam

cut_up_fasta.py coassembly/final.contigs.fasta -c 10000 -o 0 --merge_last -b contigs_10K.bed > contigs_10K.fa

concoct_coverage_table.py contigs_10K.bed mapped_sorted.bam > coverage_table.tsv

concoct --composition_file contigs_10K.fa -t 24 --coverage_file coverage_table.tsv -b concoct_output/

merge_cutup_clustering.py concoct_output/clustering_gt1000.csv > concoct_output/clustering_merged.csv

mkdir concoct_output/fasta_bins
extract_fasta_bins.py coassembly/final.contigs.fasta concoct_output/clustering_merged.csv --output_path concoct_output/fasta_bins
```

### 7. checking quality of bins

checkm lineage_wf -t 28 concoct_output/fasta_bins/ coassembled_bins_checkm -x fa
