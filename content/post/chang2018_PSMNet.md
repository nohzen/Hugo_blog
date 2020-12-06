---
title: "Pyramid Stereo Matching Network (PSMNet)"
date: 2020-12-06T05:33:22+09:00
lastmod: 2020-12-06T05:33:22+09:00
draft: false
tags: ["Stereo Matching"]

---
[J.-R. Chang, et al. "Pyramid Stereo Matching Network" (2018) CVPR 2018](https://arxiv.org/abs/1803.08669)

# 概要
- **Pyramid Stereo Matching Network** (PSMNet)と名付けた視差推定のネットワークを提案
    - [GCNet]をベースとする
    - 画像のグローバルなコンテキストを活用できる**Spatial Pyramid Pooling Module**を特徴量抽出に利用
    - cost volumeの正則化は、**Hourglass 構造(Encoder-Decoder)の3D convを複数回**使い、**中間出力からも視差を学習**する


# 論文情報
## リンク
- [公式実装 PyTorch](https://github.com/JiaRenChang/PSMNet)

## 著者
台湾のNational Chiao Tung University (NCTU, 国立交通大学)のメンバー
- [Jia-Ren Chang](https://jiarenchang.github.io/)
- [Yong-Sheng Chen](https://people.cs.nctu.edu.tw/~yschen/)


# 内容
## 手法
{{< figure src="/images/chang2018_PSMNet/Fig1.png" caption="[Chang+(2018)] Fig. 1より引用" class="center" width="1000" >}}
- **GCNetをベース**としている。違いは以下の3点:
    - 特徴量抽出に**Spatial Pyramid Pooling** (SPP) moduleを追加することで、グローバルなコンテキストを利用
    - Cost volumeの正規化を行う3D convに**複数回のHourglass構造**を使い、**中間出力からも学習**する(GCNetは1回のHourglass構造)
    - **smooth L1 loss**で回帰問題として扱う(GCNetはL1 loss)

### Spatial Pyramid Pooling (SPP) module
- Textureがない領域や繰り返しパターンの領域では、広い範囲の情報(グローバルなコンテキスト)を利用する必要がある
- そのために、**Spatial pyramid pooling** (SPP)と**Dilated conv**を利用してreceptive fieldを広げた特徴量を作り、それを元にcost volumeを作成する
    - 畳み込み層は、実効的なreceptive filedが理論的な最大範囲ほど広くならないので、Spatial pyramid poolingを利用
- ほとんど**PSPNet**のPyramid Pooling Moduleと同じ(Ave. poolingのサイズだけが違う)
    - 入力特徴量サイズH/4xW/4x128に依らず、出力が64x64, 32x32, 16x16, 8x8のAverage pooling(PSPNetは、1x1, 2x2, 3x3, 6x6のAverage pooling)
    - 各Average poolingの出力に対して、convで特徴量次元を128から32に減らす(本文では1x1 convになっていて、Table 1では3x3 convになっている？PSPNetは1x1 conv)
    - 元解像度(H/4xW/4)にbilinear upscale
    - 入力特徴量と4つのスケールのPooling結果をconcate(conv2の特徴量もconcateする)

### Cost Volume
- GCNetと同様に、左右の画像のSPP moduleの出力特徴量を**concate**することで、4D cost volumeを作成

### Stacked multiple hourglass 3D Conv
cost volumeの正則化として、**basic architecture**と**stacked hourglass architecture**の2種類を提案
- basic architecture: 12個の3x3x3 conv Residual block -> bilinear upsampleで元解像度に
- stacked hourglass(encorder-decoder) architecture: **3個のhourglass構造**で、各hourglassの後でbilinear upsampleした後に視差を計算する。学習時は**中間出力からの視差も学習**し、テスト時は最終出力からの視差のみ使う。GCNetは1個のhourglassに相当(層の数などは違う)。

### 視差出力
- GCNetと同様に、**コストのSoftmaxを確率として加重平均**したものを視差の推論結果とする
    - これにより微分可能（学習可能）で、分類問題として扱うよりロバストになる

### 回帰 Loss
- 物体検出のBounding Boxの回帰に使われる**smooth-L1 loss** (小さいところではL2で、大きいところではL1)を使う
    - L2より外れ値に強くなる
    - GCNetはL1 lossを使っている

## 実験
- Scene Flow: 視差が大きすぎるPixelも含まれるので、一定以上に視差の場合はLossをゼロにする。事前学習なし。
- KITTI2012, 2015: Scene Flowで事前学習 
- 学習時は、画像をランダムに512x256にCrop(spatial pyramid poolingが含まれるので、学習時と推論時でサイズが違うと良くないのでは？)
- ablation study (KITTI2015で)
    - dilation convありの方がいい
    - pyramid poolinありの方がいい
    - basic architectureよりstacked hourglass architectureの方がいい
    - 中間出力のLossの重みは、最終出力に近い方を大きめにする方がいい


# 感想
- 手法自体は、GCNetから順当な小さな改善をした感じ。コードをしっかり公開しているので、引用数が伸びてそう。
- 3D convのところは、basic architectureは中間出力なしで、stacked hourglass architectureだけが中間出力ありなのは理由がある？フェアに比較するなら、揃えた方が良さそう。
    - stacked hourglass architectureは学習しにくいから、中間出力からも学習している？
    - Table2, 3を見ると、中間出力なしのbasic architectureの方が、中間出力なしのstacked hourglass architectureより精度がいい。中間出力ありのbasic architectureは試していないのが気になる。中間出力ありで比較すると、basicの方がstacked hourglassより良くなったりしない？
- Scene Flowも特徴量抽出のところはImageNetで事前学習しておくと、精度が上がったりしない？
- 実験結果で、”by a noteworthy margin”で先行研究よりいいと書かれているが、そこまで大きく改善している訳ではない。Deep learningの論文にありがちなんだけど、無駄な形容詞で結果を良く言うのが嫌い。


# 参考文献
- Stereo Matching
    - [13] [GCNet] A. Kendall, et al. "End-to-end learning of geometry and context for deep stereo regression" (2017)
- Spatial pyramid pooling (SPP)
    - [9] [SPP-net] K. He, et al. "Spatial pyramid pooling in deep convolutional networks for visual recognition" (2014)
    - [16] [ParseNet] W. Liu, et al. "ParseNet: Looking wider to see better" (2015)
    - [32] [PSPNet] H. Zhao, et al. "Pyramid scene parsing network" (2017)
- Dilated conv
    - [2] L.-C. Chen, et al. "DeepLab: Semantic image segmentation with deep convolutional nets, atrous convolution, and fully connected crfs" (2016)
    - [29] F. Yu et al. "Multi-scale context aggregation by dilated convolutions" (2016)


