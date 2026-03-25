# 🧬 SRA 批量转换 FASTQ（SLURM 集群版）

本流程基于 SRA Toolkit，实现：

✅ 批量 SRA → FASTQ
✅ 支持 `.1` / `.lite.1` 文件
✅ 多线程加速
✅ 自动压缩
✅ 适配 SLURM 集群

---

# 📦 一、环境准备

## 1️⃣ 创建独立环境（推荐）

```bash
mamba create -n sra_env -c bioconda -c conda-forge sra-tools pigz -y
```

## 2️⃣ 加载环境（服务器）

```bash
module load miniforge/24.11
source activate /public3/home/scg4618/.local/share/mamba/envs/sra_env
```

---

# 📂 二、目录结构

建议工作目录如下：

```
RNA-seq/
├── SRRxxxxxx.1
├── SRRxxxxxx.lite.1
├── run_sra.sh
├── fastq/
└── tmp/
```

---

# 🚀 三、运行脚本（SLURM）

将以下脚本保存为 `run_sra.sh`：

```bash
#!/bin/bash
#SBATCH -p amd_512
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 16

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
```

---

# ▶️ 四、提交任务

```bash
sbatch run_sra.sh
```

---

# ⚙️ 五、参数说明

| 参数              | 含义            |
| --------------- | ------------- |
| `--split-files` | 拆分双端 reads    |
| `-e 16`         | 使用 16 线程      |
| `--temp ./tmp`  | 临时目录（避免系统盘爆满） |
| `-O fastq`      | 输出目录          |
| `pigz -p 12`    | 多线程压缩         |

---

# 📌 六、输出结果

运行完成后：

```
fastq/
├── SRRxxxxxx_1.fastq.gz
├── SRRxxxxxx_2.fastq.gz
```

---

# ⚠️ 七、注意事项

## ❗ 1. 磁盘空间

SRA → FASTQ 会膨胀约：

```
1x SRA ≈ 3–5x FASTQ
```

---

## ❗ 2. `.lite` 文件说明

* `.lite.1` 为精简版 SRA
* 可能缺失部分质量信息
* 用于转录组分析（如 DESeq2）一般可接受

---

## ❗ 3. 临时目录建议

```bash
--temp ./tmp
```

否则可能占满 `/tmp`

---

## ❗ 4. 线程设置建议

| 资源      | 推荐             |
| ------- | -------------- |
| CPU 16核 | `-e 12~16`     |
| 压缩      | `pigz -p 8~12` |

---

# 🚀 八、扩展优化（推荐）

## ✅ 并行多个样本（提速）

可结合 GNU parallel 或 SLURM array

## ✅ 自动跳过已完成样本

避免重复计算

## ✅ 集成完整 RNA-seq 流程

```
SRA → FASTQ → fastp → HISAT2/STAR → featureCounts → DESeq2
```

---

# 📬 九、联系 / 扩展

如果你需要：

✅ SLURM array 并行版本
✅ 自动化 RNA-seq pipeline
✅ 结合 DESeq2 的全流程

可以进一步扩展本项目 🚀
