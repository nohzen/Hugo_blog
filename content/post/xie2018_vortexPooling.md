---
title: "Vortex Pooling"
date: 2020-12-28T01:34:07+09:00
lastmod: 2020-12-28T01:34:07+09:00
draft: false
tags: ["Semantic Segmentation"]

mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true
---
[C.-W. Xie, et al. "Vortex pooling: Improving context representation in semantic segmentation" (2018)](https://arxiv.org/abs/1804.06242)


# 概要
- DeepLab v3のAtrous Spatial Pyramid Pooling(ASPP) moduleを改良した**Vortex Pooling** moduleを提案し、Semantic Segmentationで精度が向上することを確認


# 論文情報
## リンク
- 野良実装
    - https://github.com/MTCloudVision/deeplabv3-mxnet_gluon

## 著者
Nanjing University（南京大学）のグループ
- [Chen-Wei Xie](http://www.lamda.nju.edu.cn/xiecw/?AspxAutoDetectCookieSupport=1)
- [Hong-Yu Zhou](http://zhouhy.org/)
- [Jianxin Wu](https://cs.nju.edu.cn/wujx/)


# 内容
- 単純な畳み込み層では実効的なreceptive fieldが狭く、Globalなコンテキストが有効に活用できない
- そこで、DeepLabの**Atrous Spatial Pyramid Pooling** (ASPP) moduleや**Spatial Pyramid Pooling** moduleのような複数のスケールの特徴量を合体させる手法が提案されてきた
    - ASPP moduleとPSP moduleを合体させたような**Vortex Pooling**を提案した
- DeepLab v3のASPP moduleに比べて、少し精度が上がり、少し処理時間が増えた

## 先行研究: Atrous Spatial Pyramid Pooling (ASPP)
- 異なるdilation(例えば1, 12, 24, 36)のAtrous conv (=Dilated conv)をConcateすることで、様々なスケールの特徴量を利用する（下図）
{{< figure src="/images/xie2018_vortexPooling/Fig2.png" caption="[Xie+(2018)] Fig. 2より引用" class="center" width="500" >}}
- DeepLab v3のASPPでは、4つのdilation convに加えて、global average poolingもconcateする

## 先行研究: Spatial Pyramid Pooling
- 入力特徴量の解像度に依らず、固定のBox数のAverage Poolingを行い、concateする（下図）
{{< figure src="/images/xie2018_vortexPooling/PSP_Fig3.png" caption="[PSPNet] Fig. 3より引用" class="center" width="400" >}}

## Vortex Pooling
- ASPPでは、入力特徴量のほんの一部のpixelしか出力に寄与しない
- そこで、dilated convをする前にkxk average poolingをすることで、寄与するpixelを増やす（下左図: Module A）
- 大きいdilationのdilated convほどGlobalな情報なので、average poolingのカーネルを大きくする（下右図: Module B = Vortex Pooling）
    - Module Bの方がModule Aより精度が高くなり、工夫して計算することで計算量も減らせる
    - ave poolingの後に3x3 convを行っているが、これが1x1 convならSpatial Pyramid Poolingっぽくなる（厳密には、Spatial Pyramid Poolingはbilinear補間するのに対し、Vortex poolingは全部真面目に計算するので別物）
{{< figure src="/images/xie2018_vortexPooling/Fig3.png" caption="[Xie+(2018)] Fig. 3より引用" class="center" width="900" >}}
- Module B (Vortex Pooling)は、average poolingのカーネルを$k, k^2, k^3$のようなサイズに設定することで、下図のように小さいカーネルのaverage poolingの計算結果を元に次に大きいカーネルのaverage poolingの計算を行える。
{{< figure src="/images/xie2018_vortexPooling/Fig4.png" caption="[Xie+(2018)] Fig. 4より引用" class="center" width="500" >}}
- DeepLab v3と同様に、Global Average Poolingと合わせてConcateする
{{< figure src="/images/xie2018_vortexPooling/Fig5.png" caption="[Xie+(2018)] Fig. 5より引用" class="center" width="800" >}}


# 感想
- ASPPに比べてけっこう計算量が増えそう。ただ、Dilated convは演算量が少なくてもキャッシュ効率が悪いので、演算量ほど差はないかも知れない。
    - 処理時間は、ネットワーク全体のものしか書かれていないが、ASPP vs Vortex pooingのところだけだとどれくらいの比なのか気になった
- Module B (Vortex Pooling)の入力は前段のネットワークで計算した特徴量だが、それを画像だと思うとImage Pyramidに似ている
    - 厳密にはカーネルが大きくなっても解像度は小さくならないので、Image Pyramidとは少し違う
    - Spatial Pyramid Poolingの方がまんまImage Pyramid
    - そう考えると、ASPPはlow-pass filterをしてないImage Pyramidみたいになっている。**ASPPはエイリアス**が発生しそう。
- なぜ、Vortex Poolingって名前？


# 参考文献
- [PSPNet] [H. Zhao, et al. "Pyramid Scene Parsing Network" (2016)](https://arxiv.org/abs/1612.01105)
- [DeepLab v3] [L.-C. Chen, et al., "Rethinking Atrous Convolution for Semantic Image Segmentation" (2017)](https://arxiv.org/abs/1706.05587)
