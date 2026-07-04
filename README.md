# Doshisha Logo Detection

画像の中から同志社大学のロゴ（三角形のシンボルマーク）を検出するプロジェクト。

R. Girshick et al., *"Rich feature hierarchies for accurate object detection and semantic segmentation"* (CVPR 2014) のR-CNNで使われている「候補領域を抽出してからCNNで分類する」という2段階のアプローチをベースに実装している。

## 仕組み

大きく分けて2つのステップで構成される。

1. **Selective Search** で画像から「物体がありそうな領域」を大量に抽出する
2. 切り出した各領域を、事前学習済みCNN（ResNet18）をファインチューニングした二値分類器にかけて「同志社ロゴ」か「その他」かを判定する

Selective Searchは色や質感の近さを手がかりに領域を階層的にグルーピングしていく手法で、画像全体をいきなりCNNに通すのではなく、まず候補領域を絞り込んでから分類する構成になっている。

ロゴ自体は形がシンプルなので、学習データも

- positive画像（ロゴを切り出した画像）: 数十枚程度
- negative画像（ロゴを含まない適当な領域）: ランダムサンプリングで用意

程度の少量で構成している。定量的な精度評価は行わず、テスト画像上で検出結果を目視確認する運用としている。

## 使っている技術

- Python / Google Colab
- PyTorch, torchvision（ResNet18の転移学習）
- OpenCV (`opencv-contrib-python`) の `cv2.ximgproc.segmentation`（Selective Search）
- Non-Maximum Suppression（`cv2.dnn.NMSBoxes`）で重複する検出枠を整理

## ノートブックの流れ

[`doshisha_logo_detection.ipynb`](doshisha_logo_detection.ipynb) の構成は以下の通り。

1. はじめに
2. Google Driveのマウントとフォルダ準備
3. ライブラリの準備
4. テスト画像の確認
5. Selective Searchで候補領域を抽出
6. 学習データ（Positive / Negative）の確認
7. CNNモデルの準備（ResNet18の転移学習）
8. CNNの学習
9. テスト画像でロゴ検出
10. 結果・感想

## 使い方

1. Google Driveに以下のフォルダ構成で画像を用意する（ノートブック内で自動作成もされます）
   ```
   DoshishaLogoDetection/
     data/
       positive/   # 同志社ロゴを切り出した画像
       negative/   # ロゴを含まない適当な画像
       test/
         test.jpg  # 検出を試したいテスト画像
   ```
2. Google Colabで `doshisha_logo_detection.ipynb` を開き、上から順にセルを実行する
3. 最後のセルで、テスト画像上に検出結果（矩形とスコア）が描画される

学習用の画像自体はこのリポジトリには含まれていない。

## 主なハイパーパラメータ

| 項目 | 値 |
|---|---|
| CNN | ResNet18（ImageNet事前学習済み、`fc`層のみ2クラス分類用に付け替え） |
| 入力サイズ | 224×224 |
| エポック数 | 5 |
| バッチサイズ | 16 |
| 最適化手法 | Adam (lr=0.001) |
| 損失関数 | CrossEntropyLoss |
| 検出しきい値 | 0.80 |
| NMS score_threshold | 0.50 |
| NMS nms_threshold | 0.30 |

## 既知の制約

- 「Doshisha」の文字部分など、ロゴ以外の領域も検出されることがある
- 学習データ数が少ないため、学習序盤で検証Lossが大きく振れるなど不安定になりやすい
- 精度向上にはpositive/negative画像の追加、エポック数・学習率の調整が必要

## 参考

- R. Girshick, J. Donahue, T. Darrell, J. Malik, "Rich feature hierarchies for accurate object detection and semantic segmentation", CVPR 2014.
