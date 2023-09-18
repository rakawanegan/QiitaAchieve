---
title: ViTCaseStudy-HumanActivityRecognition
tags: Kaggle kotaniken Python ViT
author: NakagawaRen
slide: false
---
# データについて

## 概要
Human Activity RecognitionはデータサイエンスコンペティションプラットフォームKaggleで2017年に行われたコンペティションの名称である。  

ワイヤレスセンサデータから得られたx-y-z軸の3チャネルから成る時系列データを入力として用い、目的変数として人の状態「座る」「立つ」「歩く」「小走り」「昇る」「降りる」の6状態に分類する。  

以下の画像は「降りる」ラベルのデータの一例である。  
![download.png](image/28skEUldEG.png)  

次元としては80×3のデータであり、サンプル数は27454個である。  

データ分割は1/3をテストデータに用いた。  
つまり、訓練データが18394個、検証データが9060個である。  

また、データのダウンロードは以下のリンクより可能である。  
https://www.kaggle.com/datasets/paulopinheiro/wireless-sensor-data  

## 前処理
今回は上記のサイトよりダウンロードしたデータのうち、「WISDM_ar_v1.1_raw.txt」を用いる。  

まず、前処理としてPandasでデータを読み込みができるようにdata_format.pyを実行する。  

data_format.py  
```python  
import csv  


def read_csv(filename):  
    with open(filename) as f:  
        e = f.read()  
    e = (  
        e.replace(",\n", ",,")  
        .replace("\n", "")  
        .replace(",;", ",")  
        .replace(";", ",")  
        .split(",")  
    )  
    return e  


def write_csv(filename, data):  
    with open(filename, "w") as f:  
        writer = csv.writer(f)  
        for idx in range(len(data) // 6):  
            writer.writerow(data[6 * idx : 6 * idx + 6])  


if __name__ == "__main__":  
    data = read_csv("./data/WISDM_ar_v1.1_raw.txt")  
    # print(data)
    write_csv("./data/WISDM_ar_v1.1.csv", data)  

```  

data_format.pyを実行することによりdataディレクトリに「WISDM_ar_v1.1.csv」が生成される。  

これで実行環境は整った。  

# 1D-CNN（比較対象）

比較対象例として以下の1D-CNNがある。  

https://www.kaggle.com/code/paulopinheiro/human-activity-recognition-1d-cnn  

元ネタはおそらくこれ（未読）  
https://arxiv.org/pdf/1809.08113.pdf  

## モデルのパラメータ

合計パラメータ数：346,566  
| Layer (type)              | Output Shape | Param #  |  
|---------------------------|--------------|----------|  
| conv1d_1 (Conv1D)         | (None, 69, 160)  | 5,920  |  
| conv1d_2 (Conv1D)         | (None, 60, 128)  | 204,928  |  
| conv1d_3 (Conv1D)         | (None, 53, 96)   | 98,400  |  
| conv1d_4 (Conv1D)         | (None, 48, 64)   | 36,928  |  
| global_max_pooling1d_1 (GlobalMaxPooling1D) | (None, 64) | 0  |  
| dropout_1 (Dropout)       | (None, 64)  | 0  |  
| dense_1 (Dense)           | (None, 6)   | 390  |  

## 精度評価

|   tf-1DCNN  | Precision | Recall | F1-Score | Support |  
|-------------|-----------|--------|----------|---------|  
| Downstairs  |   0.86    |  0.83  |   0.85   |   824   |  
| Jogging     |   0.97    |  0.98  |   0.98   |  2832   |  
| Sitting     |   0.99    |  0.97  |   0.98   |   501   |  
| Standing    |   0.97    |  0.98  |   0.97   |   410   |  
| Upstairs    |   0.86    |  0.86  |   0.86   |  1025   |  
| Walking     |   0.98    |  0.98  |   0.98   |  3468   |  
|             |           |        |          |         |  
| Accuracy    |           |        |   0.95   |  9060   |  
| Macro Avg   |   0.94    |  0.93  |   0.94   |  9060   |  
| Weighted Avg|   0.95    |  0.95  |   0.95   |  9060   |  

![tf1dcnn_predict.png](image/xH86x6yud2.png)  

# 一次元におけるViTの処理

ViTにおいてデータの処理の流れは以下のようになる。  

1. 固定長80のスペクトルデータについてパッチサイズPに分割  

2. チャネルごとに結合し、80×3から(3×P) × (80/P)の系列データに変換  
（以後、N = 80 / Pとする）  

3. (3×P) × Nの系列データをD×Nの系列データに線形投影  

4. CLSトークンを加えD×(N+1)の系列データにする  

入力データはN+1個のTOKENに対応する。  


# 1D-ViT

ViTの条件としては、  
- 画像ViTと同じPre-Normalization  
- 一回でも評価が悪化した場合にはEarly Stoppingを実行  
- ハイパーパラメータ探索にはベイズ推定を用いたパラメータチューニングPython ModuleであるOptunaを用いた。  

Optunaの探索空間  
| Parameter | patch_size | dim   | depth | heads | mlp_dim | dropout | emb_dropout | batch_size |  
| --------- | ---------- | ----- | ----- | ----- | ------- | ------- | ----------- | ---------- |  
| MIN       | 1          | 10    | 5     | 3     | 256     | 0.0     | 0.0         | 8          |  
| MAX       | 40         | 1000  | 20    | 30    | 2048    | 0.8     | 0.8         | 256        |  


合計パラメータ数：936,930  

| Parameter     | patch_size | dim  | depth | heads | mlp_dim | dropout              | emb_dropout          | batch_size |  
|---------------|------------|------|-------|-------|---------|----------------------|----------------------|------------|  
| Value         | 10         | 117  | 7     | 3     | 409     | 0.02396243350698014  | 0.07984604757032415  | 256        |  


## 精度評価

|1D-ViT(optuna)| precision | recall | f1-score | support |  
|--------------|-----------|--------|----------|---------|  
| Downstairs   |   0.66    |  0.68  |   0.67   |   824   |  
| Jogging      |   0.96    |  0.96  |   0.96   |   2832  |  
| Sitting      |   0.97    |  0.85  |   0.91   |   501   |  
| Standing     |   0.86    |  0.96  |   0.91   |   410   |  
| Upstairs     |   0.73    |  0.71  |   0.72   |   1025  |  
| Walking      |   0.95    |  0.96  |   0.95   |   3468  |  
|              |           |        |          |         |  
| accuracy     |           |        |   0.90   |   9060  |  
| macro avg    |   0.86    |  0.85  |   0.85   |   9060  |  
| weighted avg | 0.90    |  0.90  |   0.90   |   9060  |  

![vit1d_predict.png](image/klKZSmUny1.png)  


# 問題点の考察
- サンプル数に対して自由度が大きすぎる？  

- データの値域を揃える必要がある？  

- パッチ処理する際にパッチ化、層正規化、線形射影、層正規化としているが良いのか？  

- Sitting-Standing、Up-Downの識別が弱い  

- Optunaの探索空間が広すぎる  

- Early Stoppingの条件が厳しすぎる  

# これからの展望
- 位置エンコーディングに学習可能な重みから三角波エンコーディングに変更する（自由度の削減：1,000程度の削減が可能）  

- 自己復元タスクなどTransformerに適したタスクで事前学習させてからFine-Turningをする  

- headが張り付いている理由の確認  

- ボトルネックの解消を目指した学習を考える（2値分類での解決）  

- 位置エンコーディングの可視化  

- dimはP×3の整数倍、Dropoutも離散的で良さそう。  
