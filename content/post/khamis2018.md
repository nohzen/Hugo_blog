---
title: "StereoNet: Guided Hierarchical Refinement for Real-Time Edge-Aware Depth Prediction"
date: 2020-11-03T19:49:36+09:00
lastmod: 2020-11-03T19:49:36+09:00
draft: false
tags: ["Stereo Matching"]

---
[S. Khamis et al. "StereoNet: Guided Hierarchical Refinement for Real-Time Edge-Aware Depth Prediction" (2018) ECCV2018](https://arxiv.org/abs/1807.08865)

# 概要
- Stereo matchingを行うDNN **StereoNet**を提案
- 先行研究GC-Netと同じく、**Cost volume**と**3D conv**を利用
- GC-Netと違い、**cost volumeを低解像度にして後段でRefinement**をすることで、精度を保ちつつ**高速化**


# 論文情報
## リンク
- https://wiki.davidl.me/view/StereoNet:_Guided_Hierarchical_Refinement_for_Real-Time_Edge-Aware_Depth_Prediction
- 野良実装(著者実装は非公開)
    - https://github.com/zhixuanli/StereoNet
    - https://github.com/meteorshowers/StereoNet-ActiveStereoNet

## 著者
Googleの研究グループ
- [Sameh Khamis](https://www.samehkhamis.com/) 現Nvidia
- [Sean Fanello](http://www.seanfanello.it/) 現Google
- [Christoph Rhemann](https://www.linkedin.com/in/christoph-rhemann-75340b44/) 現Google、Cost volume filteringの著者
- [Adarsh Kowdle](https://www.linkedin.com/in/adarshkowdle/) 現Google、
- Julien Valentin 現Microsoft
- Shahram Izadi 現Google


# 内容
## 先行研究
- 古典的な方法
    - Globalな方法: コスト関数をBelief propagationやGraph cutで最小化
    - Localな方法: Block Matchingなど。windowが小さすぎると中にTextureが無くなり、windowが大きすぎると視差が異なる場所が含まれてエッジが鈍る。そこで、画素値が中心の画素と似ている場所のweightを大きくする。これはcost volume filtering [Rhemann+(2013)]として定式化できる。計算量はどれくらい細かい視差を求めるかに比例する。
- Deep Learning
    - よくあるのはEncoder-Decoder Networkだが、ステレオマッチング問題の幾何学的な特徴を利用できてない。
    - 問題の構造を利用して、異なるカメラ・異なる解像度に対して再学習しなくても使えるネットワークを作る。
- 本研究は、cost volume filtering [Rhemann+(2013)]をDLにした手法[GC-Net]をベースとする。
    - [GC-Net]と同じ点: 
        - End to end
        - Cost volumeで定式化
        - Cost volumeのための特徴量も学習
        - Cost volumeを3D convで正則化
        - soft argminで視差推定
    - [GC-Net]と異なる点(カッコ内はGC-Net): 
        - Cost volumeは特徴量のdiff (vs 特徴量のconcate)
        - 3D convは荒い解像度のみ (vs 3D convはEncoder-Decoder構造で)
        - soft argminで求めた荒い視差をRefinement (vs Refinementはなし)
        - Lossはsmooth L1系 (vs L1 Loss)

## 手法
{{< figure src="/images/khamis2018/fig1.png" caption="[Khamis+(2018)] Fig. 1より引用" class="center" width="1000" >}}

### 前段: 荒い視差をCost volume filteringで推定
1. 平行化された2枚の画像に対して特徴量を計算
    - 古典的な方法cost volume filtering [Rhemann+(2013)]では、画素値そのものを特徴量として使っているが、本研究ではCNNで学習したものを特徴量として使う([GC-Net]と同様)
    - 特徴量は**2枚の画像で同じweight**のネットワークで計算する。Textureがない領域でも機能するように、受容野を十分大きくする。
    - **特徴量は低解像度**(入力解像度の1/8 or 1/16)で、特徴量次元は32 ([GC-Net]は入力解像度の1/2)

2. 特徴量からcost volume(幅x高さx視差x特徴量)を構成
    - 古典的な方法cost volume filtering [Rhemann+(2013)]では、視差だけずらした特徴量の差でcost volumeを構成する
    - 本研究でも同様に、**特徴量の差でcost volumeを構成**(ただし、古典では特徴量が画素値の1次元だが、本研究では学習した32次元)
    - [GC-Net]では、特徴量をconcateしてcost volumeを構成していた
        - [GC-Net]では差よりconcateの方が、後段に相対的な情報だけでなく絶対的な情報も伝達できるので良いと言っている
        - 本論文では、実験で差でもconcateでも精度が変わらなかったと言っているが、実験結果は掲載がなかった
    - cost volumeの解像度は特徴量と同じなので、低解像度(入力解像度の1/8 or 1/16) ([GC-Net]は入力解像度の1/2)

3. cost volumeをFilter
    - 古典的な方法cost volume filtering [Rhemann+(2013)]では、空間方向にのみFilterするが、本研究では視差方向にもFilterする([GC-Net]と同様)
    - 古典的な方法と違い、Filterは**3D conv**で学習する([GC-Net]と同様)
        - 論文では、3D convでFilterしたcostを「距離」と呼んでいる(視差を推定するためのものなので)
    - **3D convは一つの解像度のみで**行う (cost volumeがもともと低解像度なので)
        - [GC-Net]はcost volumeが高解像度なので、計算量を節約するためにEncoder-Decoder構造で、複数の解像度で3D convを行っている
        - 本研究の実験結果によると、多くの処理時間を高解像度の3D convで費やしているが、精度向上は低解像度の3D convに由来しているので、こういう構成にしている
    - filter後の最終出力は、各pixel・視差ごとに1次元の特徴量

4. cost volumeの視差に関するargminをとって、視差を推定
    - 古典的な方法cost volume filtering [Rhemann+(2013)]では、cost(特徴量間のEuclidean distance)が最も小さくなる視差を選ぶ(winner-tales-all, WTA, argmin)
    - argminだと微分可能でなく学習できないので、**soft argmin**を使う
        - soft argmin: 視差をコストのsoftmaxで重み付けして平均する
        - [6]で提案され、[GC-Net]でも使われている
    - soft argmin以外にも、確率的に視差を選ぶ方法も試したが、soft argminの方が精度が良かった
        - softmaxを確率分布として、確率的に視差選ぶ。学習は期待値を最小化することで。
        - この部分はあまりよく理解してない。論文には、このタイプのargminの先行研究が列挙してあるので暇ができたら読みたい。
        - soft argminとの比較の実験結果は記載なし

### 後段: 荒い視差を、入力画像をガイドとしてRefinement
- [GC-Net]と違いcost volumeから計算する視差は低解像度なので、追加で**入力画像をガイドとしたRefinement**を行うことで、入力画像のエッジを保存した視差を推定する
- Refinementの方法は[8,7,20]を参考にしている
- 推論するのは、荒い視差と入力解像度でのGTとの差分(**delta disparity**)
    - Refinementの出力(delta disparity)とupsampleした荒い視差を足して最終出力とする
- 入力: 荒い視差を出力解像度にbiliear upsampleしたものと、RGB画像を出力解像度にリサイズしたもののconcate
    - Deconvはcheckerboard artifactが発生する場合があるので、bililear upsampleとconvを使う[40]
- atrous convを利用
- 多重解像度と、最大解像度のみの両方試した
- Lossは各解像度ごとに設定し、smoothed L1 lossっぽいもの[2] ([GC-Net]はL1 loss)

## 実験
### Scene Flow
- 2枚の画像ペアに対し、同時に2つの視差(base画像を変える)を学習した方が学習の効率がいい（なぜ？）
- Fig3: Sub-pixel精度の実験
    - 古典的な手法のsub-pixel精度は0.25pixel程度
        - 古典的な方法はpixel精度で離散的なcostを計算して、それをparabora fittingすることでsub-pixel精度にする
        - [45]のよると現実的なsub-pixel精度は1/10pixel程度、[10]によるとノイズがない場合の理論限界は1/100pixel程度
    - 学習ベースの方法だと1/30pixel程度のsub-pixel精度が出る
        - 本研究では、Refinementなしではその解像度で0.03pixel程度の精度
        - Refinementなしだと、入力解像度ではcost volumeの縮小度に応じて線形で精度が悪化する
        - しかし、Refinementをすると、ほとんどcost volumeの解像度に依らなくなる (Fig4, 5も参考)
        - したがって、本研究のモデルはcost volumeは低解像度にして、Refinementで回復している
        - ただし、あまりに小さい物体とか薄いものだとcost volumeで完全に消えてしまい、回復できなくなる
- Table1:
    - 特徴量の解像度(縮小回数K=3 or 4), Refinementで多重解像度あり/なし の比較
        - 当然、高解像度の特徴量/Refinementで多重解像度ありが精度は少しいいが、処理速度が書かれていないのでなんとも言えない
    - Scene FlowではSOTA(vs [GC-Net], CRL) 
        - 本研究でreifinementなしのモデルの方が[GC-Net]より高速で精度も高い(表には載っていないが、本文に記載)
        - [GC-Net]はパラメータが多すぎて性能が下がっている？
{{< figure src="/images/khamis2018/fig5.png" caption="[Khamis+(2018)] Fig. 5より引用" class="center" width="1000" >}}

### KITTI
- Scene Flowで事前学習してKITTIでfine-tuning (KITTIはCNNを学習できるほど学習データがない)
- Table 2, 3
    - Kittiでは先行研究(vs MC-CNN, SGM-Net, CG-Net, CRL)と変わらない精度で高速(割と差があるような？)
- 失敗している原因は、反射やocclutionのようなStereo matchingでは正解がないシーン
    - これはどっちかと言うとInpaintingのタスクで、SOTAの手法はInpaintingで効果のあるhour-glassネットワークを使っているから精度がいいと思われる

## 処理時間
- StereoNet: 720p 60Hz @GPU
    - [GC-Net] 0.95fps 960p
- Fig.7: 処理時間が一番かかっているのはRefinement


# 感想
- 元となった[GC-Net]ではconcateでcost volumeを構成しており、差などの相対的なものよりconcateのような絶対的なものの方がいいと言っているが、本研究は特徴量間の差でcost volumeを構成している
    - [GC-Net]ではconcateの方が精度がでると主張しており、本研究は変わらないといっていて、どっちが正しいのか気になった
- 古典的な手法が整数精度のcostのparabora fittingでsub-pixel精度にする方法に比べて、本研究ではsoft argminによる回帰でsub-pixel精度にしているためにsub-pixel精度がいいと思われる。
    - [GC-Net]の論文に分類問題として扱うより、回帰問題として扱った方が良いと書かれていた。
    - 本研究では「DLだと古典的な手法に比べて、sub-pixelの精度が高い」と言っているが、DLだと常にそうなのかは怪しい気がする（DLでも分類問題として扱った場合にはsub-pixel精度は高くならなさそう）
- SceneFlowだとSOTAだが、KITTIだとSOTAに結構負けているのが気になった
    - 論文では、「KITTIはInpainting的なタスクになっていて、そのせいだ」と言っているが、本当だろうか？
    - 学習データが少ない場合に精度がでないモデルになっていたりしない？
- 読みたい先行研究
    - refinement[8,7,20]
    - 教師なし[63]


# 参考文献
- [Rhemann+(2013)] A. Rhemann et al. "Fast cost-volume filtering for visual correspondence and beyond" (2013)
- [GC-Net] A. Kendall et al. " End-to-end learning of geometry and context for deep stereo regression" (2017)
- [63] 教師なし
古典survay
- [22] Hamzah et al. "Literature survey on stereo vision disparity map algorithms" (2016)
- [49] Scharstein et al. "A taxonomy and evaluation of dense two-frame stereo correspondence algorithms" (2002)

