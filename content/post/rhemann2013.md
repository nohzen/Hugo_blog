---
title: "Fast Cost-Volume Filtering for Visual Correspondence and Beyond"
date: 2020-10-04T17:15:50+09:00
lastmod: 2020-10-04T17:15:50+09:00
draft: false
tags: ["Stereo Matching", "Optical Flow", "Segmentation"]

mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true
---

# 概要
- Stereo matching, optuical flow, interactive segmentationなどをラベル問題として定式化
- コスト関数の平滑化にGuided Filterを用いることで、精度を落とさずに高速化

# 論文情報
- [論文website](https://www.ims.tuwien.ac.at/publications/tuw-202088)
公式Matlabの実装が公開。論文に依ると、CUDA実装もしたみたいだが、非公開。
- [野良実装?](https://github.com/fjordyo0707/StereoMatching-CostFilter)
- https://slideplayer.com/slide/6108371/
- https://slideplayer.com/slide/8874463/

## 著者
ウィーン工科大学, Microsoftの人たち
- Christoph Rhemann(現Google)
- Asmaa Hosni
- Michael Bleyer(現Microsoft)
- [Carsten Rother](https://hci.iwr.uni-heidelberg.de/vislearn/people/carsten-rother/) (ハイデルベルク大学)  
GrabCutの著者。[この方の研究室](https://hci.iwr.uni-heidelberg.de/vislearn/)、最近の研究も面白そう。
- [Margrit Gelautz](https://www.ims.tuwien.ac.at/people/margrit-gelautz) (ウィーン工科大学)

# 内容
## Stereo matchingについて具体例
{{< figure src="/images/rhemann2013/stereo_match.png" class="center" width="320" >}}
1. cost volume $C_{x, y, l}$は2枚の画像 $I, I^{'}$ の変位$l$の時の差分と、x方向の微分の差分の重み付き和(上の画像の(b))
{{< figure src="/images/rhemann2013/cost_volume.png" width="320" >}}
2. 変位ごとに、元の画像をガイドとして重み付き平均する(上の画像の(e))
{{< figure src="/images/rhemann2013/smooth.png" width="160" >}}
3. 平滑化したcost volumeのargminが解となる変位
4. occlusionがあるところ（左右の画像の両方をベースにして2回マッチングをし、左右の結果がコンシステントでない場所）は周りの変位で修正する

## ラベリング問題
以下の3つの例は、画像の各Pixelのラベリング問題として解釈できる。
- Stereo matchingは平行化した2枚の画像の間に、各Pixelでどれくらい変位があるかを求める問題。各Pixelに対して、変位を割り当てるラベル問題と解釈できる。
- Optuical flowは各Pixelに対して、動きベクトルを求める問題。これも各Pixelに対して、変位を割り当てるラベル問題と解釈できる
- interactive segmentationはユーザーが一部のPixelに対して与えた初期ラベルを元に、各Pixelを前景と背景に分類する問題。2ラベル問題になっている。

## 手法
画像の各Pixelのラベリング問題を解くアルゴリズムとして以下を考案。
1. 各Pixel・各ラベルに対して、コストを計算。コストはpixel座標$x, y$とラベル$l$のindexをもつので3次元なので**cost volume** $C_{x, y, l}$と呼んでいる
2. cost volumeを、元画像をガイドとしてGuided Filterで平滑化
3. 平滑化されたcost volumeのargminを取ることで、最終的なラベルを決定

- 先行研究[31]では2.のcost volumeの平滑化をBilateral filterでやっており、それにより元画像のエッジが保存される結果が得られていたが、処理時間が遅かった。  
本研究では、cost volumeの平滑化にGuided filterを使うことで、エッジ保存の平滑化を高速に実行できる。  
（Guided filterは、積分画像を使って高速に計算可能なBox filterの組み合わせで実装できるため早い）
- sub-pixel精度が必要な場合
元の画像をbicubic拡大して、sub-pixelの変位の場合のコスト関数も計算する。
コスト関数の変位の次元は大きくなるが、座標の次元はそのまま。

# 処理速度
- 1Mpix画像に対して5msec（CUDA実装）
- 論文の処理速度はCUDA実装で測定した結果で、公開されているのはMatlab実装なのが悲しい

# 感想・批判
- [Khamis+(2018)]などで引用されていたので目を通した。
- Stereo matchingで、cost関数をただの差分だけに(x方向の微分の項をなくす)して、平滑化をbox filterにすると、シンプルなSADでのブロックマッチングに帰着する。それに比べて平滑化をbilateral filterとかguided filterにすると、画素値が近い周りのpixelの影響は大きくなるが、画素値が遠い周りのPixelの影響は小さくなる。このように画素値の違いで寄与を変えると、異なる奥行きの物体間の境界などでは良さそうだが、同じ物体でTextureがある時などに悪影響がありそう。実験でCG画像で精度が悪いという結果は、これが関係してそうな気がする。（学習ベースにして、単に画素値で寄与度を決めるのではなくて、同じ物体に属するかで決めると精度が上がりそう。）
- costにただのL1ノルムではなく、カットオフみたいなものを導入している。L2よりL1の方が外れ値に強いが、更にそれを推し進めた感じ。初めて見たが、一般的なのだろうか？

# 参考文献
- [S. Khamis et al. "StereoNet: Guided Hierarchical Refinement for Real-Time Edge-Aware Depth Prediction" (2018)](https://arxiv.org/abs/1807.08865)


