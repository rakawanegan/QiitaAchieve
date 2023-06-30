---
title: HumanActivityRecognitionImplement-Transformer1
tags: kotaniken Transformer HUR
author: NakagawaRen
slide: false
---
論文のトレースを行った試行録。  

対象は以下の論文。  
「Inertial-Measurement-Unit-Based Novel Human Activity Recognition Algorithm Using Conformer」  

https://www.mdpi.com/1424-8220/22/10/3932  

論文の概要をまとめたものがこちら  

https://qiita.com/NakagawaRen/items/c2576ea4aa679d4416ba  

論文の趣旨はConformerだが、今回は副次的に実装されていたTransformerを実装する。  

なお、データ拡張については今回は行わず、次回以降の課題とする。  

4.2.2節によれば「inertial-based human activity recogniton with transformers」についてdim=256に変更したのみだという。  

「inertial-based human activity recogniton with transformers」でのハイパーパラメータは以下。  
|params|d|L|head|Dropout rate|  
|------|-|-|----|------------|  
|value |64|6 |8 |     0.1    |  

隠れ層の次元が64から256に変えるということは元の次元より大きくしたほうが良いということだろう。  

論文は以下。  

https://ieeexplore.ieee.org/document/9393889  

論文について概要をまとめたものがこちら  

https://qiita.com/NakagawaRen/items/c47ed3fce2f954b5612a  

以下のソースコードを参考に実装を行う。  

https://github.com/yolish/har-with-imu-transformer/tree/main  

Conv.backboneでは入力チャネル数から出力チャネル数へカーネルサイズ1の畳み込みで変換を行っている。  
その際に活性化関数GERUに通す。この操作を4回行う。  
カーネルサイズ1の為、周辺のコンテキストは含んでいない。  
これはシーケンスの要素ごとの変換を線形、非線形を1セットにして計4回行っている。  
点単位畳み込み層と呼ぶらしい。ViTのハイブリッドモデルでも用いられている。  

コードで表すと以下のようになる。  
```python  
class ConvBackbone(nn.Module):  
    def __init__(self,input_ch,transformer_ch,):  
        self.input_proj = nn.Sequential(nn.Conv1d(input_ch, transformer_ch, 1), nn.GELU(),  
                                        nn.Conv1d(transformer_ch, transformer_ch, 1), nn.GELU(),  
                                        nn.Conv1d(transformer_ch, transformer_ch, 1), nn.GELU(),  
                                        nn.Conv1d(transformer_ch, transformer_ch, 1), nn.GELU())  

    def forward(self,x):  
        x = self.input_proj(x)  
        return x  
```  



ハイパーパラメータ探索を行わない単純なモデルのコードを以下に実装した。  

https://github.com/rakawanegan/HumanActivityRecognition/blob/master/convbackbone_transformer.py  

また、Optunaでハイパーパラメータ探索を行ったコードを以下に実装した。  

https://github.com/rakawanegan/HumanActivityRecognition/blob/master/optuna_convbackbone_transformer.py  

どちらのコードも依存ローカルモジュールは存在しないため、以下のCSVファイルをダウンロードすることでローカルで再現可能である。  

https://github.com/rakawanegan/HumanActivityRecognition/blob/master/data/WISDM_ar_v1.1.csv  


実装結果について。  
今回Optunaの探索区間・条件を以下のように設定した。  

- Epoch=10の場合におけるACCスコアが最も高いハイパーパラメータを探す  
- それぞれ離散的な値を設定し、場合の数は72000通りで探索回数は1000回  

| 探索区間     | hidden_ch    | depth    | heads    | mlp_dim   | dropout    | emb_dropout | batch_size |  
|--------------|--------------|----------|----------|------------|------------|-------------|------------|  
| Min          | 3            | 3        | 3        | 256        | 0.01       | 0.01        | 16         |  
| Max          | 15           | 8        | 10       | 2048       | 0.8        | 0.8         | 512        |  


探索結果、最良のACCは0.9187573356807514であり、ハイパーパラメータは以下の様になった。  
| パラメータ     | hidden_ch    | depth    | heads    | mlp_dim   | dropout    | emb_dropout | batch_size |  
|--------------|--------------|----------|----------|------------|------------|-------------|------------|  
| 値             | 15           | 3        | 3        | 1024       | 0.01       | 0.01        | 64         |  

また、探索の詳細（0.90以上のスコアのパラメータに絞込）については以下から参照できる。  

https://github.com/rakawanegan/HumanActivityRecognition/blob/master/results/preconvtransformer_y.log_pcs.csv  

Optunaによって得られたハイパーパラメータを元に再度スクラッチから学習させた。  
結果、37EPOCHでEarly Stoppingし、ACC=0.91でローカルでの最良スコア0.89を更新した。  

今後の課題・展望としては、  

- dimを変更する埋め込み層の実装  
- Optunaの探索条件を実際の学習と同じものにする（Early Stopping等）  
- 学習計画（Warmup等）の実装  
- データ拡張の実装  
- コードのリファクタリング  

を考えている。  
