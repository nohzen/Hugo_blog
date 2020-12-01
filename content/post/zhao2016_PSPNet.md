---
title: "Pyramid Scene Parsing Network (PSPNet)"
date: 2020-11-30T01:08:08+09:00
lastmod: 2020-11-30T01:08:08+09:00
draft: false
tags: ["Semantic Segmentation"]

---
[H. Zhao, et al. "Pyramid Scene Parsing Network" (2016)](https://arxiv.org/abs/1612.01105)

# 概要
- Semantic Segmentationを行うFully Convolutional networkに、**Pyramid Pooling Module**を追加した**Pyramid Scene Parsing Network**(PSPNet)を提案した
- Pyramid Pooling Moduleによって、glabalなコンテキストを考慮したSegmentation結果が得られる
- ImageNet semantic segmentation 2016で1位


# 論文情報
## リンク
- [プロジェクトページ](https://hszhao.github.io/projects/pspnet/index.html)
- [公式実装 Caffe](https://github.com/hszhao/PSPNet)
- [公式実装 PyTorch](https://github.com/hszhao/semseg)

## 著者
University of Hong KongとSenseTimeのメンバー
- [Hengshuang Zhao](https://hszhao.github.io/) 現Oxford。Segmentation系の研究。
- [Jianping Shi](https://shijianping.me/) SenseTime
- [Xiaojuan Qi](https://xjqi.github.io/) University of Hong Kong
- [Xiaogang Wang](http://www.ee.cuhk.edu.hk/~xgwang/) University of Hong Kong
- [Jiaya Jia](http://jiaya.me/) University of Hong Kong


# 内容
- ネットワーク名にもある"Scene parsing"は、Semantic segmentationと同じ意味。
    - 画像の各pixelに、カテゴリを割り当てるタスク
    - Instance segmentationと違って、instanceは区別しない(2台の車が写っていても、どちらも同じ「車」というカテゴリラベルをつける)

## Semantic segmentationの既存手法の問題点
- 最近のSemantic segmentationでは、fully convolutional network (FCN)をベースとした手法が多い
- FCNベースの手法は、画像全体にわたるグローバルな情報・特徴量を使えていないという問題点がある。
- 以下の画像が具体例。
{{< figure src="/images/zhao2016_PSPNet/Fig2.png" caption="[Zhao+(2016)] Fig. 2より引用" class="center" width="1000" >}}
    - 1列目: FCNでは、ボートが車と間違えられている。画像に写っているのが、川の近くのボートハウスのシーンというコンテキスト / priorを考慮すると、正しいラベルがつけれるはず(車はふつう川の上にない)。
    - 2列目: FCNでは、一部が建物に、一部が超高層ビルに分類されている。全体が建物 or 超高層ビルのどちらかになっているはずで、画像全体で他の場所との整合性が取れていない（この問題は、提案手法でも解決されていない気がする。提案手法でも、他のpixelがどう分類されるかは、該当pixelの結果と相互作用する訳ではないので）
    - 3列目: FCNでは、ベッドと模様が似た枕がベッドと分類されている。ベッドの端には枕があることが多いというコンテキストがあると、検出しやすい。また、ベッドは画像全体のサイズではなく、画像の一部にだけ写っているので、ある領域でのコンテキストが必要（"pyramid" poolingを使う理由）

## Pyramid Scene Parsing Network (PSPNet)
- グローバルなコンテキストを得るには、receptive fieldを大きくする必要がある。
    - CNNの実効的なreceptive fieldは、理論的な最大範囲よりも小さいことが知られている[42]。
- poolingを使えば、実効的なreceptive fieldを広くできる
    - 先行研究[24]では、global average poolingを使うことで、グローバルなコンテキストを利用してFCNの精度を向上させている。
    - [24]では画像全体で一つの特徴量しか利用しておらず空間的な情報を失っているので、[SPP-net](**Spatial pyramid pooling**)にならって、複数のスケールでのグローバルなコンテキストを利用する。
    - [SPP-net]のSpatial pyramid poolingはClassificationを対象としていたため、各スケールで得たpoolingの値をフラットにして、fully connectedに渡していた
- Spatial pyramid poolingをsemantic segmentationに使えるようにしたのが、本研究で提案する**Pyramid Pooling Module**(下図)
{{< figure src="/images/zhao2016_PSPNet/Fig3.png" caption="[Zhao+(2016)] Fig. 3より引用" class="center" width="1200" >}}
    - 前段のCNNの最終層を受け取って、4種類のスケールでaverage poolingする (実験結果で、max poolingより精度が良かった)
    - Poolingした結果の特徴量の次元を減らすために、1x1 conv
    - CNNの最終層(ローカルな特徴量)と、4種類のPooling結果をbilinear upsamplingしたもの(グローバルな特徴量)をconcate
- **Pyramid Scene Parsing Network** (PSPNet)は、dilated Conv(receptive fieldを広げられる)を使ったFCNをベース[3][26]として、間にPyramid Pooling Moduleを入れたもの
- Pyramid Pooling Moduleは、Semantic segmentation以外のpixel-levelのタスク(stereo matching, optical flow, depth estimation, etc.)でも有効と思われる

## 最適化の工夫
- 以下の図のように、最終的なLossに加えて、ネットワークの途中の段階でもLoss(auxiliary loss)を追加する。
- auxiliary lossで学習がしやすくなる
{{< figure src="/images/zhao2016_PSPNet/Fig4.png" caption="[Zhao+(2016)] Fig. 4より引用" class="center" width="500" >}}

## Sec5 実験結果
未読


# 感想
- 既存手法の問題点 -> 解決策としての提案手法 の流れがとても理にかなっていて、読みやすかった
    - 最適化の話を除くと、ツッコミどころがない
- encoder-decoder系の方法では、実効的なreceptive fieldがあまり広がっていない的な話がイントロか何かにあってもいい気がした（CNNの実効的なreceptive fieldがあまり広くないという話は書かれていたが）
    - 2016年の段階では、まだencoder-decoder系のsemantic segmentationの手法が提案されてなかった？
- ネットワークの途中の段階にもLossを追加する話は、深いネットワークが学習しにくくなる問題を緩和できるってのは分かるけど(層が浅くなるので、勾配が消失しにくい)、それで最終結果も良くなるかは分からない気がする。
    - 余分なLossがあると。ちゃんと学習できた場合に、余分なLossがない場合に比べて精度が下がったりしない？
    - 余分なLossのweightは、学習が進むにつれてだんだん小さくすればいい？最適化も絡む問題なので、ちゃんとしたことを言うのは難しそう。


# 参考文献
- Spatial pyramid pooling
    - [18] S. Lazebnik, et al. "Beyond bags of features: Spatial pyramid matching for recognizing natural scene categories" (2006) DL以前のSpatial pyramid pooling
    - [12] [SPP-net] K. He, et al. "Spatial pyramid pooling in deep convolutional networks for visual recognition" 2014 ClassificationでのSpatial pyramid pooling
- dilated FCN
    - [26] [FCN] J. Long, et al. "Fully convolutional networks for semantic segmentation" (2015)
    - [3] L. Chen, et al. "Semantic image segmentation with deep convolutional nets and fully connected crfs" (2014)
    - [40] F. Yu, et al. "Multi-scale context aggregation by dilated convolution" (2015)
- [42] B. Zhou, et al. "Object detectors emerge in deep scene cnns" (2014) CNNの理論的なreceptive filedの範囲より、実際の有効な範囲は狭い

