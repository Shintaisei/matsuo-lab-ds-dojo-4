# matsuo-lab-ds-dojo-4

松尾研究所 DS Dojo 4 の社内データ分析コンペで取り組んだ、Rotten Tomatoes の映画レビュー予測プロジェクトです。

最終順位は **46人中4位** でした。

## 概要

映画・批評家・レビューに関する情報から、レビューが Fresh / Rotten のどちらになるかを予測する二値分類タスクです。Kaggle 環境で作成したベースラインをローカル実験環境へ移行し、特徴量追加、Embedding、協調フィルタリング、GNN、BERT 系モデルなどを比較しました。

最終的には、安定して強かった LightGBM ベースラインを軸に、映画情報・タイトル情報のテキスト Embedding を組み合わせる方針を中心に改善しました。

## Result

| 項目 | 内容 |
| --- | --- |
| コンペ | 松尾研究所 DS Dojo 4 社内コンペ |
| タスク | Rotten Tomatoes Review Prediction |
| 最終順位 | **4位 / 46人** |
| 主なモデル | LightGBM, CatBoost, BERT, GNN |
| 主な特徴量 | 映画メタ情報、批評家特徴、時系列特徴、OpenAI Embedding、次元削減特徴 |
| 最高 Public Score メモ | `0.75449` |

## Approach

主に以下の観点で改善を進めました。

- **ベースライン強化**: 映画・批評家・レビュー日時などの基本特徴量を LightGBM で学習
- **時系列を意識した検証**: レビュー日時に基づく分割でリークを抑えた CV を実施
- **テキスト Embedding**: 映画タイトル・映画概要を OpenAI Embedding でベクトル化し、PCA などで圧縮
- **上位解法の検証**: Target Encoding、NMF/SVD、既知/未知映画向けの分岐モデル、シードアンサンブルを試行
- **追加実験**: GNN、BERT/ModernBERT、協調フィルタリング、2-hop 特徴量などを比較

スコア改善に最も効いたのは、複雑なモデルを単純に足すことではなく、強いベースラインに映画単位のテキスト表現を安定して加えることでした。

## Repository Structure

```text
.
├── train_baseline.ipynb                 # ベースライン学習・CV・提出作成
├── train_7patterns_from_baseline.ipynb  # ベースライン派生の提出実験
├── train_20patterns_from_doc20.ipynb    # 複数パターンの実験
├── train_gnn_bipartite.ipynb            # 二部グラフ GNN 実験
├── run_embedding_experiments.py         # Embedding 実験
├── run_gnn.py                           # GNN 実験
├── run_bert.py                          # BERT 系実験
├── lib/                                 # 学習パイプライン・分析・Embedding・GNN 実装
├── preprocess/                          # データ読み込み・前処理
├── feature_engineering/                 # 特徴量生成
├── scripts/                             # 検証・補助スクリプト
├── docs/                                # 実験ログ・方針・結果メモ
├── archive/                             # 旧実験・再利用用コード
├── config/                              # API キー例など
└── requirements.txt
```

## Setup

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
python -m ipykernel install --user --name ds_dojo4 --display-name "Python 3 (ds_dojo4)"
```

データは `data/` 配下に配置する想定です。コンペデータや生成済みの提出ファイル・Embedding などの大きなファイルは、基本的に Git 管理対象外にしています。

OpenAI Embedding を再生成する場合は、`config/openai_api_key.example.txt` を参考に API キーを設定してください。

## Usage

まずは `train_baseline.ipynb` を開き、カーネルを `Python 3 (ds_dojo4)` に設定して実行します。ベースラインの学習、CV、提出ファイル作成までを確認できます。

Embedding 系の実験は以下のスクリプト・ノートブックから実行できます。

```bash
python run_embedding_experiments.py
python run_quick_embedding_submissions.py
```

GNN / BERT 系の追加実験は以下から実行します。

```bash
python run_gnn.py
python run_bert.py
```

## Docs

実験の詳細は `docs/` にまとめています。

- `docs/PROJECT_MIND.md`: 方針、成功した実験、失敗した実験、次の打ち手
- `docs/05_EMBEDDING_SUBMISSION_RESULTS.md`: Embedding 種別・次元削減ごとの提出結果
- `docs/08_COLLABORATIVE_FILTERING.md`: 協調フィルタリング系の検討
- `docs/20_FEATURES_AND_MODELS_AND_40_SUBMISSIONS.md`: 特徴量・モデル・提出パターンの整理
- `docs/21_TRAIN_20PATTERNS_CODE_CHECK.md`: 20パターン実験のコード確認

## What I Learned

- コンペ終盤では、複雑な手法の追加よりも、CV と Public LB のズレを見ながら「効く特徴量」を絞ることが重要だった
- Embedding は単体で強いというより、既存の表形式特徴量と組み合わせたときに安定して効いた
- GNN や BERT のような重いモデルは、実装・計算コストに対してスコア改善が限定的な場合があり、締切までの時間配分が重要だった
- 実験ログを `docs/` に残したことで、うまくいかなかった手法を再度試すロスを減らせた

## Note

このリポジトリはコンペ参加時の実験コードを含みます。探索的なノートブックや途中実験も残しているため、再現の起点としては `train_baseline.ipynb` と `docs/PROJECT_MIND.md` を参照してください。
