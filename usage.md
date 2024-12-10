# nf-core RNA-Seq and DEG Analysis Tutorial

## 目次

### 1. nf-core/rnaseqのセットアップと実行
- nf-coreとNextflowのインストール
- `rnaseq`パイプラインの基本構造
- コンフィグファイルの作成（Docker/Singularity環境対応）
- RNA-Seq解析の実行と結果の確認

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
## 1. nf-core/rnaseqのセットアップと実行
### 1.1 nf-coreとNextflowのインストール

まずは以下の手順でNextflowをインストールします。

```bash
# Java（必要な場合）のインストール
sudo apt-get update && sudo apt-get install -y openjdk-11-jdk

# Nextflowのインストール
curl -s https://get.nextflow.io | bash

# 実行可能にするため、PATHに追加
mv nextflow ~/bin/  # 必要に応じて適切なパスに移動
export PATH=$PATH:~/bin/
