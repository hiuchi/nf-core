# nf-core RNA-Seq and DEG Analysis Tutorial

## 目次

### 1. ソフトウェアのインストール
- 1.1 Nextflowのインストール
- 1.2 `nf-core/rnaseq`と`nf-core/differentialabundance`のpull
- 1.3 SRA-toolkitのインストール

### 2. 解析するデータと必要なファイルの確認
- 2.1 解析するデータの紹介
- 2.2 データのダウンロード

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
# nf-core/rnaseqとnf-core/differentialabundanceパイプラインをPullする
nextflow pull nf-core/rnaseq
nextflow pull nf-core/differentialabundance
```

### 1.3 SRA-Toolkitのインストール
公共データをダウンロードするために必要なSRA-Toolkitをインストールします。
```bash
conda install -c bioconda sra-tools
```
