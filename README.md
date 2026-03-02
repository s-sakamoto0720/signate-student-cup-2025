<<<<<<< HEAD
# SIGNATE × TECH OCEAN Student Cup 2025

[![Signate](https://img.shields.io/badge/Signate-Competition-00A4E4)](https://user.competition.signate.jp/ja/competition/detail/?competition=385dcbba17b645f3ac10f827dfba03f6)

## Overview

| 項目 | 内容 |
|------|------|
| プラットフォーム | Signate (Student Cup 2025) |
| タスク | テキスト分類 (NLP) |
| 評価指標 | Accuracy（両方のペアが合って正解） |
| 参加期間 | 2026/01/14 - 2026/02/12 |
| 最終順位 | 36 / 245 (Top 15%) |
| チーム / ソロ | Solo |

---

## Problem

LLMによって生成された架空のストーリーを解析し、その元となる2つの映画タイトルを予測するテキスト分類タスク。

各テストデータは2つの映画のあらすじを混ぜ合わせた「合成ストーリー」であり、全50作品の候補（base_stories）の中から元の2作品のペアを特定する必要がある。

---

## Approach

### 類似度計算（埋め込みモデルのアンサンブル）

TF-IDF と Hugging Face 上の埋め込みモデルを複数試し、検証データに対する**正解ペアの平均順位**が特に低い（=良い）3モデルを選定した。

| モデル | 種別 |
|--------|------|
| [Qwen3-Embedding-0.6B](https://huggingface.co/Qwen/Qwen3-Embedding-0.6B) | SentenceTransformer |
| [ruri-v3-310m](https://huggingface.co/cl-nagoya/ruri-v3-310m) | SentenceTransformer |
| [stsb-xlm-r-multilingual](https://huggingface.co/sentence-transformers/stsb-xlm-r-multilingual) | SentenceTransformer |
| TF-IDF（Sudachipy形態素解析） | 古典的手法 |

各モデルの類似度行列を MinMax スケーリングで正規化した後、グリッドサーチで検証データの平均順位が最小になる重みを探索した。最終的な重みは以下の通り：

| モデル | 重み |
|--------|------|
| Qwen3-Embedding-0.6B | 0.2 |
| ruri-v3-310m | 0.2 |
| stsb-xlm-r-multilingual | 0.1 |
| TF-IDF | 0.5 |

### LLM 推論（Gemini）

- モデル: **gemini-3.0-flash**
- 類似度上位**20件**を候補としてプロンプトに含める
- 1プロンプトにつき1問を対処
- プロンプトは比較的シンプルに設計

### その他試したこと

- **類似度1位を確定させる戦略**: 1位と2位の類似度の差が相対的に大きいケースについて、1位を確定とし残り1つだけを推論させた → スコアは改善せず
- **候補数の変化**: 15件 vs 20件で比較し、20件の方がスコアが高かった。それ以上はAPI料金がかさむため今回は20件に限定
- **Step by Stepプロンプト**: 思考過程を明示的に指示するプロンプトを試した → うまくいかず

---

## What I Learned

### シンプルなプロンプトの方が精度が高かった

同じモデル（gemini-3.0-flash）でも、プロンプトの設計次第で精度が大きく変わることを実感した。Step by Step を指示したプロンプトより、シンプルなプロンプトの方がスコアが高かった。これは推論モデルが内部で十分な思考を行えるため、ペルソナや思考過程を詳細に指定するとかえって推論の幅が制限されて精度が下がるのだと考えられる。

### 埋め込みモデルのアンサンブルは多様性が重要

Kaggle の heart-disease コンペでも学んだことだが、今回もアンサンブルの候補モデルを「単体の性能が高い順」で選んでしまった。単体性能ではなく、もっと多くの埋め込みモデルを候補にして Optuna 等で探索した方が、類似度計算の精度が上がった可能性がある。ただし検証データが20件しかないため、過学習のリスクもあり参考程度にしかならなそうだが、、

### リランカーの存在

コンペ終了後にリランカー（Reranker）という手法を知った。候補を埋め込みで絞り込んだ後、cross-encoder 等で再ランク付けすることで精度を向上させるアプローチで、今後の検索・マッチングタスクで試してみたい。

---

## Setup

Gemini API キーは `.env` ファイルに設定して使用します。

```bash
GEMINI_API_KEY=your_api_key
```

---

## Notebooks

| No. | ファイル | 内容 |
|-----|---------|------|
| 01 | [01_embedding_weight_optimization.ipynb](notebook/01_embedding_weight_optimization.ipynb) | 埋め込みモデルのアンサンブル重み最適化（グリッドサーチ） |
| 02 | [02_gemini_inference.ipynb](notebook/02_gemini_inference.ipynb) | Gemini による推論・精度検証・提出ファイル生成 |
=======
# signate-student-cup-2025
>>>>>>> 128b599579062b31bf373c177ab82c1c88b08e51
