# NextflowでRNA-seq解析を行う

## 目次

### 1. ソフトウェアのインストール
- 1.1 Nextflowのインストール
- 1.2 `nf-core/rnaseq`と`nf-core/differentialabundance`のpull
- 1.3 SRA-toolkitのインストール

### 2. 解析するデータと必要なファイルの確認
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
## 1. ソフトウェアのインストール
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
nf-coreのパイプラインは、```nextflow pull```を使用してローカル環境にpull（取得）できます。以下のコマンドを実行すると、`rnaseq`パイプラインをローカルにpullします。

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

## 2. 解析するデータと必要なファイルの確認
### 2.1 解析するデータの紹介
今回は新たに設計されたチロシンキナーゼ阻害剤APG-2449をヒト組織に投与し、反応があった群と反応がなかった群の比較を行います。

Fang DD, Tao R, Wang G, Li Y, Zhang K, Xu C, Zhai G, Wang Q, Wang J, Tang C, Min P, Xiong D, Chen J, Wang S, Yang D, Zhai Y. Discovery of a novel ALK/ROS1/FAK inhibitor, APG-2449, in preclinical non-small cell lung cancer and ovarian cancer models. BMC Cancer. 2022 Jul 11;22(1):752. doi: 10.1186/s12885-022-09799-4. PMID: 35820889; PMCID: PMC9277925.

### 2.2 データのダウンロードと変換
NCBIから上記のデータをダウンロードし、fastq形式に変換します。作業環境により10分から数十分程度かかることがあります。

```bash
# データのダウンロード
mkdir fastq
cd fastq
for i in `seq 68 73` ; do prefetch SRR182738$i ; done

# ダウンロードしたデータをfastq形式に変換
for i in `seq 68 73` ; do fasterq-dump -p -e 8 ./SRR182738$i/SRR182738$i.sra ; done
```

## 3. nf-core/rnaseqの実行
### 3.1 解析に必要なデータの準備

### 3.2 解析の実行
### 3.3 実行結果の確認



## 4. nf-core/differentialabundanceの実行
### 4.1 解析に必要なデータの準備
### 4.2 解析の実行
### 4.3 実行結果の確認
