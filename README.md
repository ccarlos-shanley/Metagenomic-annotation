# Metagenomic-annotation
# For co-assembling and MAGs analysis follow: 1 - 3 - 4 -

# 1. Quality filtering/trimming raw reads using fastp 0.19.4
fastp -i SAMPLE1_R1_001.fastq.gz -I SAMPLE1_R2_001.fastq.gz -o SAMPLE1_R1_trimmed.fastq -O SAMPLE2_R2_trimmed.fastq

# 2. removing host reads using bbmap version (use output from 1)


# 3. taxonomic classification of the reads with Kaiju 1.7.3 (use output from 1 or 2)
kaiju -a mem -z 14 -t ~/databases/kaiju/nodes.dmp -f ~/databases/kaiju/kaiju_db_nr_euk.fmi -i SAMPLE1_R1_trimmed.fastq -j SAMPLE1_R2_trimmed.fastq -o SAMPLE1_kaiju

kaiju2table -t ~/databases/kaiju/nodes.dmp -n ~/databases/kaiju/names.dmp SAMPLE1_kaiju SAMPLE2_kaiju SAMPLE3_kaiju ... -r genus -o kaiju_table_genus

# 4. co-assembling using MEGAHIT v1.1.3 (use output from 1 or 2)
cat SAMPLE1_R1_trimmed.fastq SAMPLE2_R1_trimmed.fastq SAMPLE3_R1_trimmed.fastq ... > combined_R1.fastq
cat SAMPLE1_R2_trimmed.fastq SAMPLE2_R2_trimmed.fastq SAMPLE3_R2_trimmed.fastq ... > combined_R2.fastq

megahit -t 28 -1 combined_R1.fastq -2 combined_R2.fastq -o coassembly

# 5. individual assembling with MEGAHIT v1.1.3 (use output from 1 or 2)
megahit -t 28 -1 SAMPLE1_R1_trimmed.fastq -2 SAMPLE1_R2_trimmed.fastq -o SAMPLE1

# 6. gene calling with prokka 1.12 (use output from 4 or 5)
prokka --metagenome --compliant --fast --norrna --notrna --noanno --mincontiglen 1000 --cpus 28 SAMPLE1/final.contigs.fasta --outdir SAMPLE1_prokka

# 7. ko annotation with kofamscan (use output from 6)
./exec_annotation -f mapper -c config.yml -o SAMPLE1.ko SAMPLE1_prokka/PROKKA_XXXXXXXX.faa

# 8. combine ko annotation files from different metagenomes into a table using a custom script (output from 7)
./file_column_map.py SAMPLE1.ko SAMPLE2.ko SAMPLE3.ko ... > counts.csv

# 9. binning using MaxBin-2.2.7

# 10. checking quality of bins


