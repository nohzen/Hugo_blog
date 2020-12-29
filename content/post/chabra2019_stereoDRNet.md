---
title: "StereoDRNet"
date: 2020-12-29T19:22:12+09:00
lastmod: 2020-12-29T19:22:12+09:00
draft: false
tags: ["Stereo Matching"]

mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true
---
[R. Chabra, et al. "Stereodrnet: Dilated residual stereonet" (2019) CVPR2019](https://arxiv.org/abs/1904.02251)

# 論文情報
## リンク
- [3次元再構成の結果動画](https://www.youtube.com/watch?v=AkopUYMIxV4)
- 著者実装（pytorch）の公開はなし

## 著者
第一著者がFacebook Reality Labsでインターン中の仕事
- [Rohan Chabra](http://www.cs.unc.edu/~rohanc/)
- [Julian Straub](http://people.csail.mit.edu/jstraub/) Facebook Reality Labs
- Chris Sweeney, Facebook Reality Labs
- Richard Newcombe, Facebook Reality Labs
- [Henry Fuchs](http://henryfuchs.web.unc.edu/), University of North Carolina


# 概要
- [GCNet]や[PSMNet]をベースとした**StereoDRNet**という視差推定のネットワークを提案
- Refinementのモジュールを追加
    - 入力にphotometric errorだけでなく、geometric errorをRefinementの追加
    - 出力に視差だけでなく、occlutionを追加
- 特徴量抽出部で**Vortex Pooling**を、cost volume finteringで**Atrous Spatial Pyramid Pooling (ASPP) の3D conv版**を利用することで、Globalな情報を活用
    - Textureがない領域や反射がある領域で精度の向上が期待できる


# 内容
- 全体の構造は下図
{{< figure src="/images/chabra2019_stereoDRNet/Fig2.png" caption="[Chabra+(2019)] Fig.2より引用" width="1000" >}}
- 詳細はTable 8を参照

## 特徴量抽出
- 先行研究と同様に、左右の画像で同じ重み
- Globalな情報を得るために、PSMNetではSpatial Pyramid Poolingを使っているが、本研究では**Vortex Pooling**を使う(Fig2のSpatial Poolingの部分)
- 特徴量のサイズは入力解像度のW/4 x H/4で、特徴量次元は32にする

## Cost Volume作成 
- 左右の画像の**特徴量の差**でcost volumeを作る
    - [DispNet]では内積、[GCNet]ではconcateすることでcost volumeを作っている。
    - 精度的にはどれを使ってもあまり変わらない？（実験結果はなし）
- 左をBase画像とした視差と、右をBase画像とした視差を両方同時に学習するので、特徴量の差はBaseを変えて2つ計算してconcateする

## Cost Volume Filtering: Dilated Residual Cost Filtering
- 3D convを使ったFilterの途中で、**Atrous Spatial Pyramid Pooling (ASPP) の3D conv版**みたいなもの（下図のdilationが違う畳み込みが並列している部分）を使う（receptive fieldを広げる目的？）
- Filterは同じような構造のものを3回繰り返し、2回目以降は差分を学習
    - 3回繰り返し、中間出力も学習し、差分を学習するのは[PSMNet]と同じ
{{< figure src="/images/chabra2019_stereoDRNet/Fig4.png" caption="[Chabra+(2019)] Fig.4より引用" width="1000" >}}

## Regression
- [GCNet]と同様にSoft argminを使って視差の推論結果を出力し、LossはHuber loss(smooth-L1的なやつ)
- Cost Volume Filteringの3つの中間出力からLossを計算([PSMNet]と同じ)
- 推論した視差は元解像度のW/4 x H/4なので、Bilinear拡大してからLossを計算する

## Refinement
- ネットワークは[StereoNet]の畳み込みをdilationありにしたもの
- Refinement前の視差はW/4 x H/4のサイズで計算していたが、Refinementの計算は入力解像度W x Hで行う
- 入力
    - 前段で推論した荒い視差（3つのうち、最後の出力を利用）
    - 入力画像
    - **再構成誤差(photometric error)**: 推論した視差で、左右の画像を比較した時のL1誤差
    - **geometric error**: 左右の画像をそれぞれBase画像とした時の2つの視差を比較した時のL1誤差
- photometric errorとgeometric errorはLossとして学習するより、入力とした方が精度が良い（実験結果はなし）
    - これらはocculutionがある時にはゼロにならないので
- Lossは、**occlutionのcross entropyと、視差のHuber loss**
- 詳細はTable 7を参照
{{< figure src="/images/chabra2019_stereoDRNet/Fig5.png" caption="[Chabra+(2019)] Fig.5より引用" width="500" >}}

## 実験
- SceneFlow
    - 左右の画像それぞれをBaseとした時のGTがある
    - occlutionのGTはないので、左右の視差が1pixel以上離れていた時とする（GTのgeometric errorが1以上？）
    - Table 2
        - Spatial pyramid pooling < Vortex Pooling
        - 荒い視差推定で、中間出力ありだと精度向上
        - photometric errorとgeometric errorとocclutionの推定はどれもあったほうがいい
- KITTI2012, KITTI2015
    - SceneFlowで学習済みのモデルをfine-tunning
    - KITTI2015の方ではSOTAではない
    - GTがsparseなので、Refinementは効果なし
- ETH3D
    - SceneFlowで学習済みのモデルをfine-tunning
    - SOTAで、DN-CSSも同程度
- 3D再構成
    - [25]の方法でGTデータ作成
    - [15] Truncated signed distance function (TSDF) でdepthから3D 再構成
    - fine-tunningの時に一部をマスク


# 感想
- photometric error, geometric error
    - 教師なし学習系手法のLossでして使われているのは見るが、入力として使うのは初めて見た
    - Lossとして使うより入力として使った方がいいと書いてあったが、実験結果が無かったので、残念
    - 入力として使う場合、これらは2枚の入力画像と2つの視差マップから計算できるので冗長だが…
- Vortex Poolingで、average poolingのサイズ(=dilateionのサイズ)を原論文の3-9-27から3-5-15に変えている
    - その方が視差推定では精度が良かったらしい
    - $k, k^2, k^3$でないと効率的に計算できないのでは？全体の計算量からすると無視できる？


# 参考文献
- Stereo Matching
    - [2] [PSMNet] J.-R. Chang, et al. “Pyramid Stereo Matching Network” (2018) 
    - [7] [GCNet] A. Kendall et al. “End-to-End Learning of Geometry and Context for Deep Stereo Regression” (2017)
    - [13] [DispNet] N. Mayer et al. “A Large Dataset to Train Convolutional Networks for Disparity, Optical Flow, and Scene Flow Estimation” (2015)
- Globalな情報の活用
    - [Spatial Pyramid Pooling] H. Zhao, et al. “Pyramid Scene Parsing Network” (2016)
    - [3] L.-C. Chen, et al. "Deeplab: Semantic image segmentation with deep convolutional nets, atrous convolution, and fully connected crfs" (2018) Atrous spatial pyramid pooling
    - [26] C.-W. Xie, et al. "Vortex pooling: Improving context representation in semantic segmentation" (2018) Vortex pooling
- 入力画像or特徴量のphotometric errorを入力としたRefinementを使っている研究
    - [17] [CRL] J. Pang, et al. "Cascade residual learning: A two-stage convolutional neural network for stereo matching" (2017)
    - [12] [iResNet] Z. Liang, et al. "Learning for disparity estimation through feature constancy" (2018)
    - [8] [StereoNet] S. Khamis, et al. "Stereonet: Guided hierarchical refinement for real-time edge-aware depth prediction" (2018) この研究はPhotometric errorとか使ってない（紹介論文には使っていると書かれていたが）
    - [5] [FlowNet2] E. Ilg, et al. "Flownet 2.0: Evolution of optical flow estimation with deep networks" (2017)
    - [31] [ActiveStereoNet] Y. Zhang, et al. "Activestereonet: end-to-end self-supervised learning for active stereo systems" (2018)
- 3D再構成
    - [15] R. A. Newcombe, et al. "Kinectfusion: Real-time dense surface mapping and tracking" (2011) Dense 3D reconstruction
    - [25] T. Whelan, et al. "Reconstructing scenes with mirror and glass surfaces" (2018) structure light


