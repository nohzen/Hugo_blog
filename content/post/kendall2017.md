---
title: "Goemetry and Context Network (GC-Net)"
date: 2020-10-26T01:01:31+09:00
lastmod: 2020-10-26T01:01:31+09:00
draft: false
tags: ["Stereo Matching"]

mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true
---

# 論文情報
[A. Kendall et al. "End-to-End Learning of Geometry and Context for Deep Stereo Regression" (2017)](https://arxiv.org/abs/1703.04309)

## リンク
- [ICCV2017 open access](https://openaccess.thecvf.com/content_iccv_2017/html/Kendall_End-To-End_Learning_of_ICCV_2017_paper.html)
- [ICCV2017 oral](https://www.youtube.com/watch?v=VtAzDS1NLmo)

野良実装（著者実装はなさそう）
- https://github.com/kelkelcheng/GC-Net-Tensorflow
- https://github.com/Jiankai-Sun/GC-Net
- https://github.com/zyf12389/GC-Net
- https://github.com/laoreja/CS231A-project-Stereo-matching
- https://github.com/EnriqueSolarte/GC-Net-tensorflow

## 著者
[Skydio Inc.](https://www.skydio.com/)というドローンを作っている会社のメンバー
- [Alex Kendall](https://alexgkendall.com/): 現University of Cambridge、SegNetやPoseNetの著者
- [Abraham Bachrach](http://people.csail.mit.edu/abachrac/pmwiki/): skydio CTO, 他にもVOの論文とかあり
- [Adam Bry](): skydio CEO
他: Hayk Martirosyan, Saumitro Dasgupta, Peter Henry, Ryan Kennedy


# 概要
**Goemetry and Context Network (GC-Net)** と呼ばれるStereo matchingを行うDeepLearning手法を提案した。  
**GC-Net**は以下のような特徴がある。
- **End to end**に学習できる
- Goemetry: 視差推定の問題の構造を利用するために、**cost volume**を使う
- Context: cost volumeの正則化に**3D conv**を利用し、semanticな情報も学習する
- 視差は**soft argmin**を使ってcost volumeから**回帰する**


# 内容
## 背景
既存のDeep learningを使ったStereo matching手法はEnd to endではなく、特徴量を作るところだけがDLで、正則化や後処理は古典的な方法を使っていた。
(正則化: stereo matchingのエネルギーに平滑化項を入れる / cost volume filteringの平滑化のためのfilter)  
本研究では、古典的なstereo matching手法に含まれる各要素を微分可能なLayerにすることで、End to endに学習できるDL法モデルを提案。
cost volumeを使うという点でstereo matchingの問題の構造を取り入れつつ、DLを使うことでsemanticな情報/contextを利用できるのが強み。  
新規性は以下の2点:
- 高さx幅x視差の3次元のcost volumeを3D convで正則化
- soft argmin: 微分可能で学習でき、cost volumeからsub-pixel精度で視差を回帰できる

## ネットワーク構造
{{< figure src="/images/kendall2017/Fig1.png" caption="[Kendall+(2017)] Fig.1" class="center" width="1213" >}}

### 特徴量抽出
Cost volume filtering [Rhemann+(2013)]などの非DL系の手法では、画素値を特徴量としてcost volumeを作ることが多い。  
一方、本研究では画素値そのものではなく、画素値に畳み込みをかけたものを特徴量としてcost volumeを計算する。
こうすることで、画像の明るさの変化に強くなり、局所的なcontextを利用できる。
特徴量を計算するCNNの重みは、**左右の画像で共有**する。

### Cost volume
2枚の画像から計算された特徴量(高さx幅x特徴量)を**concate**して高さx幅x視差x特徴量のcost volumeを作る。
他にも、特徴量の次元について引いたり、特徴量間の距離を使ってcost volumeを作る方法も考えられる。
[Rhemann+(2013)]などの非DL系の手法では特徴量(画素値)の引き算をしている。[32]では特徴量の次元については内積をして潰している。  
**特徴量間の距離などを使うと、相対的な表現しか学習できず、絶対的な特徴量がcost volumeの正則化で使えない**。
一方、concateするとcost volumeで絶対的な特徴量が使え、semanticな情報を学習できる。

### cost volumeの正則化/refinement
[Rhemann+(2013)]では空間方向にしかfilterしないが、**3D convを使うことで空間方向に加えて視差方向にも学習**できる。
ただし、3D convは計算量が多いので、**Encoder-Decoder構造**(空間+視差の次元を4回縮小)を使う。
最後の畳み込みで元解像度になるようにUpscaleする。

### argmin
単純にcost volumeの視差の次元についてのargminで視差を推定すると、sub-pixel精度にならない・微分可能でないので学習できない といった問題がある。  
そこで以下のような**soft-argmin**を使うことで、sub-pixel精度の回帰ができ、微分可能で学習可能になる。
各pixelにおいて、Cost $C(d)$のマイナスのsoftmaxをとり確率にする。それを重みとして視差を加重平均することで視差を推論する。
$$
\hat{d} := \sum_d d * \frac{e^{-C(d)}}{\sum_{d^{'}} e^{-C(d^{'})}}
$$

普通のargminと違い、コストがmulti-modalな形をしている場合には以下の図の(b)のように問題が起こる。
{{< figure src="/images/kendall2017/Fig2.png" caption="[Kendall+(2017)] Fig.2" class="center" width="1200" >}}
そこで、最後の3D convの後にBNを入れないことで、**ネットワークがsoftmaxの「温度」を学習してpeakの鋭さをコントロール**できる(統計力学のカノニカル分布と同じ形をしているので、「温度」と呼んでいる)。
つまり、multi-modalの時にはCの絶対値が大きくなり、peakが鋭くなる。

### Loss
**視差のL1ノルム**をLossとする。
KITTIデータセットのLIDARのようにGTがスパースな場合もあるので、pixel数で割って平均する。  
分類問題としても扱うことができるが、**回帰問題**として扱うことでsub-pixel精度にできる。
また、回帰問題として定式化することで、photometric lossを使った教師なし学習も使うことができる[13]。

## 実験
TensorFlowで実装、バッチサイズ1、入力サイズ:256x512、入力画素値は[-1, 1]に正規化。
- Scene Flowでモデル設計の違いを比較(Table2)
    - 3D conv + 多重解像度 > 3D conv single resolution > 3D convなし
    - Regression loss (soft argmin) > hard classification loss (普通の分類のやり方で、cost volumeとして確率を推論し、cross entropy lossを使う) > soft classification loss (Gaussian分布を学習) [32]
    - Table2を見ると、3D convありだとregression lossの方がいいが、3D convなしだとclassification lossの方がいい（理由はある？）
    - Fig.3: Classification lossの方が学習が早く進むが、最終的な精度はregression lossの方がいい
- KITTIで先行研究と比較(Table3)
    - KITTIはScene FlowでPretrain(学習データが十分にないので)
    - 先行研究より高性能で処理速度も早い(Table3に処理速度の比較あり)
    - End to end, regression loss, 3D convが効いていると考察
- Model saliencyの考察
    - [51]の方法を使って、有効的な受容野を可視化


# 感想
- DLを使ったStereo Matchingの論文ではトップクラスに引用数が多く、後の研究でよく比較されている。
- 非DL系の有名手法であるCost volume filtering [Rhemann+(2013)]を真っ当に学習可能にしたという感じ。
    - ただし、cost volumeの正則化(filtering)を視差の足について独立に行うのではなく、3D convにしているあたりは工夫がなされている。
    - cost volumeを使うとしても、3D conv, soft argminあたりは他のやり方もありそう。
- Lossに単純なL1を使っているが、smooth L1とかにすると精度が上がったりしないだろうか
- ClassificationとRegressionの比較は面白い
    - Table2によると、3D conv + 多重解像度ではregression lossの方が精度が高かったみたいだが、3D convなしではclassification lossの方が精度が高いみたい。論文の本文でそれについての言及がなかったが、理由などが気になる。
    - また、3D conv + 多重解像度では、classification lossの方が学習が早く進み、regression lossの方が精度が高いというのも不思議で理由が気になる。
    - 単眼のDepth推定では、Ordinal Regressionみたいなclassification的に扱った方が精度が上がるという話もあった気がする。


# 参考文献
review:
- [39] D. Scharstein et al. "A Taxonomy and Evaluation of Dense Two-Frame Stereo Correspondence Algorithms" (2002)
- [43] F. Tombari et al. "Classification and evaluation of cost aggregation methods for stereo correspondence" (2008) 

Patch baseの先行研究:
- [32] W. Luo et al. "Efficient deep learning for stereo matching" (2016): soft classification lossを利用
- [34] N. Mayer et al. "A Large Dataset to Train Convolutional Networks for Disparity, Optical Flow, and Scene Flow Estimation" (2015):  E2E, regrssion loss, 1D correlation layer

非DL:
- [C. Rhemann et al. "Fast Cost-Volume Filtering for Visual Correspondence and Beyond" (2013)](https://www.ims.tuwien.ac.at/publications/tuw-202088)  前提知識
