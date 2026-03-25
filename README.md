# convert-sra-to-fastq
SRA是NCBI上下载的转录组原始数据格式

#!/bin/bash
#SBATCH -p amd_512
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 16

# mamba create -n sra_env -c bioconda -c conda-forge sra-tools -y

module load miniforge/24.11
source activate /public3/home/scg4618/.local/share/mamba/envs/sra_env

cd /public3/home/scg4618/thk/RNA-seq

mkdir -p fastq tmp
for f in *.1 *.lite.1; do
    srr=$(basename $f | cut -d'.' -f1)
    echo "Processing $srr ..."
    
    fasterq-dump $srr \
        --split-files \
        -e 16 \
        --temp ./tmp \
        -O fastq
    
    pigz -p 12 fastq/${srr}_*.fastq
done
