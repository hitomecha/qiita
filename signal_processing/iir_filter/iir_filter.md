---
title: R言語を用いたIIRフィルタの作成
tags: R
author: hitomecha
slide: false
---
初心者向きにR言語を用いたIIRフィルターの作成方法をまとめました。記事が冗長ですが、お許しください。
IIRフィルターは、ノイズ除去や特定の周波数帯域の信号抽出でよく用いられます。ローパスフィルターやバンドパスフィルター等として使うことができます。今回はR言語を用いて実装しました。


## パッケージの準備  
今回は signal パッケージ[1]を使用し、IIRフィルターを作成します。まず、パッケージをインストールします。

```R
#signalパッケージのダウンロード
install.packages('signal')
#ライブラリーの読み込み
library('signal')
```

入力データは10Hzと50Hzの正弦波の合成波を使用します。  

```R
#入力データの作成（周波数が10Hzと50Hzのサイン波のデータの合成、サンプリング周波数：1kHz、１０秒間のデータ）
time <- seq(from=0, by=1/1000, length=10000) #時間軸
data <- sin(2*pi*10*time) + sin(2*pi*50*time)
#入力データの図示
plot(time, data, t="l", xlim=c(1, 1.5))
```  

<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/afe3ee6c-4af8-342c-526d-6d58e9578bd7.png" width=50%>

上図より、2つの周波数の合成波になっています。また、周波数解析を行うと、

```R
data_fft <- fft(data)
data_psd <- abs(data_fft)^2
plot_fft <- seq(from=1/10000, by=1000/10000, to=500)

#入力データの周波数特性の図示
plot(plot_fft, data_psd[1:(length(data_psd)/2)], t="l", log="x", xlab="frequency (Hz)", ylab="PSD (1/Hz)")
```

<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/6d9d9659-b211-2713-0422-91217dedb448.png" width=50%>


ピークが2つ（10Hzと50Hz）になっていることがわかります。

## IIRフィルターの設計
今回はIIRフィルターの一種であるButterworth filterを実装するために、signalパッケージのbutter()を使用します。帯域通過周波数が5〜15Hzのバンドパスフィルターを実装します。

```R
#バンドパスフィルタの設計
#バターワースフィルターの各パラメータ
fil_N <- 2 #フィルターの次数
fs <- 1000  #サンプリング周波数
fn <- fs/2  #ナイキスト周波数
fc <- c(5,15) #帯域通過周波数
#fc <- 5 #ローパスフィルターやハイパスフィルターでは、周波数の指定は一つの値のみ
fc_norm <- fc/fn  #周波数の正規化

butterworth_filter <- butter(fil_N, fc_norm, type="pass", plane="z")
```

butter()では、帯域周波数をナイキスト周波数で割った正規化周波数を使います。typeは、pass, low, high, stopの指定により、様々な種類のフィルターを実装できます。

## フィルター特性の確認

```R
freqz(butterworth_filter, Fs=fs)  #フィルター特性の図示
```

<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/65ba523c-574b-32fe-6e08-6b5c002cf460.png" width=70%>


5〜15Hzのバンドパスフィルターを設計できました。

## 設計したフィルターを入力データに通す

```R
#設計したフィルターを入力データに適応
data_fil <- filtfilt(butterworth_filter, data)
```

フィルターによる位相遅れを防ぐためにfiltfilt()を使用し、順方向と逆方向の両方の処理を行うゼロ位相デジタルフィルター処理を実装します。

## フィルターを通したデータの確認

```R
#フィルターを通したデータの図示
plot(time, data_fil, t="l", xlim=c(1, 1.5))
#周波数特性の図示
data_fft <- fft(data_fil)
data_psd <- abs(data_fft)^2
plot(plot_fft, data_psd[1:(length(data_psd)/2)], t="l", log="x", xlab="frequency (Hz)", ylab="PSD (1/Hz)")
```

<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/fad753f7-de2f-f381-c76b-75e13c82c5a7.png" width=50%><img src="https://qiita-image-store.s3.amazonaws.com/0/293436/c609c7f9-ebd8-792d-af72-31424f6ce668.png" width=50%>


フィルターにより50Hz成分がなくなりました。

### 参考文献
[1] Uwe Ligges et al. "Signal Processing", ver. 0.7-6, https://cran.r-project.org/web/packages/signal/signal.pdf, 参照Sept 21, 2018.
