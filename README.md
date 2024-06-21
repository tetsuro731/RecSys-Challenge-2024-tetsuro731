# RecSys-Challenage-2024-tetsuro731

- Recsys Challenge2024のコードをまとめるためのリポジトリ。
コンペ開催中はprivate設定でチームメンバーのみに公開

# 基本方針
- 通常レコメンドにおいてはcandidate generation -> rerankといった2段階の構成を取ることが多い
- 今回のニュースレコメンドについては、あらかじめimpressionごとに表示されたarticleのlistが公開されているので、rerank phaseのみを頑張る
- 有効なfeatureを頑張って生成してLightGBMにぶち込む

# ハイパーパラメータのチューニング
- Optunaにぶち込む

# How to run coes

- `RecSys2024_preprocess.ipynb`を回すことでLightGBMのinputとなるfeatureの下処理をする
  - パラメータをtrain/valid/testにすることでそれぞれの期間におけるデータを生成
  - 公式から準備されているsmall dataでも回せる
  - fullで回すと時間がかかるので、まずはsmallで回して検証し、良さそうならfullで回す
 
-  `preprocess_create_embed.ipynb`
  - featureとして使うembeddingを生成するためのコード
  - 公式から与えられたarticle_idごとのembeddingに加え、yyamaさんに作成してもらった記事のtitle, subtitle, title+subtitleのembeddingを読み込む
  - そのままだと次元が多いので、PCAを使って32次元や16次元などにする
  - 最終的には16次元を採用（次元を増やすほど精度は上がるが計算に時間がかかる）
- `preprocess2_embed_similarity.ipynb`
  - PCAで生成したn次元のembeddingをベースにuserとarticleの類似度を計算する
  - userのベクトルは過去のクリック履歴から、過去n件のベクトルの平均とする
  - 最終subでは過去5件の平均をuser vectorとした
- `RecSyS2024_LGBM_train.ipynb`
  - 一番メインのコード、LightGBMの学習を行う
  - rank学習を行い、metricsはnDCG@kで見る (コンペのmain metricsはAUCだが、やや特殊な計算方法なので)
  - パラメータをtrainにした場合はtrain dataで学習してvalid dataでvalidation
  - パラメータをvalidにした場合はvalid dataで学習してモデル生成 (iteration回数は事前にtrainで学習して決めておく)
  - learning rate=0.05, early_stopping = 20 で基本的に回すが、最終subのfull dataはlearning rate=0.03, early_stopping=40で回した
  - seedは基本42固定だが、最終subのときは52,62とアンサンブル
- `RecSyS2024_LGBM_test.ipynb`
  - 推論用コード、trainで生成したモデルをベースにtest dataを推論する
  - train, validの2種類のモデルでそれぞれ推論結果を得ることができるので、最終的には1:1でアンサンブルした

# Feature Generation


