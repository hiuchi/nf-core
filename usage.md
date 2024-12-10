# nf-core RNA-Seq and DEG Analysis Tutorial

## 目次

### 1. セットアップ
- Nextflowのインストール
- `nf-core/rnaseq`と`nf-core/differentialabundance`のセットアップ

### 2. 解析するデータと必要なファイルの確認
- 解析するデータ
- 生物学的条件のアノテーション
- `differentialabundance`用の入力データ作成

### 3. nf-core/rnaseqの実行
- 解析に必要なデータの準備
- 解析
- 実行結果の確認

### 4. nf-core/differentialabundanceの実行
- 解析
- 実行結果の確認

---
## 1. Nextflowのインストール、nf-core/rnaseqとnf-core/differentialabundanceのセットアップ
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

### 1.2 `nf-core/rnaseq`と`nf-core/differentialabundance`のセットアップ
#### nf-coreパイプラインのPull方法

nf-coreのパイプラインは、```nextflow pull```を使用してローカル環境にPull（取得）できます。以下のコマンドを実行すると、`rnaseq`パイプラインをローカルにPullします。

```bash
# nf-core/rnaseqとnf-core/differentialabundanceパイプラインをPullする
nextflow pull nf-core/rnaseq
nextflow pull nf-core/differentialabundance
```
