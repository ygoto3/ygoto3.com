title: フロントエンドエンジニアのための動画ストリーミング技術基礎 
categories:
  - Video
date: 2016-09-29 16:22:53
tags:
  - HLS
    MPEG-DASH
    MSE
    video
---

[AbemaTV](https://abema.tv/) という動画サービスをリリースしてから半年経ち、新しくサービスのフロントエンドに関わる人数が少し増えてきたため、動画に関して社内で勉強会を行いました。本記事はその勉強会資料です。

## Web でメディアを見るためにはデータのダウンロードが必要

![Download via HTTP](/images/streaming-technology-basics-for-frontend-engineers/hls.png)

Web サービスが HTML を介して提供するコンテンツはテキスト、画像、音声、動画などいろいろありますが、テキスト以外のデータは HTML にインラインで返したりせず、基本的には外部ファイルとして非同期に取得されることがほとんどだと思います。

### 画像の場合

HTML 内の `img` 要素の `src` 属性に表示したい画像ファイルのパスを指定することで、Web ブラウザはその画像をリクエストし、ダウンロードしたデータをデコードして画像として表示します。

```html
<img src="sample.jpg" />
```

![https://placekitten.com/g/300/300](https://placekitten.com/g/300/300)

### 動画の場合

動画の場合も同じです。`video` 要素を使って `img` 要素と同様に `src` 属性に動画ファイルのパスを指定します。

```html
<video src="sample.mp4"></video>
```

### 動画はデータ容量が大きい

画像と違い、動画コンテンツはデータ容量がとても大きいため、データをダウンロードして再生するまでに待ち時間が発生します。

![動画のダウンロード](/images/streaming-technology-basics-for-frontend-engineers/video-dl.png)

動画のデータ容量が大きい理由はとても単純で、動画は画像データが集合したものだからです。静止画像を人間の目が滑らかに感じられる速さで切り替えて表示することで絵を動かすという表現を実現しています（よくパラパラマンガに例えられますが、そんな感じです）。この人間の目が滑らかに感じる速さというのが 1 秒間に 30 枚だったり 24 枚を切り替えることになります。29.97 (≒30) fps とか 24 fps とかの数字を耳にしたことがあるかと思いますが、24 fps の場合は 1 秒間（s）の間（p）に 24 フレーム（f）を切り替えることを意味します。

データを全て自分の端末にダウンロードしてから再生しようとすると、かなり長い待ち時間が発生してしまいます。もし 2 時間の映画を見ようと思ったら 172,800 (= 24 フレーム * 60 秒 * 60 分 * 2 時間) 枚の画像をダウンロードするのを待つことになります。しかも動画を構成する要素は画像だけではなく、音声データも含まれるため、純粋な情報量としてはそれ以上になります。

## ストリーミング

動画データを全てダウンロードしてから再生するのではなく、ダウンロードしたデータで再生できる部分から再生を始め、同時に残りのデータをダウンロードしていく方式を、ストリーミング再生といいます。長時間の動画でもダウンロードしながら再生することができるので、再生するまでの待ち時間を短かくすることができます。

また、ストリーミングでは動画を途中から再生することも可能にします。2 時間映画のたとえば 1 時間経ったあたりから見たいとき、1 時間経過した部分からデータをダウンロードし始め、再生を始めることができます。

![シーク](/images/streaming-technology-basics-for-frontend-engineers/seek.png)

## AbemaTV で使用しているストリーミングプロトコル

ストリーミング再生は、映像を配信する側と映像を再生する側で、データをどのような手順で通信するかをあらかじめ決めて、その手順通りに両者がデータを処理することによって実現します。その通信手順のことをストリーミングプロトコルと呼びます。ここでは AbemaTV で使用しているはストリーミングプロトコルを 2 つ説明します。

### HTTP Live Streaming

HTTP Live Streaming はアップル社が自社プロダクトである QuickTime、OS X、iOS、Safari 向けに開発したストリーミングプロトコルです。略して HLS と呼ばれるので、この記事でも HLS と表記します。その名前の通り、通信は HTTP で行われます。専用のプロトコルが必要ないため、通常の Web サーバーを用意するだけで配信ができてしまいます。

![HLS](/images/streaming-technology-basics-for-frontend-engineers/hls.png)

HLS を配信するために必要なファイルは、動画を数秒ごとの「MPEG-2 TS」形式のファイルに分割したセグメントファイル、それらをどの順番で再生するかを記したプレイリストだけです。

![m3u8 ファイルと ts ファイル](/images/streaming-technology-basics-for-frontend-engineers/m3u8-ts.png)

#### 簡単な HLS の配信を試してみる

まず、プレイリストとセグメントファイルを作成します。ここでは ffmpeg というツールを使い、 `input.mp4` というファイル名で保存されている動画から `output.m3u8` というプレイリストと分割されたセグメントファイルを作成します。

```sh
$ ffmpeg -i input.mp4 \
  -vcodec libx264 \
  -s 1280x720 \
  -acodec aac -b:a 256k\
  -flags +loop-global_header \
  -bsf h264_mp4toannexb \
  -f segment -segment_format mpegts \
  -segment_time 10 \
  -segment_list output.m3u8 output_%04d.ts
```

すると、 `output.m3u8` と `output_****.ts` というファイルが作成されます。 `output.m3u8` の内容は下記のようになります。

```
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-ALLOW-CACHE:YES
#EXT-X-TARGETDURATION:18
#EXTINF:10.500000,
output_0000.ts
#EXTINF:12.625000,
output_0001.ts
#EXTINF:10.416667,
output_0002.ts
#EXTINF:10.416667,
output_0003.ts
...略
output_0058.ts
#EXTINF:5.125000,
output_0059.ts
#EXT-X-ENDLIST
```

Web サーバーを起動します。ここでは Mac OS X にプリインストールされている Python2 を使用します。

```sh
$ python -m SimpleHTTPServer
```

Python2 の SimpleHTTPServer モジュールはデフォルトで `8000` 番ポートを使用するので、Safari で [http://localhost:8000/output.m3u8](http://localhost:8000/output.m3u8) にアクセスします。

![Safari で HLS を再生](/images/streaming-technology-basics-for-frontend-engineers/hls-playing.png)

ここでアクセスする Web ブラウザに Safari を指定しているのは、Safari 以外のメジャーブラウザでは HLS をネイティブサポートしていないためです。Safari 以外のブラウザで HLS を再生するには、Flash などのプラグインを使用するか、後述する Media Source API を使用して、JavaScript で追加実装する必要があります。HLS はアップル社が開発したということもあり、Safari だけは m3u8 をロードしてそのまま再生することができます。

### MPEG-DASH

MPEG-DASH は HLS と同様に通信に HTTP を使用したストリーミングプロトコルです。DASH は Dynamic Adaptive Streaming over HTTP の略です。Apple 社が開発した HLS のほかに Microsoft 社が開発した Smooth Streaming や Adobe が開発した HTTP Dynamic Streaming など HTTP ベースのストリーミングプロトコルがいくつかありますが、残念ながら各々互換性がありません。MPEG-DASH は ISO 国際標準規格 (ISO/IEC 23001-6) としてリリースされています。

MPEG-DASH も HLS 同様、通常の Web サーバーと動画のセグメントファイルとプレイリストを用意するだけで配信ができてしまいます。MPEG-DASH ではセグメントファイルは fragmented mp4 もしくは ts 形式、プレイリストは MPD（Media Presentation Description）と呼ばれる XML で記述されたファイルを用意します。

![MPD](/images/streaming-technology-basics-for-frontend-engineers/mpd.png)

#### 簡単な MPEG-DASH の配信を試してみる

MPEG-DASH 用のセグメントファイルとプレイリストを用意します。今回はセグメントファイルは fragmented mp4 を使用することにします。まず、ffmpeg を使って動画を fragmented mp4 で映像の圧縮に使う「H.264/AVC」と音声の圧縮に使う「AAC」というコーデックでリエンコードします。コーデックについては後述します。

```sh
$ ffmpeg -i ./input.mp4 \
  -vcodec libx264 \
  -vb 500k \
  -r 30 \
  -x264opts no-scenecut \
  -g 15 \
  -acodec aac \
  -ac 2 \
  -ab 128k \
  -frag_duration 5000000 \
  -movflags frag_keyframe+empty_moov \
  ./encoded.mp4
```

次に MP4Box というツールを使って、動画を分割してセグメントファイルとプレイリストを作成します。

```sh
$ MP4Box -frag 4000 \
  -dash 4000 \
  -rap \
  -segment-name sample \
  -out ./output.mp4 \
  ./encoded.mp4
```

プレイリスト `output.mpd` と `output.m4s` と連番になったセグメントファイル郡が作成されます。 `output.mpd` は下のようになっています。

```xml
<?xml version="1.0"?>
<!-- MPD file Generated with GPAC version 0.6.1-revrelease  at 2016-09-29T12:57:43.136Z-->
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" minBufferTime="PT1.500S" type="static" mediaPresentationDuration="PT0H9M56.466S" maxSegmentDuration="PT0H0M4.000S" profiles="urn:mpeg:dash:profile:full:2011">
 <ProgramInformation moreInformationURL="http://gpac.sourceforge.net">
  <Title>./output.mpd generated by GPAC</Title>
 </ProgramInformation>

 <Period duration="PT0H9M56.466S">
  <AdaptationSet segmentAlignment="true" maxWidth="320" maxHeight="180" maxFrameRate="30" par="16:9" lang="und">
   <ContentComponent id="1" contentType="video" />
   <ContentComponent id="2" contentType="audio" />
   <Representation id="1" mimeType="video/mp4" codecs="avc3.640014,mp4a.40.2" width="320" height="180" frameRate="30" sar="1:1" audioSamplingRate="48000" startWithSAP="1" bandwidth="631708">
    <AudioChannelConfiguration schemeIdUri="urn:mpeg:dash:23003:3:audio_channel_configuration:2011" value="2"/>
    <SegmentList timescale="1000" duration="4000">
     <Initialization sourceURL="outputinit.mp4"/>
     <SegmentURL media="output1.m4s"/>
     <SegmentURL media="output2.m4s"/>
     <SegmentURL media="output3.m4s"/>
     ...略
     <SegmentURL media="output149.m4s"/>
     <SegmentURL media="output150.m4s"/>
    </SegmentList>
   </Representation>
  </AdaptationSet>
 </Period>
</MPD>
```

再び Web サーバーを起動します。

```sh
$ python -m SimpleHTTPServer
```

HLS とは違い、残念ながら MPEG-DASH をネイティブでサポートしている Web ブラウザはありません。後程 Media Source Extensions を説明するときに MPEG-DASH プレイヤーを作成するので、そこで確認したいと思います。

## HTML5 で扱うストリーミング

HLS と MPEG-DASH は HTML5 用の JavaScript API である Media Source Extensions を利用することで追加でプラグインをインストールすることなく、ストリーミング再生が可能です。

### Media Source Extensions

Media Source Extensions は MSE と呼ばれていますので、本記事でも MSE と表記します。MSE は W3C によって標準化されている HTTP ダウンロードを利用してストリーミング再生するために作られた JavaScript API です。

MSE で扱うメディアデータは、W3C で定められている仕様に従って、短い時間で区切ったデータ構造にセグメント化されている必要があります。MSE では、セグメントを 2 種類に分けて扱います。

* 初期化に必要なヘッダ情報である初期化セグメント
* 短い時間で区切られたメディアデータ本体が含まれるメディアセグメント

MSE は最初に初期化セグメント、その後にメディアセグメントを順番にソース・バッファに渡すと、そのメディアセグメントの順番で再生していきます。

### MSE で簡単な MPEG-DASH プレイヤーを作成してみる

ここでは `XMLHttpRequest` と `MediaSource` API を使用して簡単な MPEG-DASH プレイヤーを作成して、先程 ffmpeg と MP4Box で作った MPEG-DASH コンテンツを再生してみます。

最初に `id` をつけた `video` 要素を用意します。

```
<video id="video"></video>
```

次に `XMLHttpRequest` で MPD を取得します。MPD は XML ファイルなので、パースして `Representation` 要素から MIME タイプやコーデックの情報を取得しておきます。

```javascript
var type, mpd;

var xhr = new XMLHttpRequest();
xhr.open("GET", "http://localhost:8000/output.mpd", true);
xhr.responseType = "document";
xhr.overrideMimeType("text/xml");
xhr.onload = e => {
  const mpd = xhr.responseXML;
  const representation = mpd.getElementsByTagName("Representation")[0];
  const mimeType = representation.getAttribute("mimeType");
  const codecs = representation.getAttribute("codecs");
  type = `${mimeType}; codecs="${codecs}"`
  mpd = mpd;
  initializeVideo(); // 次の関数へ
};
xhr.send(null);
```

次に `MediaSource` API で最初に用意した `video` 要素を拡張し、ソースとしてダウンロードした動画のセグメントを追加できるようにします。

```javascript
var mediaSource;

function initializeVideo() {
  mediaSource = new MediaSource();
  const video = document.getElementById("video");

  mediaSource.addEventListener("sourceopen", initializeSourceBuffer, false); // mediaSource が開いたらソース・バッファを作成する
  video.src = URL.createObjectURL(mediaSource);
}
```

ソース・バッファを作成し、初期化情報が入ったセグメントとメディア本体のセグメントを追加できるように準備します。

```javascript
var sourceBuffer;

function initializeSourceBuffer() {
  sourceBuffer = mediaSource.addSourceBuffer(this.type);
  sourceBuffer.addEventListener("updateend", appendMediaSegment, false);
  appendInitializationSegment(); // 次の関数へ
}
```

先に取得した `mpd` から `Initialization` 要素の `sourceURL` の値を取得し、 `XMLHttpRequest` で取得します。セグメントファイルはバイナリデータなので、 `responseType` を`arraybuffer` に指定しておきます。

```javascript
function appendInitializationSegment() {
  const xhr = new XMLHttpRequest();
  const url = mpd.getElementsByTagName("Initialization")[0].getAttribute("sourceURL");
  xhr.open("GET", `http://localhost:8000/media/${url}`, true);
  xhr.responseType = "arraybuffer";
  xhr.onload = appendSegment;
  xhr.send(null);
}
```

そしてセグメントをロードしたタイミングでソース・バッファに追加します。

```javascript
function appendSegment(e) {
  sourceBuffer.appendBuffer(e.target.response);
}
```

初期化情報がバッファに追加されソースが更新されたら、続けてメディア本体のセグメントファイルを取得し、ソース・バッファに追加します。この処理をメディアセグメントの数だけ繰り返します。

```javascript
var segmentIndex = 0;

function appendMediaSegment() {
  const xhr = new XMLHttpRequest();
  const url = mpd.getElementsByTagName("SegmentURL")[segmentIndex++].getAttribute("media");
  xhr.open("GET", `http://localhost:8000/media/${url}`, true);
  xhr.responseType = "arraybuffer";
  xhr.onload = appendSegment;
  xhr.send(null);
}
```

## メディアセグメント - 動画とは何か

HLS や MPEG-DASH などのストリーミング配信では、セグメントファイルが実際の動画データになります。本記事の最初に書いた通り、動画データの容量は大きいです。ストリーミング配信の仕組みだけでは、動画の再生開始までの待ち時間は短かくすることはできても、再生を続けるために必要な１秒あたりのデータ量は減らすことはできません。ストリーミング再生では 1 秒あたりに必要なデータ量を少なくとも 1 秒以内に取得し続ける必要があります。でないと再生を継続できません。

同じ情報量を表現するデータの容量を小さくしたい場合、データに圧縮処理をかけます。圧縮のアルゴリズムはいくつもありますが、動画は映像と音声で構成されているため、映像の圧縮に適したアルゴリズムと音声の圧縮に適したアルゴリズムは異なることを考慮する必要があります。映像圧縮に適したアルゴリズムで処理した映像ファイルと音声圧縮に適したアルゴリズムで処理した音声ファイルを１つのファイルとしてまとめたものが動画ファイルです。

## コンテナとコーデック

動画ファイルは映像ファイルと音声ファイルをまとめたものと説明しましたが、このまとめ方の形式のことをコンテナフォーマットといいます。また、映像データや音声データを圧縮するアルゴリズムのことをコーデックといいます。

![コンテナとコーデック](/images/streaming-technology-basics-for-frontend-engineers/containers-codecs.png)

### コンテナ

コンテナフォーマットは一般的にコンテナと略します。コンテナと呼ぶと難しそうですが、コンテナはファイルフォーマットの１種なので、私たちが普段動画ファイルとして意識している単位と一致します。ファイルフォーマットとはファイルの保存形式のことです。以下にリストしたものが代表的なコンテナですが、聞いたことがある名前が多いと思います。

* AVI
* MP4
* MOV
* MPEG
* MKV
* WMV
* FLV
* ASF

コンテナは映像と音声データがどのように格納されるのかを定義しています。また動画は映像と音声を同時に再生する必要があるため、両者の同期を取るための情報もコンテナが格納しています。ほかにも動画タイトルや説明などのメタ情報、字幕などの情報もコンテナが格納されている場合があります。

コンテナは対応しているコーデックの映像と音声データのみ格納することができます。１つのコンテナがいくつかのコーデックに対応している場合も多々あるので、コンテナの種類が分かっても格納されているコーデックの種類は分かりません。そのため、動画プレイヤーが同じコンテナで保存された２つの動画ファイルのうち、片方だけ再生できるということもあります。

たとえば Flash 動画のコンテナである FLV は映像コーデックとして「Sorenson Spark」と「H.264/AVC」を格納できます。もし動画プレイヤーが「Sorenson Spark」には対応していても 「H.264/AVC」には対応していなかった場合、「Sorenson Spark」を格納している FLV ファイルは再生できても、「H.264/AVC」を格納している FLV ファイルはコーデックエラーが発生して再生できません。

### コーデック

映像や音声は圧縮する必要があります。特にストリーミング再生などのデータ通信と再生を同時に行うような場合は必須です。コーデックはその圧縮のアルゴリズムです。

なぜ映像を圧縮する必要があると言うと、映像はたくさんの静止画をパラパラマンガのようにめくって人間の目に物体や背景が動いているように見せているので、このたくさんの静止画は情報量として膨大なのです。

映像を構成する画像データはラスタという色のついたピクセルの集合で表現します。1 ピクセルの情報量は 24 bit で表現できます（24bit フルカラーの場合、R -赤- G -緑- B -青- の各色成分につき 256 段階の指定ができるため、1 ピクセルは `Math.log2(256 * 256 * 256) = 24 bit` の情報量が必要）。

そうすると例えば、フル HD の 1 フレームを構成する 1920 * 1080 ピクセルの情報量は 49,766,400 (= 24 * 1920 * 1080) bit になります。これはまだ 1 フレームなので、24 fps の動画の場合、1 秒間に 1,194,393,600 (= 49,766,400 * 24) bit が必要になります。

これは 1 秒間に 1,194 Mbit のデータを通信を介して取得する必要があるということになります。しかし、例えば受信実効速度が 76.6Mbps と記載されているソフトバンク提供の超高速データ通信サービス [SoftBank 4G LTE](http://www.softbank.jp/mobile/network/explanation/4glte/) でデータ通信をした場合でも、 1 秒間に取得できるデータ量は 76.6 Mbit なので、先程の 1,194 Mbit に遠く及びません。

しかし、この 1,194 Mbit の映像データは「H.264/AVC」というコーデックで圧縮した場合、典型的な圧縮率としては 1/100 のデータ量に圧縮することができます。すると 12 Mbit 程度になるので、76.6 Mbps のデータ通信速度でも視聴が可能になります。

この「H.264/AVC」は AbemaTV でも映像コーデックとして使用していますが、映像コーデックにはほかにも以下のような種類があります。

* H.265
* VP8
* VP9
* MPEG-4
* WMV9

ここでは映像コーデックしか取り上げませんが、音声コーデックは代表的なものに「AAC」や「MP3」があり、AbemaTV では「AAC」を使用しています。

コーデックはデータ量を圧縮するものですが、ただデータ量を減らせればいいのではなく、人間が知覚できる範囲の画質や音質を落とすことなく圧縮しなくてはいけません。なので、選択するコーデックが悪いと画質や音質を落とすことになります。

## AbemaTV で使用しているコンテナ MPEG-2 TS

「MPEG-2 TS」は MPEG-2 システムのうち放送・通信用のコンテナです。地上波デジタル放送でも使用されているコンテナですが、HLS でも「MPEG-2 TS」を使用します。DevTools の Network パネルを開いた状態で AbemaTV の動画を視聴しているとたくさんの `**.ts` という拡張子のデータがリクエストされるのが確認できます。これが「MPEG-2 TS」のファイルです。

「MPEG-2 TS」は放送・通信用に作られたコンテナのため、通信途中でデータが途切れたとしてもちゃんと再生できるように設計されています。「MPEG-2 TS」では動画を 184 バイト単位のデータに分割し、それに 4 バイトの TS ヘッダと呼ばれるデータを付加して計 188 バイト固定長のパケットを連続で転送することでデータ伝送を行います。4 バイトの TS ヘッダのうち最後の 4bit は巡回カウンターと呼ばれるデータを持っていて、これがパケットごとに 1 ずつカウンターするため、これを検査することでパケットの欠落がないかを確認できるようになっています。

![TS パケット](/images/streaming-technology-basics-for-frontend-engineers/ts-packet.png)

MPEG-2 システムには蓄積メディア用のコンテナとして別に「MPEG-2 PS」がありますが、こちらはデータが連続していることが前提なので、ランダムアクセスなどに優れた設計になっています。

## AbemaTV で使っている映像コーデック H.264/AVC

AbemaTV では「MPEG-2 TS」コンテナに「H.264/AVC」コーデックで圧縮した映像データを格納しています。「H.264/AVC」は正式名称を「H.264」もしくは「MPEG-4 Part 10 Advanced Video Coding」といいます。（正式名称が２つあるのは ITU-T と ISO/IEC という２つの組織が共同で策定したものをそれぞれの名称をつけているだけです。）「MPEG-4」という名前が付けられている通り、その圧縮アルゴリズムの原理は、従来方式の「MPEG-1」、「MPEG-2」を継承しています。ここでは「MPEG」の圧縮アルゴリズムの原理を学んでいきます。

### 圧縮の基本

データを圧縮する基本は

* 出現するデータパターンに偏りを持たせること
* 出現頻度が高いパターンを短く表現すること

です。単純な例で見ていきます。

#### 出現頻度が高いパターンを短く表現する

たとえば、文字 a-d があったとき、それらを識別する符号を下記のように表現できます。

文字 | 符号 |
--- | ---
a | 00
b | 01
c | 10
d | 11

文字列「bbabcbdbaacba」は「01 01 00 01 10 01 11 01 00 00 10 01 00」という符号で表現されます。この文字列を表現するのに必要なデータ量は 26(=2*13)bit です。この文字列にて、各々の文字の出現回数は均一ではありません。

文字 | 出現回数 | 出現率
--- | --- | ---
a | 4 | 0.31
b | 7 | 0.54
c | 2 | 0.15
d | 1 | 0.08

そこで出現回数が 1 番多い b に 1 番短い符号、2 番目に多い a に次に短い符号を割り当ててみます。

文字 | 符号 |
--- | ---
a | 10
b | 0
c | 110
d | 111

すると先程の文字列「bbabcbdbaacba」は「0 0 10 0 110 0 111 0 10 10 110 0 10」と表現されますが、データ量が 23bit に減りました。このように、データの出現頻度が均一ではなく偏りがあると、異なる長さの符号を割り当てることによりデータ量を圧縮することができます。

このように可変長の符号を出現頻度に応じて割り当てることエントロピー符号といいますが、その割り当てパターンを作成する方法の 1 つに**ハフマン符号**があります。

![ハフマン符号](/images/streaming-technology-basics-for-frontend-engineers/huffman.png)

この図のように出現確立が高いものからツリー上に符号を割り当てていきます。これにて全ての文字が一意かつ瞬時に解読できる少ないデータ量の符号を作成することができます。

#### 出現するデータパターンに偏りを持たせる

一見出現率に偏りがない場合でも情報の表現方法を変えることでデータの出現頻度に偏りを持たせることができます。

たとえば、「1 2 3 2 1 0 -1 -2」のような数列はそのままだと下記のような出現回数ですが、

数字 | 出現回数 | 出現率
--- | --- | ---
-2 | 1 | 0.125
-1 | 1 | 0.125
0 | 1 | 0.125
1 | 2 | 0.25
2 | 2 | 0.25
3 | 1 | 0.125

これを前の数字との差分として表現すると「1 2 3 2 1 0 -1 -2」→「0 +1 +1 -1 -1 -1 -1 -1」となり、データの出現頻度に大きな偏りを作ることができました。

差分 | 出現回数 | 出現率
--- | --- | ---
0 | 1 | 0.125
+1 | 2 | 0.25
-1 | 5 | 0.625

これをハフマン符号することでデータを圧縮することができます。

### MPEG の圧縮

文字列や数列データの圧縮の例について見てきましたが、動画圧縮の場合も基本的な考え方は同様です。しかし、MPEG の場合は動画特有の性質を利用して圧縮率を高める工夫をしています。

MPEG の圧縮アルゴリズムは静止画の圧縮と映像の圧縮で構成されています。

* 静止画自体のデータサイズを圧縮する
* 連続する映像フレームのデータの差分だけを記録する

### 静止画の圧縮

MPEG の静止画の圧縮アルゴリズムの基礎は画像圧縮規格である「JPEG」です。`**.jpg` の拡張子で馴染のアレです。

静止画の圧縮アルゴリズムは簡単に以下のようなことを行います。

* 画像は隣り合うピクセルが似ているという特徴を利用して差分情報だけで表現する
* 人間の目が変化に鈍感な情報を省略する
* エントロピー符号する

#### 画像は隣り合うピクセルが似ている

たとえば空の写真を撮影した場合、その画像を構成するピクセルの多くは空の青色と雲の白色の微妙な色味の変化になると思います。空ほど色数が少なくない写真や絵の場合でも、基本的に画像は色が段階的にしか変化していないピクセルの方が出現頻度が圧倒的に多く、急な変化の頻度は少ないはずです。

![空](/images/streaming-technology-basics-for-frontend-engineers/xsky_00001.jpg)

この画像の性質を利用して、画像データの出現頻度に偏りを作って符号化することを **DPCM 符号化**といいます。

#### 人間の目が変化に鈍感な情報を省略する

MPEG は**離散コサイン変換**という演算を行うことで、人間の目にあまり目立たない細かい情報をデータから取り除いてしまうことで圧縮率を上げています。離散コサイン変換は英語では Discrete Cosine Transform というので DCT と略されます。

DCT では画像を波形として扱い、フーリエ変換のように周波数ごとの波の強度で画像を表現します。ここで高い周波数の波は人間の目にあまり目立たない情報となるので、省略してしまうことで画質への影響を最小限に抑えながら圧縮率を高めることが可能になります。

![DCT](/images/streaming-technology-basics-for-frontend-engineers/DCT.png)

#### エントロピー符号

ここまで静止画の圧縮について、DPCM 符号化と DCT の処理を見てきましたが、これらで求められた値をエントロピー符号することで更に圧縮効率を高めます。

### 映像の圧縮

静止画の圧縮では、映像における 1 枚 1 枚のフレームのデータ量を削減しました。映像の圧縮では、時間の流れを利用してデータの圧縮率を高める工夫をしています。

#### 画素の省略

MPEG では画素情報を RGB ではなく、**輝度信号（Y）**、 **色差信号（Cr）（Cb）** で表現します。RGBの各成分、輝度信号（Y）、色差信号（Cr）（Cb）の関係は下記です。

```
Y = 0.299*R + 0.587*G + 0.114*B
Cr = 0.500*R - 0.419*G - 0.081*B
Cb = -0.169*R - 0.332*G + 0.500*B
```

人間の目は明るさの変化に対しての方が色の変化に対してより敏感です。MPEG ではその人間の視覚の癖を利用し、フレームごとに Cr と Cb 信号を画素の情報から省いています。Cr と Cb が少々省かれたとしても 明るさの情報である Y が省かれていなければ、人はそれ程違和感を感じないのです。

![CrCb の省略](/images/streaming-technology-basics-for-frontend-engineers/y-cr-cb.png)

#### フレーム間予測

静止画の場合と似ていますが、映像の場合も時間的に隣合うフレームが持つ画像は似ているはずです。MPEG はその映像の特徴を利用して、映像のフレームにその画像を表示するための全ての情報を持たせません。MPEG には 3 種類のフレームがあります。

* I ピクチャ
* P ピクチャ
* B ピクチャ

**I ピクチャ**を除いて、他のフレームが持ってる情報と自身が持ってる情報を合わせて画像を表示することができるようになります。この 3 種類はそれぞれ役割りが違います。**I ピクチャ**は画像を表示するための全ての情報を持っています。**P ピクチャ**は過去に表示した**I ピクチャ**もしくは**P ピクチャ**が持っていたデータとその差分データを使用して画像を表示します。**B ピクチャ**は過去だけではなく未来の**I ピクチャ**もしくは**P ピクチャ**が持っているデータを利用することでより圧縮率を高めます。

![I/P/B ピクチャ](/images/streaming-technology-basics-for-frontend-engineers/ipb-pictures.png)

## まとめ

動画は昔からある技術分野ですが、Web のフロントエンドエンジニアだった自分には足を踏み込んだら分からないことだらけの難しい分野だと感じました。しかし、最近はストリーミング関連の技術も進み、Web においても動画を扱った事業に関わることが増えてきています。本記事は社内勉強会向けですが、フロントエンドエンジニア視点から動画を学んでいくスタートポイントになればと思います。

## 参考

**HTTP Live Streaming**

[https://en.wikipedia.org/wiki/HTTP_Live_Streaming](https://en.wikipedia.org/wiki/HTTP_Live_Streaming)

**H.264**

[https://ja.wikipedia.org/wiki/H.264](https://ja.wikipedia.org/wiki/H.264)

**MPEG-2システム**

[https://ja.wikipedia.org/wiki/MPEG-2%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0](https://ja.wikipedia.org/wiki/MPEG-2%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0)

**HLSとは**

[http://qiita.com/STomohiko/items/eb223a9cb6325d7d42d9](http://qiita.com/STomohiko/items/eb223a9cb6325d7d42d9)

**ffmpeg で mp4 をiPhone用のストリーミング（HLS）に対応させる。**

[http://takuya-1st.hatenablog.jp/entry/2016/04/06/034906](http://takuya-1st.hatenablog.jp/entry/2016/04/06/034906)

**MPEG DASHを知る**

[http://qiita.com/gabby-gred/items/c1a3dbe026f83dd7e1ff](http://qiita.com/gabby-gred/items/c1a3dbe026f83dd7e1ff)

**MPEG-DASH content generation with MP4Box and x264**

[https://bitmovin.com/mp4box-dash-content-generation-x264/](https://bitmovin.com/mp4box-dash-content-generation-x264/)

**Media Source Extensionsを使ってみた (MP4編)**
[http://qiita.com/tomoyukilabs/items/54bd151aba7d3edf8946](http://qiita.com/tomoyukilabs/items/54bd151aba7d3edf8946)

**動画・音声の規格について ~コーデック・コンテナ~**

[http://michisugara.jp/archives/2011/video_and_audio.html](http://michisugara.jp/archives/2011/video_and_audio.html)

**VIDEO-ITを取り巻く市場と技術**

[http://www.mpeg.co.jp/libraries/video_it/index.html](http://www.mpeg.co.jp/libraries/video_it/index.html)

**動画形式の種類と違い（AVI･MP4･MOV･MPEG･MKV･WMV･FLV･ASF等）【コンテナ】**

[http://aviutl.info/dougakeisiki-konntena/](http://aviutl.info/dougakeisiki-konntena/)

**【動画が再生できない!?】そんなときに必ず役立つ5つの知識**

[http://smarvee.com/column/can-not-play/](http://smarvee.com/column/can-not-play/)

**「映像がH.264/AVCでエンコードされたFLV」を「FLV5」と呼ぶのは間違い**

[http://goldenhige.cocolog-nifty.com/blog/2009/10/h264avcflvflv5-.html](http://goldenhige.cocolog-nifty.com/blog/2009/10/h264avcflvflv5-.html)

**量子化行列のナゾ～その１**

[http://www.nnet.ne.jp/~hi6/lab/quantize/](http://www.nnet.ne.jp/~hi6/lab/quantize/)

**モニタ解像度 図解チャート＆一覧 / monitor resolution data sheet&chart**

[http://www.quel.jp/etc/monitor-size/](http://www.quel.jp/etc/monitor-size/)
