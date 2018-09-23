初心者向きにR言語を用いたFIRフィルターの作成方法をまとめました。基本的な流れは[R言語を用いたIIRフィルターの作成](https://qiita.com/hitomecha/items/80d6e3309aa33941cb5e)とほぼ同じです。

## パッケージの準備
signal パッケージ[1]を使用し、FIRフィルターを作成します。まず、パッケージをインストールします。

```R
#signalパッケージのダウンロード
install.packages("signal")
#ライブラリーの読み込み
library("signal")
```

## 入力データの準備
入力データは10Hzと50Hzの正弦波の合成波を使用します。

```R
#入力データの作成（周波数が10Hzと50Hzのサイン波のデータの合成、サンプリング周波数：1kHz、１０秒間のデータ）
time <- seq(from=0, by=1/1000, length=10000) #時間軸
data <- sin(2*pi*10*time) + sin(2*pi*50*time)
#入力データの図示
plot(time, data, t="l", xlim=c(1, 1.5))
```

<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/ba610be0-f14a-761c-e13e-4656e1a3682c.png" width=50%>

上図より、2つの周波数の合成波になっています。また、周波数解析を行うと、

```R
data_fft <- fft(data)
data_psd <- abs(data_fft)^2
plot_fft <- seq(from=1/10000, by=1000/10000, to=500)
#入力データの周波数特性の図示
plot(plot_fft, data_psd[1:(length(data_psd)/2)], t="l", log="x", xlab="frequency (Hz)", ylab="PSD (1/Hz)")
```
<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/c415c59a-4cfa-117f-0958-da7c01fcb98d.png" width=50%>

ピークが2つ（10Hzと50Hz）になっていることがわかります。

## FIRフィルターの設計
今回はFIRフィルターを実装するために、signalパッケージのfir1()を使用します。5〜15Hzを通すバンドパスフィルターを実装します。

```R
#バンドパスフィルタの設計
#FIRフィルターの各パラメータ
fil_N <- 128 #フィルターの次数
fs <- 1000  #サンプリング周波数
fn <- fs/2  #ナイキスト周波数
fc <- c(5,15) #帯域通過周波数
fc_norm <- fc/fn  #周波数の正規化

fir_filter <- fir1(fil_N, fc_norm, type="pass")
```

fir1()では、帯域周波数をナイキスト周波数で割った正規化周波数を使います。typeは、pass, low, high, stopの指定により、様々な種類のフィルターを実装できます。

## フィルター特性の確認
```R
freqz(fir_filter, Fs=fs)  #フィルター特性の図示
```

<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/204628bc-a5d9-7a80-d73b-fc88484a268b.png" width=70%>

5〜15Hzのバンドパスフィルターを設計できました。


## 設計したフィルターを入力データに通す

```R
#設計したフィルターを入力データに適応
data_fil <- filtfilt(fir_filter, data)
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

<img src="https://qiita-image-store.s3.amazonaws.com/0/293436/b078c3d8-b1a3-6718-93ca-e9006517f10d.png" width=50%><img src="https://qiita-image-store.s3.amazonaws.com/0/293436/6007a42c-da36-bab5-65fc-1bb5dc442b49.png" width=50%>

フィルターにより50Hz成分がなくなりました。

### 参考文献
[1] Uwe Ligges et al. "Signal Processing", ver. 0.7-6, https://cran.r-project.org/web/packages/signal/signal.pdf, 参照July 7, 2018.