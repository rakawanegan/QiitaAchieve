---
title: HumanActivityRecognitionSurvey-CaseStudy1
tags: kotaniken HumanActivityRecognition har ML
author: NakagawaRen
slide: false
---
# Human activity recognition based on time series analysis using U-Net
2018年に発表されたU-Netと呼ばれるCNNアーキテクチャを提唱した論文。  
データについて参考になった。  

今回ViTで取り組んでいるデータセットの名前は「WISDM dataset」と呼ばれるもののようだ。  
WISDMはWireless Sensor Data Miningの略らしい。  

3軸加速度時系列データ、つまり3チャンネルの時系列スペクトルデータである。  
20Hzでサンプリングされており、1098209サンプル存在する。  
合計時間にして915分のデータになる。  

教師ラベルは6種類存在し、内訳としては以下の通り。  
|  label   | Walking | Jugging | Upstars | Downstars | Sitting | Standing |  
|----------|---------|---------|---------|-----------|---------|----------|  
|percentage| 38.6%   | 31.2%   | 11.2%   | 9.1%      | 5.5%    | 4.4%     |  

提唱されているU-NetのACC、F1スコアは共に0.970であった。  
他にもデータセットには種類があることもわかったため、立ち戻って読むかもしれない。  

### 元論文
https://arxiv.org/pdf/1809.08113.pdf  
