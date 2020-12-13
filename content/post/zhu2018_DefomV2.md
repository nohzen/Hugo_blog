---
title: "Deformable ConvNets v2"
date: 2020-12-14T00:55:04+09:00
lastmod: 2020-12-14T00:55:04+09:00
draft: false
tags: ["Object Detection", "Instance Segmentation"]

---
[X. Zhu *et al*. "Deformable ConvNets v2: More Deformable, Better Results" (2018) CVPR2019](https://arxiv.org/abs/1811.11168) 


# 概要
- **Deformable ConvNets v1**は普通のConvNetsより改善はしているが、特徴量に寄与している領域を可視化すると改善の余地があることが分かった
- Deformable ConvNets v1を3つの点で改善した**Deformable ConvNets v2**を提案した
    1. Deformable Conv層の数を増やす
    2. 畳み込みのサンプリング位置を可変・学習可能するだめでなく、**絶対値(modulation)も可変・学習可能に**する
    3. 適切な場所から特徴量を学習できるように、**R-CNNの特徴量と一致させるようなLossを加えて学習**する(蒸留的な方法)
- Deformable ConvNets v2を、Object DetectionとInstance Segmentationに適用


# 論文情報
## リンク
- [公式実装](https://github.com/msracver/Deformable-ConvNets/tree/master/DCNv2_op)
- 野良実装
    - https://github.com/chengdazhi/Deformable-Convolution-V2-PyTorch
    - https://github.com/4uiiurz1/pytorch-deform-conv-v2

## 著者
Microsoft Research Asia (MSRA)のメンバー。第1著者がインターン中の仕事。第2著者, ラストオーサーはV1と同じ人。
- Xizhou Zhu
- [Han Hu](https://ancientmooner.github.io/)
- Stephen Lin
- [Jifeng Dai](https://jifengdai.org/) 現SenseTime Research。R-FCNなど。


# 内容・感想
日本語の記事が既にあったので、主に感想のみ。
- 解説記事
    - http://peluigi.hatenablog.com/entry/2018/11/30/150502
    - https://uiiurz1.hatenablog.com/entry/2018/12/20/190851

## より多くの層で、Deformable Convを使う
- v1では3層でしかDeformable Convを使っていなかったのを、v2では12層で使う
- 計算量とのトレードオフを考えても精度は上がる？
    - 同じ計算量で、チャンネルなどを増やした場合と、Deformable convにした場合で後者の方が効率がいい？

## Modulated Defomable Modules
- **Modulated Defomable Modules**: v1のDefomableが、サンプリング場所(offset)だけを動的に変えていたのに対し、絶対値(modulation)も動的に変える
- CNNやROI Poolingでは、サンプリング場所は元々固定だったので、可変で学習可能に変えるのは意味がありそうだが、絶対値は元々weightで可変で学習可能だったので、追加のパラメータを導入する意味はあまりないのでは？
    - パラメータ数が増えるので、精度は上がるかも知れないが… その分、計算量なども増えるだろうし。
    - 実際、実験結果でもあまり精度が上がっていない気がする

## R-CNN Feature Mimicking
- Deformable Convでは、物体領域外の有効でない場所に、サンプリング点がはみ出てしまうことがある。
- それを防ぐために、物体矩形のみから特徴量を計算するR-CNN（そのまま使うのは、矩形ごとにCNNのforwardを計算するので効率が悪い）の特徴量を真似るようなLossを追加して学習する
- 蒸留的な役割をして、学習しやすくなるという点では良さそう
    - これがないと、v2は学習できない？ より学習を安定化させるために追加しただけ？
- そもそも、サンプリング場所(offset)を計算するためのCNNの入力特徴量を、もっと広い範囲のものを使った方が良さそう
    - サンプリング場所を計算する畳み込み層の入力は、サンプリング場所が固定のCNNの入力特徴量と同じものを使っている
    - サンプリング場所は、元の入力特徴量の場所より外の領域に行きうるので、よく無さそう（receptive fieldの範囲には収まっているかも知れないが）
    - もしくは、offsetが外の範囲に行かないようなハードな拘束を入れる設計にした方が良さそう。R-CNN Feature Mimickingみたいなソフトな拘束にするより。


# 参考文献
- 蒸留/Feature Mimicking
    - [2] J. Ba, et al. "Do deep nets really need to be deep?" (2014)
    - [22] G. Hinton, et al. "Distilling the knowledge in a neural network" (2015)
    - [28]  Q. Li, et al. "Mimicking very efficient network for object detection" (2017)

