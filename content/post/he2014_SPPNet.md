---
title: "Spatial Pyramid Pooling Network (SPP-net)"
date: 2020-11-29T16:52:18+09:00
lastmod: 2020-11-29T16:52:18+09:00
draft: false
tags: ["Classificatiom", "Detection"]

# mathjax: true
# mathjaxEnableSingleDollar: true
# mathjaxEnableAutoNumber: true
---
[K. He et al. "Spatial Pyramid Pooling in Deep Convolutional Networks for Visual Recognition" (2014)](https://arxiv.org/abs/1406.4729)

# 概要
- これまで固定の入力サイズしか扱えなかったCNNを、任意の入力サイズを扱えるようにする**Spatial Pyramid Pooling**(SPP)を提案した
- Spatial Pyramid Poolingを用いた**Spatial Pyramid Pooling Network**(SPP-net)を提案し、画像分類に加え、R-CNNより高速で高精度な物体検出を提案
- ILSVRC 2014のコンペの物体検出部門で2位、分類部門で3位(1位はGoogLeNet、2位はVGG)


# 論文情報
## リンク
- [プロジェクトページ](http://kaiminghe.com/eccv14sppnet/index.html)
- [公式実装](https://github.com/ShaoqingRen/SPP_net)
- [slide](http://kaiminghe.com/eccv14sppnet/sppnet_ilsvrc2014.pdf)

野良実装
- https://github.com/yhenon/keras-spp
- https://github.com/bahl24/SpatialPyramidPooling_Keras

## 著者
Microsoft Research Asia(MSRA)のメンバー(第2、3著者がインターンの時にやったプロジェクト)。
- [Kaiming He](http://kaiminghe.com/) 現Facebook AI Research(FAIR)。Guided Filter, ResNet, Mask-RCNNなどの第一著者
- Xiangyu Zhang, 現Megvii?。ResNet, Shufflenetなど
- [Shaoqing Ren](https://www.shaoqingren.com/) 現NIO。ResNet, FasterRCNNなど
- [Jian Sun](http://www.jiansun.org/) 現Megvii。ResNet, Faster-RCNNなど


# 内容
## 既存のネットワークの問題点
- ImageNetのコンペなどで使われる既存のClassificationのネットワークは、固定サイズ(224x224)しか入力できない
    - 前段の畳み込み層は任意の入力サイズに対して使えるが、後段の全結合層が固定サイズしか対応できないため
    - 固定サイズの入力にするために、Cropしたり(対象物体が見切れてしまう)、リスケールしたりしている(アスペクト比が変わってしまう)
{{< figure src="/images/he2014_SPPNet/Fig1.png" caption="[He+(2014)] Fig. 1より引用。上が既存のネットワーク、下が提案手法。" class="center" width="1000" >}}

## CNN以前のSpatial Pyramid Pooling
- **Bag-of-Words**(BoW)は元々、自然言語処理で使われていた手法で、文章中の単語の出てくる頻度をベクトルとして、その文章の特徴量として使う手法である（文章中で単語が出てくる順番の情報は捨てる）。
- BoWは画像処理にも応用され、画像の局所特徴量(SIFT)を考え、それの画像全体でのヒストグラムをベクトルとして、その画像の特徴量として扱う（ヒストグラムにするので、各局所特徴量の位置の情報は捨てる）[16]。
- 位置の情報を完全に捨ててしまうのは良くないということで、画像Pyramidを作成し、各Pyramidのブロックごとにヒストグラムを作る手法が提案された(spatial pyramid matching(SPM)やSpatial pyramid poolingと呼ばれる)[14, 15]。

## Spatial Pyramid Pooling(SPP) Layer
- CNN以前の手法では、SIFTなどをSpatial pyramid poolingの入力としていたが、CNNの出力の特徴量をSPPの入力とする
- 入力特徴量サイズに依らず、固定の数の多重解像度ブロックを考え(下の例では1x1, 2x2, 4x4)、各ブロックでmax Poolingする(つまり、各ブロック内では、位置情報は捨てられる)。
{{< figure src="/images/he2014_SPPNet/Fig3.png" caption="[He+(2014)] Fig. 3より引用。" class="center" width="1000" >}}
- 1x1だけの場合が、CNN以前のBoWに対応し、DLでは**Global (average/max) pooling**として知られている。
- 既存のPoolingと違い、固定サイズの出力にできる・多重解像度で色々なスケールの特徴量を含む といった利点がある

## Spatial Pyramid Pooling Network
- 畳み込み層で作った入力サイズに依存する特徴量を受け取って、Spatial Pyramid Poolingで固定サイズの特徴量にし、全結合層に渡す
- 既存のCNNと組み合わせて使え(既存手法の畳み込み層と全結合層の間にSPPを入れる)、実際、いろいろな既存ネットワークの精度向上に寄与した

## 学習
SPP-Net自体は任意のサイズの入力を受け取れるので、任意のサイズの画像で学習できるが、それだとGPUを使いにくい。
したがって、実際の学習では、以下の2通りを試している
- Single-size training: 既存手法と同じように固定サイズ(224x224)の入力画像で学習
- Multi-size training: エポックごとに入力サイズを変える

## 実験結果
Sec3以降は未読


# 感想
- PSMNetで引用されていたので読んだ
- Deep learning以前の手法を、Deep learningに取り入れているのが良い
- 物体検出への適用に関しては、2020年現在ではFeature pyramid Networkとかもっといい手法がある
- Spatial Pyramid Poolingは、全く同じものが違うスケールで画像に写っていた場合に、同じ結果を返すことが保証されていない
    - WxHの画像と、それをW/2xH/2に縮小したものが左下に写ったWxHの画像: 前者の1x1 poolingが、後者の2x2 poolingに対応しそうなので、全結合層でそういった違いを吸収できる必要がある
    - WxHの画像と、それをW/2xH/2に縮小した画像を比べた場合: 同じものでも、異なるreceptive fileldで捉えないとだめ
    - 同じものが違うスケールで写っていても、同じ結果を返すように、ある程度学習されるだろうけど、アーキテクチャ自体がそういう性質を持って欲しい
    - 画像分類だと普通、分類されるべきラベルのものが画像全体に大きく写っているので、あまり問題にならないのかも知れない
    - 物体検出だと、問題になりそう


# 参考文献
BoW/Spatial Pyramid Pooling 関係
- BoW解説 https://aidiary.hatenablog.com/entry/20100227/1267277731
- コンピュータビジョン最先端ガイド2 Sec 2.2.4
- [14] K. Grauman, et al. “The pyramid match kernel: Discriminative classification with sets of image features,” (2005)
- [15] S. Lazebnik, et al. “Beyond bags of features: Spatial pyramid matching for recognizing natural scene categories,” (2006)
- [16] J. Sivic, et al. “Video google: a text retrieval approach to object matching in videos,” (2003)





