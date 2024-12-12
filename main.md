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
NCBIから上記のデータをダウンロードします。作業環境により10分から数十分程度かかることがあります。

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

```
hiuchi@hpv:~/test/quant$ tree
.
├── fastqc
│   ├── raw
│   │   ├── SRR18273868_raw_1_fastqc.html
│   │   ├── SRR18273868_raw_1_fastqc.zip
│   │   ├── SRR18273868_raw_2_fastqc.html
│   │   ├── SRR18273868_raw_2_fastqc.zip
│   │   ├── SRR18273869_raw_1_fastqc.html
│   │   ├── SRR18273869_raw_1_fastqc.zip
│   │   ├── SRR18273869_raw_2_fastqc.html
│   │   ├── SRR18273869_raw_2_fastqc.zip
│   │   ├── SRR18273870_raw_1_fastqc.html
│   │   ├── SRR18273870_raw_1_fastqc.zip
│   │   ├── SRR18273870_raw_2_fastqc.html
│   │   ├── SRR18273870_raw_2_fastqc.zip
│   │   ├── SRR18273871_raw_1_fastqc.html
│   │   ├── SRR18273871_raw_1_fastqc.zip
│   │   ├── SRR18273871_raw_2_fastqc.html
│   │   ├── SRR18273871_raw_2_fastqc.zip
│   │   ├── SRR18273872_raw_1_fastqc.html
│   │   ├── SRR18273872_raw_1_fastqc.zip
│   │   ├── SRR18273872_raw_2_fastqc.html
│   │   ├── SRR18273872_raw_2_fastqc.zip
│   │   ├── SRR18273873_raw_1_fastqc.html
│   │   ├── SRR18273873_raw_1_fastqc.zip
│   │   ├── SRR18273873_raw_2_fastqc.html
│   │   └── SRR18273873_raw_2_fastqc.zip
│   └── trim
│       ├── SRR18273868_trimmed_1_val_1_fastqc.html
│       ├── SRR18273868_trimmed_1_val_1_fastqc.zip
│       ├── SRR18273868_trimmed_2_val_2_fastqc.html
│       ├── SRR18273868_trimmed_2_val_2_fastqc.zip
│       ├── SRR18273869_trimmed_1_val_1_fastqc.html
│       ├── SRR18273869_trimmed_1_val_1_fastqc.zip
│       ├── SRR18273869_trimmed_2_val_2_fastqc.html
│       ├── SRR18273869_trimmed_2_val_2_fastqc.zip
│       ├── SRR18273870_trimmed_1_val_1_fastqc.html
│       ├── SRR18273870_trimmed_1_val_1_fastqc.zip
│       ├── SRR18273870_trimmed_2_val_2_fastqc.html
│       ├── SRR18273870_trimmed_2_val_2_fastqc.zip
│       ├── SRR18273871_trimmed_1_val_1_fastqc.html
│       ├── SRR18273871_trimmed_1_val_1_fastqc.zip
│       ├── SRR18273871_trimmed_2_val_2_fastqc.html
│       ├── SRR18273871_trimmed_2_val_2_fastqc.zip
│       ├── SRR18273872_trimmed_1_val_1_fastqc.html
│       ├── SRR18273872_trimmed_1_val_1_fastqc.zip
│       ├── SRR18273872_trimmed_2_val_2_fastqc.html
│       ├── SRR18273872_trimmed_2_val_2_fastqc.zip
│       ├── SRR18273873_trimmed_1_val_1_fastqc.html
│       ├── SRR18273873_trimmed_1_val_1_fastqc.zip
│       ├── SRR18273873_trimmed_2_val_2_fastqc.html
│       └── SRR18273873_trimmed_2_val_2_fastqc.zip
├── multiqc
│   └── star_salmon
│       ├── multiqc_report_data
│       │   ├── cutadapt_filtered_reads_plot.txt
│       │   ├── cutadapt_trimmed_sequences_plot_3_Counts.txt
│       │   ├── cutadapt_trimmed_sequences_plot_3_Obs_Exp.txt
│       │   ├── fastqc_raw_adapter_content_plot.txt
│       │   ├── fastqc_raw_per_base_n_content_plot.txt
│       │   ├── fastqc_raw_per_base_sequence_quality_plot.txt
│       │   ├── fastqc_raw_per_sequence_gc_content_plot_Counts.txt
│       │   ├── fastqc_raw_per_sequence_gc_content_plot_Percentages.txt
│       │   ├── fastqc_raw_per_sequence_quality_scores_plot.txt
│       │   ├── fastqc_raw_sequence_counts_plot.txt
│       │   ├── fastqc_raw_sequence_duplication_levels_plot.txt
│       │   ├── fastqc_raw-status-check-heatmap.txt
│       │   ├── fastqc_raw_top_overrepresented_sequences_table.txt
│       │   ├── fastqc_sequence_length_distribution_plot.txt
│       │   ├── fastqc_trimmed_adapter_content_plot.txt
│       │   ├── fastqc_trimmed_per_base_n_content_plot.txt
│       │   ├── fastqc_trimmed_per_base_sequence_quality_plot.txt
│       │   ├── fastqc_trimmed_per_sequence_gc_content_plot_Counts.txt
│       │   ├── fastqc_trimmed_per_sequence_gc_content_plot_Percentages.txt
│       │   ├── fastqc_trimmed_per_sequence_quality_scores_plot.txt
│       │   ├── fastqc_trimmed_sequence_counts_plot.txt
│       │   ├── fastqc_trimmed_sequence_duplication_levels_plot.txt
│       │   ├── fastqc_trimmed-status-check-heatmap.txt
│       │   ├── fastqc_trimmed_top_overrepresented_sequences_table.txt
│       │   ├── junction_saturation_known.txt
│       │   ├── junction_saturation_novel.txt
│       │   ├── multiqc_citations.txt
│       │   ├── multiqc_cutadapt.txt
│       │   ├── multiqc_data.json
│       │   ├── multiqc_fail_strand_check_table.txt
│       │   ├── multiqc_fastqc_fastqc_raw.txt
│       │   ├── multiqc_fastqc_fastqc_trimmed.txt
│       │   ├── multiqc_general_stats.txt
│       │   ├── multiqc.log
│       │   ├── multiqc_picard_dups.txt
│       │   ├── multiqc_rseqc_bam_stat.txt
│       │   ├── multiqc_rseqc_infer_experiment.txt
│       │   ├── multiqc_rseqc_junction_annotation.txt
│       │   ├── multiqc_rseqc_read_distribution.txt
│       │   ├── multiqc_samtools_flagstat.txt
│       │   ├── multiqc_samtools_idxstats.txt
│       │   ├── multiqc_samtools_stats.txt
│       │   ├── multiqc_software_versions.txt
│       │   ├── multiqc_sources.txt
│       │   ├── multiqc_star.txt
│       │   ├── picard_deduplication.txt
│       │   ├── picard_histogram_1.txt
│       │   ├── picard_histogram_2.txt
│       │   ├── picard_histogram.txt
│       │   ├── rseqc_bam_stat.txt
│       │   ├── rseqc_infer_experiment_plot.txt
│       │   ├── rseqc_inner_distance_plot_Counts.txt
│       │   ├── rseqc_inner_distance_plot_Percentages.txt
│       │   ├── rseqc_inner_distance.txt
│       │   ├── rseqc_junction_annotation_junctions_plot_Events.txt
│       │   ├── rseqc_junction_annotation_junctions_plot_Junctions.txt
│       │   ├── rseqc_junction_saturation_all.txt
│       │   ├── rseqc_junction_saturation_plot_All_Junctions.txt
│       │   ├── rseqc_junction_saturation_plot_Known_Junctions.txt
│       │   ├── rseqc_junction_saturation_plot_Novel_Junctions.txt
│       │   ├── rseqc_read_distribution_plot.txt
│       │   ├── rseqc_read_dups_plot.txt
│       │   ├── rseqc_read_dups.txt
│       │   ├── samtools_alignment_plot.txt
│       │   ├── samtools-flagstat-dp_Percentage_of_total.txt
│       │   ├── samtools-flagstat-dp_Read_counts.txt
│       │   ├── samtools-idxstats-mapped-reads-plot_Normalised_Counts.txt
│       │   ├── samtools-idxstats-mapped-reads-plot_Observed_over_Expected_Counts.txt
│       │   ├── samtools-idxstats-mapped-reads-plot_Raw_Counts.txt
│       │   ├── samtools-idxstats-xy-plot.txt
│       │   ├── samtools-stats-dp.txt
│       │   ├── star_alignment_plot.txt
│       │   └── star_summary_table.txt
│       ├── multiqc_report.html
│       └── multiqc_report_plots
│           ├── pdf
│           │   ├── cutadapt_filtered_reads_plot-cnt.pdf
│           │   ├── cutadapt_filtered_reads_plot-pct.pdf
│           │   ├── cutadapt_trimmed_sequences_plot_3_Counts.pdf
│           │   ├── cutadapt_trimmed_sequences_plot_3_Obs_Exp.pdf
│           │   ├── fail_strand_check_table.pdf
│           │   ├── fastqc_raw_adapter_content_plot.pdf
│           │   ├── fastqc_raw_per_base_n_content_plot.pdf
│           │   ├── fastqc_raw_per_base_sequence_quality_plot.pdf
│           │   ├── fastqc_raw_per_sequence_gc_content_plot_Counts.pdf
│           │   ├── fastqc_raw_per_sequence_gc_content_plot_Percentages.pdf
│           │   ├── fastqc_raw_per_sequence_quality_scores_plot.pdf
│           │   ├── fastqc_raw_sequence_counts_plot-cnt.pdf
│           │   ├── fastqc_raw_sequence_counts_plot-pct.pdf
│           │   ├── fastqc_raw_sequence_duplication_levels_plot.pdf
│           │   ├── fastqc_raw-status-check-heatmap.pdf
│           │   ├── fastqc_raw_top_overrepresented_sequences_table.pdf
│           │   ├── fastqc_sequence_length_distribution_plot.pdf
│           │   ├── fastqc_trimmed_adapter_content_plot.pdf
│           │   ├── fastqc_trimmed_per_base_n_content_plot.pdf
│           │   ├── fastqc_trimmed_per_base_sequence_quality_plot.pdf
│           │   ├── fastqc_trimmed_per_sequence_gc_content_plot_Counts.pdf
│           │   ├── fastqc_trimmed_per_sequence_gc_content_plot_Percentages.pdf
│           │   ├── fastqc_trimmed_per_sequence_quality_scores_plot.pdf
│           │   ├── fastqc_trimmed_sequence_counts_plot-cnt.pdf
│           │   ├── fastqc_trimmed_sequence_counts_plot-pct.pdf
│           │   ├── fastqc_trimmed_sequence_duplication_levels_plot.pdf
│           │   ├── fastqc_trimmed-status-check-heatmap.pdf
│           │   ├── fastqc_trimmed_top_overrepresented_sequences_table.pdf
│           │   ├── general_stats_table.pdf
│           │   ├── picard_deduplication-cnt.pdf
│           │   ├── picard_deduplication-pct.pdf
│           │   ├── rseqc_bam_stat.pdf
│           │   ├── rseqc_infer_experiment_plot.pdf
│           │   ├── rseqc_inner_distance_plot_Counts.pdf
│           │   ├── rseqc_inner_distance_plot_Percentages.pdf
│           │   ├── rseqc_junction_annotation_junctions_plot_Events-cnt.pdf
│           │   ├── rseqc_junction_annotation_junctions_plot_Events-pct.pdf
│           │   ├── rseqc_junction_annotation_junctions_plot_Junctions-cnt.pdf
│           │   ├── rseqc_junction_annotation_junctions_plot_Junctions-pct.pdf
│           │   ├── rseqc_junction_saturation_plot_All_Junctions.pdf
│           │   ├── rseqc_junction_saturation_plot_Known_Junctions.pdf
│           │   ├── rseqc_junction_saturation_plot_Novel_Junctions.pdf
│           │   ├── rseqc_read_distribution_plot-cnt.pdf
│           │   ├── rseqc_read_distribution_plot-pct.pdf
│           │   ├── rseqc_read_dups_plot.pdf
│           │   ├── samtools_alignment_plot-cnt.pdf
│           │   ├── samtools_alignment_plot-pct.pdf
│           │   ├── samtools-flagstat-dp_Percentage_of_total.pdf
│           │   ├── samtools-flagstat-dp_Read_counts.pdf
│           │   ├── samtools-idxstats-mapped-reads-plot_Normalised_Counts-cnt.pdf
│           │   ├── samtools-idxstats-mapped-reads-plot_Normalised_Counts-log.pdf
│           │   ├── samtools-idxstats-mapped-reads-plot_Observed_over_Expected_Counts-cnt.pdf
│           │   ├── samtools-idxstats-mapped-reads-plot_Observed_over_Expected_Counts-log.pdf
│           │   ├── samtools-idxstats-mapped-reads-plot_Raw_Counts-cnt.pdf
│           │   ├── samtools-idxstats-mapped-reads-plot_Raw_Counts-log.pdf
│           │   ├── samtools-idxstats-xy-plot-cnt.pdf
│           │   ├── samtools-idxstats-xy-plot-pct.pdf
│           │   ├── samtools-stats-dp.pdf
│           │   ├── star_alignment_plot-cnt.pdf
│           │   ├── star_alignment_plot-pct.pdf
│           │   └── star_summary_table.pdf
│           ├── png
│           │   ├── cutadapt_filtered_reads_plot-cnt.png
│           │   ├── cutadapt_filtered_reads_plot-pct.png
│           │   ├── cutadapt_trimmed_sequences_plot_3_Counts.png
│           │   ├── cutadapt_trimmed_sequences_plot_3_Obs_Exp.png
│           │   ├── fail_strand_check_table.png
│           │   ├── fastqc_raw_adapter_content_plot.png
│           │   ├── fastqc_raw_per_base_n_content_plot.png
│           │   ├── fastqc_raw_per_base_sequence_quality_plot.png
│           │   ├── fastqc_raw_per_sequence_gc_content_plot_Counts.png
│           │   ├── fastqc_raw_per_sequence_gc_content_plot_Percentages.png
│           │   ├── fastqc_raw_per_sequence_quality_scores_plot.png
│           │   ├── fastqc_raw_sequence_counts_plot-cnt.png
│           │   ├── fastqc_raw_sequence_counts_plot-pct.png
│           │   ├── fastqc_raw_sequence_duplication_levels_plot.png
│           │   ├── fastqc_raw-status-check-heatmap.png
│           │   ├── fastqc_raw_top_overrepresented_sequences_table.png
│           │   ├── fastqc_sequence_length_distribution_plot.png
│           │   ├── fastqc_trimmed_adapter_content_plot.png
│           │   ├── fastqc_trimmed_per_base_n_content_plot.png
│           │   ├── fastqc_trimmed_per_base_sequence_quality_plot.png
│           │   ├── fastqc_trimmed_per_sequence_gc_content_plot_Counts.png
│           │   ├── fastqc_trimmed_per_sequence_gc_content_plot_Percentages.png
│           │   ├── fastqc_trimmed_per_sequence_quality_scores_plot.png
│           │   ├── fastqc_trimmed_sequence_counts_plot-cnt.png
│           │   ├── fastqc_trimmed_sequence_counts_plot-pct.png
│           │   ├── fastqc_trimmed_sequence_duplication_levels_plot.png
│           │   ├── fastqc_trimmed-status-check-heatmap.png
│           │   ├── fastqc_trimmed_top_overrepresented_sequences_table.png
│           │   ├── general_stats_table.png
│           │   ├── picard_deduplication-cnt.png
│           │   ├── picard_deduplication-pct.png
│           │   ├── rseqc_bam_stat.png
│           │   ├── rseqc_infer_experiment_plot.png
│           │   ├── rseqc_inner_distance_plot_Counts.png
│           │   ├── rseqc_inner_distance_plot_Percentages.png
│           │   ├── rseqc_junction_annotation_junctions_plot_Events-cnt.png
│           │   ├── rseqc_junction_annotation_junctions_plot_Events-pct.png
│           │   ├── rseqc_junction_annotation_junctions_plot_Junctions-cnt.png
│           │   ├── rseqc_junction_annotation_junctions_plot_Junctions-pct.png
│           │   ├── rseqc_junction_saturation_plot_All_Junctions.png
│           │   ├── rseqc_junction_saturation_plot_Known_Junctions.png
│           │   ├── rseqc_junction_saturation_plot_Novel_Junctions.png
│           │   ├── rseqc_read_distribution_plot-cnt.png
│           │   ├── rseqc_read_distribution_plot-pct.png
│           │   ├── rseqc_read_dups_plot.png
│           │   ├── samtools_alignment_plot-cnt.png
│           │   ├── samtools_alignment_plot-pct.png
│           │   ├── samtools-flagstat-dp_Percentage_of_total.png
│           │   ├── samtools-flagstat-dp_Read_counts.png
│           │   ├── samtools-idxstats-mapped-reads-plot_Normalised_Counts-cnt.png
│           │   ├── samtools-idxstats-mapped-reads-plot_Normalised_Counts-log.png
│           │   ├── samtools-idxstats-mapped-reads-plot_Observed_over_Expected_Counts-cnt.png
│           │   ├── samtools-idxstats-mapped-reads-plot_Observed_over_Expected_Counts-log.png
│           │   ├── samtools-idxstats-mapped-reads-plot_Raw_Counts-cnt.png
│           │   ├── samtools-idxstats-mapped-reads-plot_Raw_Counts-log.png
│           │   ├── samtools-idxstats-xy-plot-cnt.png
│           │   ├── samtools-idxstats-xy-plot-pct.png
│           │   ├── samtools-stats-dp.png
│           │   ├── star_alignment_plot-cnt.png
│           │   ├── star_alignment_plot-pct.png
│           │   └── star_summary_table.png
│           └── svg
│               ├── cutadapt_filtered_reads_plot-cnt.svg
│               ├── cutadapt_filtered_reads_plot-pct.svg
│               ├── cutadapt_trimmed_sequences_plot_3_Counts.svg
│               ├── cutadapt_trimmed_sequences_plot_3_Obs_Exp.svg
│               ├── fail_strand_check_table.svg
│               ├── fastqc_raw_adapter_content_plot.svg
│               ├── fastqc_raw_per_base_n_content_plot.svg
│               ├── fastqc_raw_per_base_sequence_quality_plot.svg
│               ├── fastqc_raw_per_sequence_gc_content_plot_Counts.svg
│               ├── fastqc_raw_per_sequence_gc_content_plot_Percentages.svg
│               ├── fastqc_raw_per_sequence_quality_scores_plot.svg
│               ├── fastqc_raw_sequence_counts_plot-cnt.svg
│               ├── fastqc_raw_sequence_counts_plot-pct.svg
│               ├── fastqc_raw_sequence_duplication_levels_plot.svg
│               ├── fastqc_raw-status-check-heatmap.svg
│               ├── fastqc_raw_top_overrepresented_sequences_table.svg
│               ├── fastqc_sequence_length_distribution_plot.svg
│               ├── fastqc_trimmed_adapter_content_plot.svg
│               ├── fastqc_trimmed_per_base_n_content_plot.svg
│               ├── fastqc_trimmed_per_base_sequence_quality_plot.svg
│               ├── fastqc_trimmed_per_sequence_gc_content_plot_Counts.svg
│               ├── fastqc_trimmed_per_sequence_gc_content_plot_Percentages.svg
│               ├── fastqc_trimmed_per_sequence_quality_scores_plot.svg
│               ├── fastqc_trimmed_sequence_counts_plot-cnt.svg
│               ├── fastqc_trimmed_sequence_counts_plot-pct.svg
│               ├── fastqc_trimmed_sequence_duplication_levels_plot.svg
│               ├── fastqc_trimmed-status-check-heatmap.svg
│               ├── fastqc_trimmed_top_overrepresented_sequences_table.svg
│               ├── general_stats_table.svg
│               ├── picard_deduplication-cnt.svg
│               ├── picard_deduplication-pct.svg
│               ├── rseqc_bam_stat.svg
│               ├── rseqc_infer_experiment_plot.svg
│               ├── rseqc_inner_distance_plot_Counts.svg
│               ├── rseqc_inner_distance_plot_Percentages.svg
│               ├── rseqc_junction_annotation_junctions_plot_Events-cnt.svg
│               ├── rseqc_junction_annotation_junctions_plot_Events-pct.svg
│               ├── rseqc_junction_annotation_junctions_plot_Junctions-cnt.svg
│               ├── rseqc_junction_annotation_junctions_plot_Junctions-pct.svg
│               ├── rseqc_junction_saturation_plot_All_Junctions.svg
│               ├── rseqc_junction_saturation_plot_Known_Junctions.svg
│               ├── rseqc_junction_saturation_plot_Novel_Junctions.svg
│               ├── rseqc_read_distribution_plot-cnt.svg
│               ├── rseqc_read_distribution_plot-pct.svg
│               ├── rseqc_read_dups_plot.svg
│               ├── samtools_alignment_plot-cnt.svg
│               ├── samtools_alignment_plot-pct.svg
│               ├── samtools-flagstat-dp_Percentage_of_total.svg
│               ├── samtools-flagstat-dp_Read_counts.svg
│               ├── samtools-idxstats-mapped-reads-plot_Normalised_Counts-cnt.svg
│               ├── samtools-idxstats-mapped-reads-plot_Normalised_Counts-log.svg
│               ├── samtools-idxstats-mapped-reads-plot_Observed_over_Expected_Counts-cnt.svg
│               ├── samtools-idxstats-mapped-reads-plot_Observed_over_Expected_Counts-log.svg
│               ├── samtools-idxstats-mapped-reads-plot_Raw_Counts-cnt.svg
│               ├── samtools-idxstats-mapped-reads-plot_Raw_Counts-log.svg
│               ├── samtools-idxstats-xy-plot-cnt.svg
│               ├── samtools-idxstats-xy-plot-pct.svg
│               ├── samtools-stats-dp.svg
│               ├── star_alignment_plot-cnt.svg
│               ├── star_alignment_plot-pct.svg
│               └── star_summary_table.svg
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
│   ├── nf_core_rnaseq_software_mqc_versions.yml
│   ├── params_2024-12-12_17-53-38.json
│   ├── params_2024-12-12_20-29-25.json
│   ├── params_2024-12-12_20-31-03.json
│   ├── params_2024-12-12_20-40-22.json
│   ├── pipeline_dag_2024-12-12_17-53-26.html
│   ├── pipeline_dag_2024-12-12_20-30-50.html
│   └── pipeline_dag_2024-12-12_20-40-11.html
├── star_salmon
│   ├── bigwig
│   │   ├── SRR18273868.forward.bigWig
│   │   ├── SRR18273868.reverse.bigWig
│   │   ├── SRR18273869.forward.bigWig
│   │   ├── SRR18273869.reverse.bigWig
│   │   ├── SRR18273870.forward.bigWig
│   │   ├── SRR18273870.reverse.bigWig
│   │   ├── SRR18273871.forward.bigWig
│   │   ├── SRR18273871.reverse.bigWig
│   │   ├── SRR18273872.forward.bigWig
│   │   ├── SRR18273872.reverse.bigWig
│   │   ├── SRR18273873.forward.bigWig
│   │   └── SRR18273873.reverse.bigWig
│   ├── deseq2_qc
│   │   ├── deseq2.dds.RData
│   │   ├── deseq2.pca.vals.txt
│   │   ├── deseq2.plots.pdf
│   │   ├── deseq2.sample.dists.txt
│   │   ├── R_sessionInfo.log
│   │   └── size_factors
│   │       ├── deseq2.size_factors.RData
│   │       ├── SRR18273868.txt
│   │       ├── SRR18273869.txt
│   │       ├── SRR18273870.txt
│   │       ├── SRR18273871.txt
│   │       ├── SRR18273872.txt
│   │       └── SRR18273873.txt
│   ├── dupradar
│   │   ├── box_plot
│   │   │   ├── SRR18273869_duprateExpBoxplot.pdf
│   │   │   ├── SRR18273870_duprateExpBoxplot.pdf
│   │   │   ├── SRR18273871_duprateExpBoxplot.pdf
│   │   │   └── SRR18273873_duprateExpBoxplot.pdf
│   │   ├── gene_data
│   │   │   ├── SRR18273869_dupMatrix.txt
│   │   │   ├── SRR18273870_dupMatrix.txt
│   │   │   ├── SRR18273871_dupMatrix.txt
│   │   │   └── SRR18273873_dupMatrix.txt
│   │   ├── histogram
│   │   │   ├── SRR18273869_expressionHist.pdf
│   │   │   ├── SRR18273870_expressionHist.pdf
│   │   │   ├── SRR18273871_expressionHist.pdf
│   │   │   └── SRR18273873_expressionHist.pdf
│   │   ├── intercepts_slope
│   │   │   ├── SRR18273869_intercept_slope.txt
│   │   │   ├── SRR18273870_intercept_slope.txt
│   │   │   ├── SRR18273871_intercept_slope.txt
│   │   │   └── SRR18273873_intercept_slope.txt
│   │   └── scatter_plot
│   │       ├── SRR18273869_duprateExpDens.pdf
│   │       ├── SRR18273870_duprateExpDens.pdf
│   │       ├── SRR18273871_duprateExpDens.pdf
│   │       └── SRR18273873_duprateExpDens.pdf
│   ├── featurecounts
│   │   ├── SRR18273868.biotype_counts_mqc.tsv
│   │   ├── SRR18273868.biotype_counts_rrna_mqc.tsv
│   │   ├── SRR18273868.featureCounts.txt
│   │   ├── SRR18273868.featureCounts.txt.summary
│   │   ├── SRR18273869.biotype_counts_mqc.tsv
│   │   ├── SRR18273869.biotype_counts_rrna_mqc.tsv
│   │   ├── SRR18273869.featureCounts.txt
│   │   ├── SRR18273869.featureCounts.txt.summary
│   │   ├── SRR18273870.biotype_counts_mqc.tsv
│   │   ├── SRR18273870.biotype_counts_rrna_mqc.tsv
│   │   ├── SRR18273870.featureCounts.txt
│   │   ├── SRR18273870.featureCounts.txt.summary
│   │   ├── SRR18273872.biotype_counts_mqc.tsv
│   │   ├── SRR18273872.biotype_counts_rrna_mqc.tsv
│   │   ├── SRR18273872.featureCounts.txt
│   │   ├── SRR18273872.featureCounts.txt.summary
│   │   ├── SRR18273873.biotype_counts_mqc.tsv
│   │   ├── SRR18273873.biotype_counts_rrna_mqc.tsv
│   │   ├── SRR18273873.featureCounts.txt
│   │   └── SRR18273873.featureCounts.txt.summary
│   ├── log
│   │   ├── SRR18273868.Log.final.out
│   │   ├── SRR18273868.Log.out
│   │   ├── SRR18273868.Log.progress.out
│   │   ├── SRR18273868.SJ.out.tab
│   │   ├── SRR18273869.Log.final.out
│   │   ├── SRR18273869.Log.out
│   │   ├── SRR18273869.Log.progress.out
│   │   ├── SRR18273869.SJ.out.tab
│   │   ├── SRR18273870.Log.final.out
│   │   ├── SRR18273870.Log.out
│   │   ├── SRR18273870.Log.progress.out
│   │   ├── SRR18273870.SJ.out.tab
│   │   ├── SRR18273871.Log.final.out
│   │   ├── SRR18273871.Log.out
│   │   ├── SRR18273871.Log.progress.out
│   │   ├── SRR18273871.SJ.out.tab
│   │   ├── SRR18273872.Log.final.out
│   │   ├── SRR18273872.Log.out
│   │   ├── SRR18273872.Log.progress.out
│   │   ├── SRR18273872.SJ.out.tab
│   │   ├── SRR18273873.Log.final.out
│   │   ├── SRR18273873.Log.out
│   │   ├── SRR18273873.Log.progress.out
│   │   └── SRR18273873.SJ.out.tab
│   ├── null.merged.gene_counts_length_scaled.SummarizedExperiment.rds
│   ├── null.merged.gene_counts_scaled.SummarizedExperiment.rds
│   ├── null.merged.gene_counts.SummarizedExperiment.rds
│   ├── null.merged.transcript_counts.SummarizedExperiment.rds
│   ├── picard_metrics
│   │   ├── SRR18273868.markdup.sorted.MarkDuplicates.metrics.txt
│   │   ├── SRR18273869.markdup.sorted.MarkDuplicates.metrics.txt
│   │   ├── SRR18273870.markdup.sorted.MarkDuplicates.metrics.txt
│   │   ├── SRR18273871.markdup.sorted.MarkDuplicates.metrics.txt
│   │   ├── SRR18273872.markdup.sorted.MarkDuplicates.metrics.txt
│   │   └── SRR18273873.markdup.sorted.MarkDuplicates.metrics.txt
│   ├── qualimap
│   │   ├── SRR18273869
│   │   │   ├── css
│   │   │   │   ├── agogo.css
│   │   │   │   ├── ajax-loader.gif
│   │   │   │   ├── basic.css
│   │   │   │   ├── bgfooter.png
│   │   │   │   ├── bgtop.png
│   │   │   │   ├── comment-bright.png
│   │   │   │   ├── comment-close.png
│   │   │   │   ├── comment.png
│   │   │   │   ├── doctools.js
│   │   │   │   ├── down.png
│   │   │   │   ├── down-pressed.png
│   │   │   │   ├── file.png
│   │   │   │   ├── jquery.js
│   │   │   │   ├── minus.png
│   │   │   │   ├── plus.png
│   │   │   │   ├── pygments.css
│   │   │   │   ├── qualimap_logo_small.png
│   │   │   │   ├── report.css
│   │   │   │   ├── searchtools.js
│   │   │   │   ├── underscore.js
│   │   │   │   ├── up.png
│   │   │   │   ├── up-pressed.png
│   │   │   │   └── websupport.js
│   │   │   ├── images_qualimapReport
│   │   │   │   ├── Coverage Profile Along Genes (High).png
│   │   │   │   ├── Coverage Profile Along Genes (Low).png
│   │   │   │   ├── Coverage Profile Along Genes (Total).png
│   │   │   │   ├── Junction Analysis.png
│   │   │   │   ├── Reads Genomic Origin.png
│   │   │   │   └── Transcript coverage histogram.png
│   │   │   ├── qualimapReport.html
│   │   │   ├── raw_data_qualimapReport
│   │   │   │   ├── coverage_profile_along_genes_(high).txt
│   │   │   │   ├── coverage_profile_along_genes_(low).txt
│   │   │   │   └── coverage_profile_along_genes_(total).txt
│   │   │   └── rnaseq_qc_results.txt
│   │   └── SRR18273870
│   │       ├── css
│   │       │   ├── agogo.css
│   │       │   ├── ajax-loader.gif
│   │       │   ├── basic.css
│   │       │   ├── bgfooter.png
│   │       │   ├── bgtop.png
│   │       │   ├── comment-bright.png
│   │       │   ├── comment-close.png
│   │       │   ├── comment.png
│   │       │   ├── doctools.js
│   │       │   ├── down.png
│   │       │   ├── down-pressed.png
│   │       │   ├── file.png
│   │       │   ├── jquery.js
│   │       │   ├── minus.png
│   │       │   ├── plus.png
│   │       │   ├── pygments.css
│   │       │   ├── qualimap_logo_small.png
│   │       │   ├── report.css
│   │       │   ├── searchtools.js
│   │       │   ├── underscore.js
│   │       │   ├── up.png
│   │       │   ├── up-pressed.png
│   │       │   └── websupport.js
│   │       ├── images_qualimapReport
│   │       │   ├── Coverage Profile Along Genes (High).png
│   │       │   ├── Coverage Profile Along Genes (Low).png
│   │       │   ├── Coverage Profile Along Genes (Total).png
│   │       │   ├── Junction Analysis.png
│   │       │   ├── Reads Genomic Origin.png
│   │       │   └── Transcript coverage histogram.png
│   │       ├── qualimapReport.html
│   │       ├── raw_data_qualimapReport
│   │       │   ├── coverage_profile_along_genes_(high).txt
│   │       │   ├── coverage_profile_along_genes_(low).txt
│   │       │   └── coverage_profile_along_genes_(total).txt
│   │       └── rnaseq_qc_results.txt
│   ├── rseqc
│   │   ├── bam_stat
│   │   │   ├── SRR18273868.bam_stat.txt
│   │   │   ├── SRR18273869.bam_stat.txt
│   │   │   ├── SRR18273870.bam_stat.txt
│   │   │   ├── SRR18273871.bam_stat.txt
│   │   │   ├── SRR18273872.bam_stat.txt
│   │   │   └── SRR18273873.bam_stat.txt
│   │   ├── infer_experiment
│   │   │   ├── SRR18273868.infer_experiment.txt
│   │   │   ├── SRR18273869.infer_experiment.txt
│   │   │   ├── SRR18273870.infer_experiment.txt
│   │   │   ├── SRR18273871.infer_experiment.txt
│   │   │   ├── SRR18273872.infer_experiment.txt
│   │   │   └── SRR18273873.infer_experiment.txt
│   │   ├── inner_distance
│   │   │   ├── pdf
│   │   │   │   ├── SRR18273868.inner_distance_plot.pdf
│   │   │   │   ├── SRR18273869.inner_distance_plot.pdf
│   │   │   │   ├── SRR18273870.inner_distance_plot.pdf
│   │   │   │   ├── SRR18273871.inner_distance_plot.pdf
│   │   │   │   ├── SRR18273872.inner_distance_plot.pdf
│   │   │   │   └── SRR18273873.inner_distance_plot.pdf
│   │   │   ├── rscript
│   │   │   │   ├── SRR18273868.inner_distance_plot.r
│   │   │   │   ├── SRR18273869.inner_distance_plot.r
│   │   │   │   ├── SRR18273870.inner_distance_plot.r
│   │   │   │   ├── SRR18273871.inner_distance_plot.r
│   │   │   │   ├── SRR18273872.inner_distance_plot.r
│   │   │   │   └── SRR18273873.inner_distance_plot.r
│   │   │   └── txt
│   │   │       ├── SRR18273868.inner_distance_freq.txt
│   │   │       ├── SRR18273868.inner_distance_mean.txt
│   │   │       ├── SRR18273868.inner_distance.txt
│   │   │       ├── SRR18273869.inner_distance_freq.txt
│   │   │       ├── SRR18273869.inner_distance_mean.txt
│   │   │       ├── SRR18273869.inner_distance.txt
│   │   │       ├── SRR18273870.inner_distance_freq.txt
│   │   │       ├── SRR18273870.inner_distance_mean.txt
│   │   │       ├── SRR18273870.inner_distance.txt
│   │   │       ├── SRR18273871.inner_distance_freq.txt
│   │   │       ├── SRR18273871.inner_distance_mean.txt
│   │   │       ├── SRR18273871.inner_distance.txt
│   │   │       ├── SRR18273872.inner_distance_freq.txt
│   │   │       ├── SRR18273872.inner_distance_mean.txt
│   │   │       ├── SRR18273872.inner_distance.txt
│   │   │       ├── SRR18273873.inner_distance_freq.txt
│   │   │       ├── SRR18273873.inner_distance_mean.txt
│   │   │       └── SRR18273873.inner_distance.txt
│   │   ├── junction_annotation
│   │   │   ├── bed
│   │   │   │   ├── SRR18273868.junction.bed
│   │   │   │   ├── SRR18273868.junction.Interact.bed
│   │   │   │   ├── SRR18273869.junction.bed
│   │   │   │   ├── SRR18273869.junction.Interact.bed
│   │   │   │   ├── SRR18273870.junction.bed
│   │   │   │   ├── SRR18273870.junction.Interact.bed
│   │   │   │   ├── SRR18273871.junction.bed
│   │   │   │   ├── SRR18273871.junction.Interact.bed
│   │   │   │   ├── SRR18273872.junction.bed
│   │   │   │   ├── SRR18273872.junction.Interact.bed
│   │   │   │   ├── SRR18273873.junction.bed
│   │   │   │   └── SRR18273873.junction.Interact.bed
│   │   │   ├── log
│   │   │   │   ├── SRR18273868.junction_annotation.log
│   │   │   │   ├── SRR18273869.junction_annotation.log
│   │   │   │   ├── SRR18273870.junction_annotation.log
│   │   │   │   ├── SRR18273871.junction_annotation.log
│   │   │   │   ├── SRR18273872.junction_annotation.log
│   │   │   │   └── SRR18273873.junction_annotation.log
│   │   │   ├── pdf
│   │   │   │   ├── SRR18273868.splice_events.pdf
│   │   │   │   ├── SRR18273868.splice_junction.pdf
│   │   │   │   ├── SRR18273869.splice_events.pdf
│   │   │   │   ├── SRR18273869.splice_junction.pdf
│   │   │   │   ├── SRR18273870.splice_events.pdf
│   │   │   │   ├── SRR18273870.splice_junction.pdf
│   │   │   │   ├── SRR18273871.splice_events.pdf
│   │   │   │   ├── SRR18273871.splice_junction.pdf
│   │   │   │   ├── SRR18273872.splice_events.pdf
│   │   │   │   ├── SRR18273872.splice_junction.pdf
│   │   │   │   ├── SRR18273873.splice_events.pdf
│   │   │   │   └── SRR18273873.splice_junction.pdf
│   │   │   ├── rscript
│   │   │   │   ├── SRR18273868.junction_plot.r
│   │   │   │   ├── SRR18273869.junction_plot.r
│   │   │   │   ├── SRR18273870.junction_plot.r
│   │   │   │   ├── SRR18273871.junction_plot.r
│   │   │   │   ├── SRR18273872.junction_plot.r
│   │   │   │   └── SRR18273873.junction_plot.r
│   │   │   └── xls
│   │   │       ├── SRR18273868.junction.xls
│   │   │       ├── SRR18273869.junction.xls
│   │   │       ├── SRR18273870.junction.xls
│   │   │       ├── SRR18273871.junction.xls
│   │   │       ├── SRR18273872.junction.xls
│   │   │       └── SRR18273873.junction.xls
│   │   ├── junction_saturation
│   │   │   ├── pdf
│   │   │   │   ├── SRR18273868.junctionSaturation_plot.pdf
│   │   │   │   ├── SRR18273869.junctionSaturation_plot.pdf
│   │   │   │   ├── SRR18273870.junctionSaturation_plot.pdf
│   │   │   │   ├── SRR18273871.junctionSaturation_plot.pdf
│   │   │   │   ├── SRR18273872.junctionSaturation_plot.pdf
│   │   │   │   └── SRR18273873.junctionSaturation_plot.pdf
│   │   │   └── rscript
│   │   │       ├── SRR18273868.junctionSaturation_plot.r
│   │   │       ├── SRR18273869.junctionSaturation_plot.r
│   │   │       ├── SRR18273870.junctionSaturation_plot.r
│   │   │       ├── SRR18273871.junctionSaturation_plot.r
│   │   │       ├── SRR18273872.junctionSaturation_plot.r
│   │   │       └── SRR18273873.junctionSaturation_plot.r
│   │   ├── read_distribution
│   │   │   ├── SRR18273868.read_distribution.txt
│   │   │   ├── SRR18273869.read_distribution.txt
│   │   │   ├── SRR18273870.read_distribution.txt
│   │   │   ├── SRR18273871.read_distribution.txt
│   │   │   ├── SRR18273872.read_distribution.txt
│   │   │   └── SRR18273873.read_distribution.txt
│   │   └── read_duplication
│   │       ├── pdf
│   │       │   ├── SRR18273868.DupRate_plot.pdf
│   │       │   ├── SRR18273869.DupRate_plot.pdf
│   │       │   ├── SRR18273870.DupRate_plot.pdf
│   │       │   ├── SRR18273871.DupRate_plot.pdf
│   │       │   ├── SRR18273872.DupRate_plot.pdf
│   │       │   └── SRR18273873.DupRate_plot.pdf
│   │       ├── rscript
│   │       │   ├── SRR18273868.DupRate_plot.r
│   │       │   ├── SRR18273869.DupRate_plot.r
│   │       │   ├── SRR18273870.DupRate_plot.r
│   │       │   ├── SRR18273871.DupRate_plot.r
│   │       │   ├── SRR18273872.DupRate_plot.r
│   │       │   └── SRR18273873.DupRate_plot.r
│   │       └── xls
│   │           ├── SRR18273868.pos.DupRate.xls
│   │           ├── SRR18273868.seq.DupRate.xls
│   │           ├── SRR18273869.pos.DupRate.xls
│   │           ├── SRR18273869.seq.DupRate.xls
│   │           ├── SRR18273870.pos.DupRate.xls
│   │           ├── SRR18273870.seq.DupRate.xls
│   │           ├── SRR18273871.pos.DupRate.xls
│   │           ├── SRR18273871.seq.DupRate.xls
│   │           ├── SRR18273872.pos.DupRate.xls
│   │           ├── SRR18273872.seq.DupRate.xls
│   │           ├── SRR18273873.pos.DupRate.xls
│   │           └── SRR18273873.seq.DupRate.xls
│   ├── salmon.merged.gene_counts_length_scaled.tsv
│   ├── salmon.merged.gene_counts_scaled.tsv
│   ├── salmon.merged.gene_counts.tsv
│   ├── salmon.merged.gene_lengths.tsv
│   ├── salmon.merged.gene_tpm.tsv
│   ├── salmon.merged.transcript_counts.tsv
│   ├── salmon.merged.transcript_lengths.tsv
│   ├── salmon.merged.transcript_tpm.tsv
│   ├── samtools_stats
│   │   ├── SRR18273868.markdup.sorted.bam.flagstat
│   │   ├── SRR18273868.markdup.sorted.bam.idxstats
│   │   ├── SRR18273868.markdup.sorted.bam.stats
│   │   ├── SRR18273868.sorted.bam.flagstat
│   │   ├── SRR18273868.sorted.bam.idxstats
│   │   ├── SRR18273868.sorted.bam.stats
│   │   ├── SRR18273869.markdup.sorted.bam.flagstat
│   │   ├── SRR18273869.markdup.sorted.bam.idxstats
│   │   ├── SRR18273869.markdup.sorted.bam.stats
│   │   ├── SRR18273869.sorted.bam.flagstat
│   │   ├── SRR18273869.sorted.bam.idxstats
│   │   ├── SRR18273869.sorted.bam.stats
│   │   ├── SRR18273870.markdup.sorted.bam.flagstat
│   │   ├── SRR18273870.markdup.sorted.bam.idxstats
│   │   ├── SRR18273870.markdup.sorted.bam.stats
│   │   ├── SRR18273870.sorted.bam.flagstat
│   │   ├── SRR18273870.sorted.bam.idxstats
│   │   ├── SRR18273870.sorted.bam.stats
│   │   ├── SRR18273871.markdup.sorted.bam.flagstat
│   │   ├── SRR18273871.markdup.sorted.bam.idxstats
│   │   ├── SRR18273871.markdup.sorted.bam.stats
│   │   ├── SRR18273871.sorted.bam.flagstat
│   │   ├── SRR18273871.sorted.bam.idxstats
│   │   ├── SRR18273871.sorted.bam.stats
│   │   ├── SRR18273872.markdup.sorted.bam.flagstat
│   │   ├── SRR18273872.markdup.sorted.bam.idxstats
│   │   ├── SRR18273872.markdup.sorted.bam.stats
│   │   ├── SRR18273872.sorted.bam.flagstat
│   │   ├── SRR18273872.sorted.bam.idxstats
│   │   ├── SRR18273872.sorted.bam.stats
│   │   ├── SRR18273873.markdup.sorted.bam.flagstat
│   │   ├── SRR18273873.markdup.sorted.bam.idxstats
│   │   ├── SRR18273873.markdup.sorted.bam.stats
│   │   ├── SRR18273873.sorted.bam.flagstat
│   │   ├── SRR18273873.sorted.bam.idxstats
│   │   └── SRR18273873.sorted.bam.stats
│   ├── SRR18273868
│   │   ├── aux_info
│   │   │   ├── ambig_info.tsv
│   │   │   ├── expected_bias.gz
│   │   │   ├── fld.gz
│   │   │   ├── meta_info.json
│   │   │   ├── observed_bias_3p.gz
│   │   │   └── observed_bias.gz
│   │   ├── cmd_info.json
│   │   ├── libParams
│   │   │   └── flenDist.txt
│   │   ├── logs
│   │   │   └── salmon_quant.log
│   │   ├── quant.genes.sf
│   │   └── quant.sf
│   ├── SRR18273868.markdup.sorted.bam
│   ├── SRR18273868.markdup.sorted.bam.bai
│   ├── SRR18273869
│   │   ├── aux_info
│   │   │   ├── ambig_info.tsv
│   │   │   ├── expected_bias.gz
│   │   │   ├── fld.gz
│   │   │   ├── meta_info.json
│   │   │   ├── observed_bias_3p.gz
│   │   │   └── observed_bias.gz
│   │   ├── cmd_info.json
│   │   ├── libParams
│   │   │   └── flenDist.txt
│   │   ├── logs
│   │   │   └── salmon_quant.log
│   │   ├── quant.genes.sf
│   │   └── quant.sf
│   ├── SRR18273869.markdup.sorted.bam
│   ├── SRR18273869.markdup.sorted.bam.bai
│   ├── SRR18273870
│   │   ├── aux_info
│   │   │   ├── ambig_info.tsv
│   │   │   ├── expected_bias.gz
│   │   │   ├── fld.gz
│   │   │   ├── meta_info.json
│   │   │   ├── observed_bias_3p.gz
│   │   │   └── observed_bias.gz
│   │   ├── cmd_info.json
│   │   ├── libParams
│   │   │   └── flenDist.txt
│   │   ├── logs
│   │   │   └── salmon_quant.log
│   │   ├── quant.genes.sf
│   │   └── quant.sf
│   ├── SRR18273870.markdup.sorted.bam
│   ├── SRR18273870.markdup.sorted.bam.bai
│   ├── SRR18273871
│   │   ├── aux_info
│   │   │   ├── ambig_info.tsv
│   │   │   ├── expected_bias.gz
│   │   │   ├── fld.gz
│   │   │   ├── meta_info.json
│   │   │   ├── observed_bias_3p.gz
│   │   │   └── observed_bias.gz
│   │   ├── cmd_info.json
│   │   ├── libParams
│   │   │   └── flenDist.txt
│   │   ├── logs
│   │   │   └── salmon_quant.log
│   │   ├── quant.genes.sf
│   │   └── quant.sf
│   ├── SRR18273871.markdup.sorted.bam
│   ├── SRR18273871.markdup.sorted.bam.bai
│   ├── SRR18273872
│   │   ├── aux_info
│   │   │   ├── ambig_info.tsv
│   │   │   ├── expected_bias.gz
│   │   │   ├── fld.gz
│   │   │   ├── meta_info.json
│   │   │   ├── observed_bias_3p.gz
│   │   │   └── observed_bias.gz
│   │   ├── cmd_info.json
│   │   ├── libParams
│   │   │   └── flenDist.txt
│   │   ├── logs
│   │   │   └── salmon_quant.log
│   │   ├── quant.genes.sf
│   │   └── quant.sf
│   ├── SRR18273872.markdup.sorted.bam
│   ├── SRR18273872.markdup.sorted.bam.bai
│   ├── SRR18273873
│   │   ├── aux_info
│   │   │   ├── ambig_info.tsv
│   │   │   ├── expected_bias.gz
│   │   │   ├── fld.gz
│   │   │   ├── meta_info.json
│   │   │   ├── observed_bias_3p.gz
│   │   │   └── observed_bias.gz
│   │   ├── cmd_info.json
│   │   ├── libParams
│   │   │   └── flenDist.txt
│   │   ├── logs
│   │   │   └── salmon_quant.log
│   │   ├── quant.genes.sf
│   │   └── quant.sf
│   ├── SRR18273873.markdup.sorted.bam
│   ├── SRR18273873.markdup.sorted.bam.bai
│   ├── stringtie
│   │   ├── SRR18273868.ballgown
│   │   │   ├── e2t.ctab
│   │   │   ├── e_data.ctab
│   │   │   ├── i2t.ctab
│   │   │   ├── i_data.ctab
│   │   │   └── t_data.ctab
│   │   ├── SRR18273868.coverage.gtf
│   │   ├── SRR18273868.gene.abundance.txt
│   │   ├── SRR18273868.transcripts.gtf
│   │   ├── SRR18273869.ballgown
│   │   │   ├── e2t.ctab
│   │   │   ├── e_data.ctab
│   │   │   ├── i2t.ctab
│   │   │   ├── i_data.ctab
│   │   │   └── t_data.ctab
│   │   ├── SRR18273869.coverage.gtf
│   │   ├── SRR18273869.gene.abundance.txt
│   │   ├── SRR18273869.transcripts.gtf
│   │   ├── SRR18273870.ballgown
│   │   │   ├── e2t.ctab
│   │   │   ├── e_data.ctab
│   │   │   ├── i2t.ctab
│   │   │   ├── i_data.ctab
│   │   │   └── t_data.ctab
│   │   ├── SRR18273870.coverage.gtf
│   │   ├── SRR18273870.gene.abundance.txt
│   │   ├── SRR18273870.transcripts.gtf
│   │   ├── SRR18273871.ballgown
│   │   │   ├── e2t.ctab
│   │   │   ├── e_data.ctab
│   │   │   ├── i2t.ctab
│   │   │   ├── i_data.ctab
│   │   │   └── t_data.ctab
│   │   ├── SRR18273871.coverage.gtf
│   │   ├── SRR18273871.gene.abundance.txt
│   │   ├── SRR18273871.transcripts.gtf
│   │   ├── SRR18273872.ballgown
│   │   │   ├── e2t.ctab
│   │   │   ├── e_data.ctab
│   │   │   ├── i2t.ctab
│   │   │   ├── i_data.ctab
│   │   │   └── t_data.ctab
│   │   ├── SRR18273872.coverage.gtf
│   │   ├── SRR18273872.gene.abundance.txt
│   │   ├── SRR18273872.transcripts.gtf
│   │   ├── SRR18273873.ballgown
│   │   │   ├── e2t.ctab
│   │   │   ├── e_data.ctab
│   │   │   ├── i2t.ctab
│   │   │   ├── i_data.ctab
│   │   │   └── t_data.ctab
│   │   ├── SRR18273873.coverage.gtf
│   │   ├── SRR18273873.gene.abundance.txt
│   │   └── SRR18273873.transcripts.gtf
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
```
nextflow run nf-core/differentialabundance \
    -r 1.5.0 \
    -profile rnaseq,docker \
    --input samplesheet_differentialabundance.csv \
    --contrasts contrast.csv \
    --matrix ./quant/star_rsem/rsem.merged.gene_counts.tsv \
    --gtf /path/to/gencode.v46.basic.annotation.gtf \
    --outdir DE
```

### 4.3 実行結果の確認
```
tree
```
