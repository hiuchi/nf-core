# NextflowでRNA-seq解析を行う

## 目次

### 1. 環境構築
- 1.1 Nextflowのインストール
- 1.2 `nf-core/rnaseq`と`nf-core/differentialabundance`のpull
- 1.3 SRA-toolkitのインストール
- 1.4 Dockerのインストール

### 2. 解析するデータの取得
- 2.1 解析するデータの紹介
- 2.2 データのダウンロードと変換

### 3. nf-core/rnaseqの実行
- 3.1 解析に必要なデータの準備
- 3.2 解析の実行
- 3.3 実行結果の確認

### 4. nf-core/differentialabundanceの実行
- 4.1 解析に必要なデータの準備
- 4.2 解析の実行
- 4.3 実行結果の確認

---
## 1. 環境構築
### 1.1 Nextflowのインストール
まずは以下の手順でNextflowをインストールします。

```bash
# Nextflowのインストール
curl -s https://get.nextflow.io | bash

# 実行可能にするため、PATHに追加
chmod +x nextflow
mkdir -p $HOME/.local/bin/
mv nextflow $HOME/.local/bin/

# インストールできたかどうかの確認
nextflow info
```

### 1.2 `nf-core/rnaseq`と`nf-core/differentialabundance`のpull
nf-coreのパイプラインは、`nextflow pull`を使用してローカル環境にpull（取得）できます。以下のコマンドを実行すると、`rnaseq`パイプラインをローカルにpullします。

```bash
# nf-core/rnaseqとnf-core/differentialabundanceパイプラインをpullする
nextflow pull nf-core/rnaseq
nextflow pull nf-core/differentialabundance
```

### 1.3 SRA-Toolkitのインストール
公共データをダウンロードするために必要なSRA-Toolkitをインストールします。
```bash
conda install -c bioconda sra-tools
```
### 1.4 Dockerのインストール
OSによってインストール方法が異なるので[公式ページ](https://matsuand.github.io/docs.docker.jp.onthefly/get-docker/)をご覧ください。
Singularityおよびcondaを用いて実行することもできますが、ここでは説明を省略します。

## 2. 解析するデータの取得
### 2.1 解析するデータの紹介
今回は新たに設計されたチロシンキナーゼ阻害剤APG-2449をヒト組織に投与し、反応があった群と反応がなかった群（各n=3）の比較を行います。

Fang DD, Tao R, Wang G, Li Y, Zhang K, Xu C, Zhai G, Wang Q, Wang J, Tang C, Min P, Xiong D, Chen J, Wang S, Yang D, Zhai Y. Discovery of a novel ALK/ROS1/FAK inhibitor, APG-2449, in preclinical non-small cell lung cancer and ovarian cancer models. BMC Cancer. 2022 Jul 11;22(1):752. doi: 10.1186/s12885-022-09799-4. PMID: 35820889; PMCID: PMC9277925.

### 2.2 データのダウンロードと変換
NCBIから上記のデータをダウンロードします。作業環境により数十分から数時間程度かかることがあります。

```bash
# データのダウンロード
mkdir fastq
cd fastq

# fastqファイルのダウンロード
for i in `seq 68 73` ; do fasterq-dump -p -e 8 SRR182738${i} --split-files ; done

# fastqを圧縮
gzip -v *.fastq

# pigzがインストールされている場合
pigz *.fastq
```

## 3. nf-core/rnaseqの実行
### 3.1 解析に必要なデータの準備
samplesheets_rnaseq.csvファイルにサンプル名とfastqファイルのパスを記述します。`strandedness`を`auto`とすることでstrandednessを自動判定してくれます。
ファイルは絶対パスで指定して下さい。
```
sample,fastq_1,fastq_2,strandedness
SRR18273868,/path/to/fastq/SRR18273868_1.fastq.gz,/path/to/fastq/SRR18273868_2.fastq.gz,auto
SRR18273869,/path/to/fastq/SRR18273869_1.fastq.gz,/path/to/fastq/SRR18273869_2.fastq.gz,auto
SRR18273870,/path/to/fastq/SRR18273870_1.fastq.gz,/path/to/fastq/SRR18273870_2.fastq.gz,auto
SRR18273871,/path/to/fastq/SRR18273871_1.fastq.gz,/path/to/fastq/SRR18273871_2.fastq.gz,auto
SRR18273872,/path/to/fastq/SRR18273872_1.fastq.gz,/path/to/fastq/SRR18273872_2.fastq.gz,auto
SRR18273873,/path/to/fastq/SRR18273873_1.fastq.gz,/path/to/fastq/SRR18273873_2.fastq.gz,auto
```

### 3.2 解析の実行
解析を実行します。今回はパイプラインのバージョン3.17.0、リファレンスゲノムはGRCh38、Dockerを用いて解析を実行します。

```
nextflow run nf-core/rnaseq \
	-r 3.17.0 \
	-profile docker \
	--input samplesheets_rnaseq.csv \
	--outdir quant \
	--fasta /path/to/genome/GRCh38.primary_assembly.genome.fa \
	--gtf /path/to/gencode.v46.basic.annotation.gtf \
	--gencode \
	--igenomes_ignore \
	--genome null \
	--skip_deseq2_qc \
    	--skip_dupradar \
    	--skip_biotype_qc
```

### 3.3 実行結果の確認

出力されたファイルを確認します。
```
tree -L 2
.
├── fastqc
│   ├── raw
│   └── trim
├── multiqc
│   └── star_salmon
├── pipeline_info
│   ├── execution_report_2024-12-12_17-53-26.html
│   ├── execution_report_2024-12-12_20-30-50.html
│   ├── execution_report_2024-12-12_20-40-11.html
│   ├── execution_timeline_2024-12-12_17-53-26.html
│   ├── execution_timeline_2024-12-12_20-30-50.html
│   ├── execution_timeline_2024-12-12_20-40-11.html
│   ├── execution_trace_2024-12-12_17-53-26.txt
│   ├── execution_trace_2024-12-12_20-29-13.txt
│   ├── execution_trace_2024-12-12_20-30-50.txt
│   ├── execution_trace_2024-12-12_20-40-11.txt
│   ├── execution_trace_2024-12-12_22-01-20.txt
│   ├── nf_core_rnaseq_software_mqc_versions.yml
│   ├── params_2024-12-12_17-53-38.json
│   ├── params_2024-12-12_20-29-25.json
│   ├── params_2024-12-12_20-31-03.json
│   ├── params_2024-12-12_20-40-22.json
│   ├── params_2024-12-12_22-01-32.json
│   ├── pipeline_dag_2024-12-12_17-53-26.html
│   ├── pipeline_dag_2024-12-12_20-30-50.html
│   └── pipeline_dag_2024-12-12_20-40-11.html
├── star_salmon
│   ├── bigwig
│   ├── deseq2_qc
│   ├── dupradar
│   ├── featurecounts
│   ├── log
│   ├── null.merged.gene_counts_length_scaled.SummarizedExperiment.rds
│   ├── null.merged.gene_counts_scaled.SummarizedExperiment.rds
│   ├── null.merged.gene_counts.SummarizedExperiment.rds
│   ├── null.merged.transcript_counts.SummarizedExperiment.rds
│   ├── picard_metrics
│   ├── qualimap
│   ├── rseqc
│   ├── salmon.merged.gene_counts_length_scaled.tsv
│   ├── salmon.merged.gene_counts_scaled.tsv
│   ├── salmon.merged.gene_counts.tsv
│   ├── salmon.merged.gene_lengths.tsv
│   ├── salmon.merged.gene_tpm.tsv
│   ├── salmon.merged.transcript_counts.tsv
│   ├── salmon.merged.transcript_lengths.tsv
│   ├── salmon.merged.transcript_tpm.tsv
│   ├── samtools_stats
│   ├── SRR18273868
│   ├── SRR18273868.markdup.sorted.bam
│   ├── SRR18273868.markdup.sorted.bam.bai
│   ├── SRR18273869
│   ├── SRR18273869.markdup.sorted.bam
│   ├── SRR18273869.markdup.sorted.bam.bai
│   ├── SRR18273870
│   ├── SRR18273870.markdup.sorted.bam
│   ├── SRR18273870.markdup.sorted.bam.bai
│   ├── SRR18273871
│   ├── SRR18273871.markdup.sorted.bam
│   ├── SRR18273871.markdup.sorted.bam.bai
│   ├── SRR18273872
│   ├── SRR18273872.markdup.sorted.bam
│   ├── SRR18273872.markdup.sorted.bam.bai
│   ├── SRR18273873
│   ├── SRR18273873.markdup.sorted.bam
│   ├── SRR18273873.markdup.sorted.bam.bai
│   ├── stringtie
│   └── tx2gene.tsv
└── trimgalore
    ├── SRR18273868_trimmed_1.fastq.gz_trimming_report.txt
    ├── SRR18273868_trimmed_2.fastq.gz_trimming_report.txt
    ├── SRR18273869_trimmed_1.fastq.gz_trimming_report.txt
    ├── SRR18273869_trimmed_2.fastq.gz_trimming_report.txt
    ├── SRR18273870_trimmed_1.fastq.gz_trimming_report.txt
    ├── SRR18273870_trimmed_2.fastq.gz_trimming_report.txt
    ├── SRR18273871_trimmed_1.fastq.gz_trimming_report.txt
    ├── SRR18273871_trimmed_2.fastq.gz_trimming_report.txt
    ├── SRR18273872_trimmed_1.fastq.gz_trimming_report.txt
    ├── SRR18273872_trimmed_2.fastq.gz_trimming_report.txt
    ├── SRR18273873_trimmed_1.fastq.gz_trimming_report.txt
    └── SRR18273873_trimmed_2.fastq.gz_trimming_report.txt

24 directories, 57 files
```


## 4. nf-core/differentialabundanceの実行
### 4.1 解析に必要なデータの準備
`nf-core/differentialabundance`のためのsamplesheet.csvにはサンプル名、fastqファイルとグループ名を記述します。
```
sample,fastq_1,fastq_2,group
SRR18273868,/home/hiuchi/test/fastq/SRR18273868_1.fastq.gz,/home/hiuchi/test/fastq/SRR18273868_2.fastq.gz,A
SRR18273869,/home/hiuchi/test/fastq/SRR18273869_1.fastq.gz,/home/hiuchi/test/fastq/SRR18273869_2.fastq.gz,A
SRR18273870,/home/hiuchi/test/fastq/SRR18273870_1.fastq.gz,/home/hiuchi/test/fastq/SRR18273870_2.fastq.gz,A
SRR18273871,/home/hiuchi/test/fastq/SRR18273871_1.fastq.gz,/home/hiuchi/test/fastq/SRR18273871_2.fastq.gz,B
SRR18273872,/home/hiuchi/test/fastq/SRR18273872_1.fastq.gz,/home/hiuchi/test/fastq/SRR18273872_2.fastq.gz,B
SRR18273873,/home/hiuchi/test/fastq/SRR18273873_1.fastq.gz,/home/hiuchi/test/fastq/SRR18273873_2.fastq.gz,B
```

contrast.csvには比較のレイアウトを記述します。
```
id,variable,reference,target
DEG_analysis,group,A,B
```

### 4.2 解析の実行
パイプラインのバージョン1.5.0で解析を実行します。
```
nextflow run nf-core/differentialabundance \
    -r 1.5.0 \
    -profile rnaseq,docker \
    --input samplesheet_differentialabundance.csv \
    --contrasts contrast.csv \
    --matrix ./quant/star_salmon/salmon.merged.gene_counts_length_scaled.tsv \
    --gtf /path/to/gencode.v46.basic.annotation.gtf \
    --outdir DE
```

### 4.3 実行結果の確認
出力されたファイルを確認します。
```
tree -L 2
.
├── other
│   └── deseq2
├── pipeline_info
│   ├── collated_versions.yml
│   ├── execution_report_2024-12-12_20-56-57.html
│   ├── execution_timeline_2024-12-12_20-56-57.html
│   ├── execution_trace_2024-12-12_20-54-01.txt
│   ├── execution_trace_2024-12-12_20-56-57.txt
│   ├── params_2024-12-12_20-57-01.json
│   └── pipeline_dag_2024-12-12_20-56-57.html
├── plots
│   ├── differential
│   ├── exploratory
│   └── qc
├── report
│   ├── study.html
│   └── study.zip
├── shinyngs_app
│   ├── study
│   └── versions.yml
└── tables
    ├── annotation
    ├── differential
    └── processed_abundance

14 directories, 10 files
```
