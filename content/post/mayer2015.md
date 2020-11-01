---
title: "A Large Dataset to Train Convolutional Networks for Disparity, Optical Flow, and Scene Flow Estimation"
date: 2020-11-02T00:42:01+09:00
lastmod: 2020-11-02T00:42:01+09:00
draft: false
tags: ["Stereo Matching", "Optical Flow", "Scene Flow", "Dataset"]

---
# 論文情報
[N. Mayer et al. "A Large Dataset to Train Convolutional Networks for Disparity, Optical Flow, and Scene Flow Estimation" (2015) CVPR2016](https://arxiv.org/abs/1512.02134) 

## リンク
- [プロジェクトページ](https://lmb.informatik.uni-freiburg.de/Publications/2016/MIFDB16/)
- [プロジェクトページ](https://lmb.informatik.uni-freiburg.de/projects/synthetic-data/)
- [Youtube](https://www.youtube.com/watch?v=1iAQp6KmhE4)

## 著者
Optical flowで有名な[フライブルク大学(University of Freiburg)のComputer Vision Group](https://lmb.informatik.uni-freiburg.de/index.php)のメンバーとVisual SLAMで有名な[ミュンヘン工科大学(TUM, Technical University of Munich)のComputer Vision Group](https://vision.in.tum.de/home)のメンバー
- [Nikolaus Mayer](https://lmb.informatik.uni-freiburg.de/people/mayern/) FlowNet2など
- [ddy Ilg](https://lmb.informatik.uni-freiburg.de/people/ilge/) FlowNet1, 2など
- [Philip Häusser](https://vision.in.tum.de/members/haeusser) FlowNetなど
- [Philipp Fischer](https://lmb.informatik.uni-freiburg.de/people/fischer/) FlowNetなど
- [Daniel Cremers](https://vision.in.tum.de/members/cremers) ミュンヘン工科大のボス。LSD-SLAM, FlowNet, DSOなど。
- [Alexey Dosovitskiy](https://lmb.informatik.uni-freiburg.de/people/dosovits/) FlowNet1, 2など
- [Thomas Brox](https://lmb.informatik.uni-freiburg.de/people/brox/) フライブルク大のグループのボス。Unet, Horn-schunkベースのOptical flow, FlowNetなど。


# 概要
- **Scene Flow**と呼ばれるOptical flow・視差・Scene flowのデータセットを作成
- 既存の視差・Scene flow推定のデータセットはCNNを学習できるほど大規模なものが無かったが、このデータセットでCNNの学習が可能になった
- Blenderを使った合成画像のデータセット


# 内容
## SceneFlow
- **Scene flow**はstereo video/RGBD videoに写っている場所の(Depth/3D positionと)3D motionを推定するタスク。
- Scene flowは、視差とOptical flow(とdisparity change)の推定結果から求めることができる。ただし、カメラの内部・外部パラメータが与えられている時のみ計算可能。(disparity changeは視差の時間変化のこと)
- disparity changeはほぼ冗長な情報だが、視差とOptical flowだけではOcclutionがある領域でScene flowが推定できないのに対して、disparity changeがあると推定できる。
- また、視差とOptical flowだけだと、Optical flowの小さい変化でも3D motionは大きく変化しうるのでノイズに弱いが、disparity changeを使うとノイズに強くなる。

## データセット
{{< figure src="/images/mayer2015/table1.png" caption="[N. Mayer+(2015)] Table 1より引用。" class="center" width="920" >}}
- 既存のデータセット
    - Middlebury[1, 21, 22]: 視差推定は実シーン、Optical flowは実シーンと合成画像、データ数は少ない。
    - MPI sintel[2]: すべて合成画像、元々はOptical flowだけだったが、視差も追加された。motion blurや曇りなどの画像劣化も加えている
    - KITTI 2012, 2015[8, 17]: Optical flowと視差。車載laser scannerから作成しているのでデータがスパース(一部のPixelでしかGTがない)。
    - Flying chair[4]: optical flow用の合成画像データ。CNNが学習できるだけの量がある
- 提案データセットはすべて合成画像で、データ数が多く、Optical flow・視差・disparity change・Segmentation・motion boundary + カメラパラメータのGTがある。
- 作成方法: **Blender**を改造して、ステレオRGB画像やOptical flow、視差、Disparity changeも出力
    - optical flow: 3D motionを画像面に射影
    - 視差: 3D positionから得られるdepthからカメラパラメータを使って計算
    - disparity change: 3D motionのdepth成分から計算
    - Segmentation: オブジェクトの種類
    - motion boundary: 異なるオブジェクト間の境界で、一定以上の動きの違いで、一定以上の大きさがあるもの
- 3Dのデータから生成しているので、隠れがある領域でもdisparity changeなどのGTが作れる
- Sintelと同様に`clean pass`と`final pass`(ボケ・motion blur・太陽グレア・輝度変化などの画像劣化を加えたもの)の2つを用意
- そのままだとサイズが大きくなりすぎるので、RGB画像はWebPに、非RGB画像はLZOに
- **3つのサブデータセット**を作成
    - **FlyingThings3D**: 色々なオブジェクトがランダムな動きをしているデータセット。
    200種類の背景シーン・36000種類の3D modelオブジェクト(ShapeNetデータベースから)・TextureはImageMagick・Flickr・ImageAfterから作成。
    3D modelやTextureはTrainとTestで混ざらないように分けている。
    5-20個のオブジェクトをランダムに選択・ランダムにTextureをつける・ランダムにスケール・ランダムに動かすことで作成。
    {{< figure src="/images/mayer2015/fig3.png" caption="[N. Mayer+(2015)] Fig.3より引用。下3列はそれぞれ、Optical flow・視差・disparity change" class="center" width="430" >}}
    - **Monkaa**: "Monkaa"というオープンソースのBlenderアニメから作成。毛やソフトな物体が含まれる。
    {{< figure src="/images/mayer2015/fig1.png" caption="[N. Mayer+(2015)] Fig.1より引用。下列はそれぞれ、Optical flow・視差・disparity change" class="center" width="320" >}}
    - **Driving**: 自然な車載カメラのシーンで、KITTIを意識して作成。

## ネットワーク
- Optical flow推定は**FlowNet**[4]を踏襲(Encoder-Decoder構造)
- 視差推定は**DispNet/DispNetCorr**の2つのネットワークを提案。Table 2にあるようにEncoder-Decoder構造のCNN。
- Scene flow推定(**SceneFlowNet**)は、事前学習したOptical flow推定と視差推定のネットワークを組み合わせてfine-tuningする。加えて、disparity changeも学習。
- 実装はCaffe

## 実験
- 視差推定 (Table3)
    - FlyingThings3Dで学習しただけのものと、KITTIでfine-tuningしたもの(表で-Kとついているモデル)
        - 当然、KITTIではfine-tuningしたモデルの方が精度が良く、他のデータセットではKITTIでfine-tuningしない方が精度がいい
    - 処理速度: KITTIの解像度でGPUで15fps
    - KITTIでは、SGMとMC-CNN[28]と比較。MC-CNN[28]と同程度の精度
        - SOTAのモデル(MC-CNN[28])と精度は同等で1000倍早い
    - Table 3によると、1D correlation layerを入れると精度が上がったり下がったりするみたい（本文では入れた方が良いと言っている？）
- Scene Flow推定
    - 定性的にはOcclutionがある領域でもdisparity changeが推定できている
    - 学習が収束してないらしい


# 感想
- 最近のDLベースのstereo matching/視差推定の論文では、標準的に使われている
- Optical flowとか視差みたいな実画像でGTを作るのが大変なタスク(視差はスパースで良ければKITTIみたいにLIDARとかを使う手はあるが)は、こういった合成画像を使うしかないので、実画像で使う時の汎化性能が問題になりそう。


# 参考文献
- [4] A. Dosovitskiy et al. "FlowNet: Learning optical flow with convolutional networks" (2015): flying chairs dataset(synthetic data of optical flow)

