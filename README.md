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
### 4. individual assembling with MEGAHIT v1.1.3
```
megahit -t 28 -1 SAMPLE1_R1_trimmed.fastq -2 SAMPLE1_R2_trimmed.fastq -o SAMPLE1
```
### 5. gene calling with prokka 1.12
```
prokka --metagenome --compliant --fast --norrna --notrna --noanno --mincontiglen 1000 --cpus 28 SAMPLE1/final.contigs.fasta --outdir SAMPLE1_prokka
```

### 6. ko annotation with kofamscan
```
exec_annotation -f mapper -c config.yml -o SAMPLE1.ko SAMPLE1_prokka/PROKKA_XXXXXXXX.faa
```
### 7. combine ko annotation files from different metagenomes into a table using a custom script
```
./file_column_map.py SAMPLE1.ko SAMPLE2.ko SAMPLE3.ko ... > counts.csv
```
### 8. binning using concoct


### 10. checking quality of bins


