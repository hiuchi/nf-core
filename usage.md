# nf-core RNA-Seq and DEG Analysis Tutorial

## 目次

### 1. セットアップ
- Nextflowのインストール
- `nf-core/rnaseq`と`nf-core/differentialabundance`のセットアップ

### 2. DEG解析の準備
- 解析結果の整理
- 生物学的条件のアノテーション
- `differentialabundance`用の入力データ作成

### 3. nf-core/differentialabundanceのセットアップと実行
- パイプラインの概要
- 実行に必要な設定
- DEG解析の実行と出力ファイルの説明

### 4. 結果の解釈と可視化
- 発現差解析結果の基本的な解釈
- ボルケーノプロット、ヒートマップの作成（例：R/ggplot2使用）
- 上位DEGのアノテーションと機能解析

### 5. 応用編：データの共有とレポート作成
- 結果を再現可能な形で共有（例：GitHub/GitHub Pages）
- レポートの自動生成（Markdownを活用）

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

### `1.2 nf-core/rnaseq`と`nf-core/differentialabundance`のセットアップ
## nf-coreパイプラインのPull方法

nf-coreのパイプラインは、```nextflow pull```を使用してローカル環境にPull（取得）できます。以下は`rnaseq`と`differentialabundance`の取得手順です。

### nf-core/rnaseqをPullする

以下のコマンドを実行すると、`rnaseq`パイプラインをローカルにPullします。

```bash
# nf-core/rnaseqとnf-core/differentialabundanceパイプラインをPullする
nextflow pull nf-core/rnaseq
nextflow pull nf-core/differentialabundance
```
