title: フロントエンドエンジニアのための生放送と RTMP 通信基礎
categories:
  - video
date: 2016-10-11 09:05:02
tags:
  - RTMP
    ActionScript
---

![生放送と RTMP 通信基礎](/images/live-streaming-and-rtmp-for-frontend-engineers/obs-sc.png)

前回「[フロントエンドエンジニアのための動画ストリーミング技術基礎](/posts/streaming-technology-basics-for-frontend-engineers/)」では HTTP ベースのストリーミング技術に関して勉強会を実施しました。視聴者に映像を届けるためのストリーミング技術に関してのお話でした。

本記事は、[AbemaTV](https://abema.tv/) の生放送番組で撮影機材から送られた映像がエンコーダーを介してリアルタイムに放送する部分について勉強会を実施した際の資料です。

## 生放送における動画データの通信

AbemaTV では生放送で撮影した動画データのやりとりに **RTMP** というプロトコルを利用しています。

![生放送の配信構成](/images/live-streaming-and-rtmp-for-frontend-engineers/live-streaming.png)

## RTMP とは

RTMP は **Real-Time Message Protocol** の略で、その名前の通りリアルタイムにコミュニケーションを行うためのプロトコルです。Web 業界では Photoshop などでお馴染の [Adobe Systems](http://www.adobe.com/) 社が開発しています。[Adobe Flash Player](http://www.adobe.com/software/flash/about/) がメディア配信サーバーとの間で音声や動画などのデータをやりとりするためのストリーミングのためのプロトコルとして開発されました。

[前回](/posts/streaming-technology-basics-for-frontend-engineers/)紹介した HLS や MPEG-DASH もストリーミングプロトコルでしたが、RTMP はこれらと異なり、**HTTP ベースではありません**。ですので、HLS や MPEG-DASH のように通常の Web サーバーでコンテンツを配信できるわけではなく、専用の RTMP サーバーが必要になります。

![RTMP 専用サーバー](/images/live-streaming-and-rtmp-for-frontend-engineers/http-and-rtmp.png)

## HTTP に対する優位性

HTTP における通信は必ずクライアントのリクエストから始まります。そのため、動画をストリーミングしようとする際、クライアントはサーバーに対して任意のインターバルで動画のセグメントをリクエストし続ける必要があります。HLS や MPEG-DASH におけるストリーミングはこの方式ですが、クライアントのタイミングで動画データをリクエストするため、本当の意味でのリアルタイム性はありません。動画データを生成しているサーバー側が送信したいタイミングでクライアントにデータをプッシュできる方が遅延が発生することがなく、効率的です。

それに対して、RTMP におけるデータ通信は持続的に接続した状態で双方向に行われます。そのため、サーバーがクライアントに送信したいタイミングでプッシュ送信することができ、遅延が発生しません。

また HTTP の場合、HTTP レスポンスヘッダーは冗長で一般的に数百バイトになります。返したいペイロードサイズに対してのオーバーヘッドを大きくしてしまっています。それに対し、RTMP パケットのヘッダーは固定長で 12 / 8 / 4 / 1 バイトのうちどれかになり、ペイロードサイズに対するオーバーヘッドは小さいです。

## 生放送の現場ではリアルタイム性を重視

![生放送の配信構成](/images/live-streaming-and-rtmp-for-frontend-engineers/live-streaming-wirecast.png)

AbemaTV の生放送の現場では撮影機材で撮れた映像を Wirecast などのエンコーダーでエンコードし、それを RTMP 通信で Wowza などのメディアストリーミングサーバーに届けています。そして、メディアストリーミングサーバーに届けられた映像を確認しながら、生で撮影しているその映像に対して遅延を極力少ない状態で CM 入りや視聴者参加型コンテンツなどのトリガーとなるシグナルを通信できる環境を構築しています。

## RTMP の種類

RTMP にはいくつかの派生種があります。

* **RTMPT** - HTTP でカプセル化した RTMP
* **RTMPS** - TLS/SSL で暗号化して HTTPS でカプセル化した RTMP
* **RTMPE** - こちらも暗号化された RTMP ですが、設計に欠陥があり RTMPS の使用が推奨されている
* **pRTMP** - Adobe Primetime DRM がかかった RTMP

RTMP は一般的に `1935` ポートを使用します。しかし、セキュリティの厳しい環境ではこの `1935` ポートが使えないこともしばしばあります。そのため、HTTP（ `80` ポート）や HTTPS（ `443` ポート）を装って通信するという手段を取ることが可能です。それが **RTMPT** と **RTMPS** になります。

## 更にリアルタイム性を重視したデータ通信

RTMP は **TCP** を利用したプロトコルですが、別に **RTMFP** という **UDP** を利用したプロトコルもあります。UDP を利用するプロトコルは TCP を利用するプロトコルと比べて通信速度面において利点があります。TCP はパケット・ロストに対して再送する仕組みですが、UDP はパケット・ロストに対して再送することはありません。その分再送のオーバーヘッドなく通信することができます。

## データ量の大きな双方向データ通信

RTMP は双方向のデータ通信が可能なプロトコルですが、両方向とも送信するデータ量が大きな通信サービスを構築する場合は UDP ベースの RTMFP が好まれます。たとえばテレビ会議やビデオチャットなどは双方が送信するデータが動画のため、データサイズが大きいにも関わらず、スムーズなコミュニケーションのためにリアルタイム性が求められます。

TCP で パケット・ロストによる再送で遅延の頻度が高まると、音声や映像が遅れた状態になる可能性が高くなります。特にテレビ会議やビデオチャットなどはデータの抜け落ちが発生したとしてもノイズ程度の劣化として許容できる場合がほとんどなため、UDP での通信が向いています。

## RTMP をサポートするメディアストリーミングサーバ

RTMP でストリーミング配信するには、専用の RTMP サーバーが必要です。ここでは RTMP をサポートする代表的なメディアストリーミングサーバーを３つ紹介します。

### Adobe Media Server

![Adobe Media Server](/images/live-streaming-and-rtmp-for-frontend-engineers/adobe-media-server.png)

Adobe Systems 社が開発しているメディアストリーミングサーバーです。Flash 技術の総本山である Adobe が開発しているだけあり、ここで紹介するメディアサーバーの中で一番知名度が高く、機能も豊富です。そしてその分ライセンス料も高いです。（RTMFP のサポート有無など機能数に応じて複数のエディションに別れています。）

参照：[Adobe Media Serverファミリー](http://www.adobe.com/jp/products/adobe-media-server-family/buying-guide-comparison.html)

### Wowza Media Server

![Wowza Media Systems](/images/live-streaming-and-rtmp-for-frontend-engineers/wowza.png)

[Wowza Media Systems](https://www.wowza.com/) 社によって開発されているメディアストリーミングサーバーです。元 Adobe Systems の社員がスピンアウトして立ち上げたこともあり、Adobe Media Server との互換性が高く、ほぼ同等の機能を持っています。それにも関わらずライセンス価格は Adobe Media Server と比較するとかなり安価なこともあり、AbemaTV でも生放送のストリーミングサーバーには Wowza Media Server を使用しています。

### Red5

![Red5](/images/live-streaming-and-rtmp-for-frontend-engineers/red5.png)

Java で実装されたオープンソースの RTMP プロトコルをサポートするメディアストリーミングサーバーです。Adobe Media Server にかなり似せて作られていて、単体のサーバーとしては同等の機能を提供してくれますが、クラスタリング構成にあまり対応していないため、大規模な配信サービスを構築する場合には注意が必要です。

## RTMP を使用するためのクライアントサイド

Web ブラウザは RTMP をネイティブでは対応していません。ブラウザ上で RTMP を使用するためには、プラグインとして Adobe Flash Player を使用する必要があります。

## 簡単な RTMP ストリーミング配信を実装してみる

まずは RTMP ストリーミングサーバーを構築します。先述したメディアサーバーを使用したいところですが、今回は単純なストリーミング機能のみを提供できれば良いので、NGINX をメディアストリーミングサーバーとして使うことができる [nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module) を使います。

### Docker で NGINX を立てる

nginx-rtmp-module を追加してコンパイルした NGINX の Docker イメージを作ります。ベースイメージには Alpine Linux を使います。ここでは RTMP 用に `1935` ポートと Flash アプリケーションを読み込むための HTML を返すために HTTP `80` ポートを開けるようにします。

#### Dockerfile を作成する

下記の Dockerfile では、コンパイルに必要なパッケージを `apk` でインストールして、任意のバージョンの NGINX と nginx-rtmp-module の Tarball をダウンロードし、コンパイルしています。 `./configure` のパラメータがやたら多いですが、必要ないモジュールを除外しているだけです。

```
FROM alpine:3.4
ENV NGINX_VERSION nginx-1.11.4
ENV NGINX_RTMP_MODULE_VERSION 1.1.7.10

ENV USER nginx
RUN adduser -s /sbin/nologin -D -H ${USER}

RUN apk --update --no-cache \
    add ca-certificates \
        build-base \
        openssl \
        openssl-dev \
        pcre-dev \
    && \
    update-ca-certificates && \
    rm -rf /var/cache/apk/*

RUN mkdir -p /tmp/build/nginx && \
    cd /tmp/build/nginx && \
    wget -O ${NGINX_VERSION}.tar.gz https://nginx.org/download/${NGINX_VERSION}.tar.gz && \
    tar -zxf ${NGINX_VERSION}.tar.gz

RUN mkdir -p /tmp/build/nginx-rtmp-module && \
    cd /tmp/build/nginx-rtmp-module && \
    wget -O nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION}.tar.gz https://github.com/sergey-dryabzhinsky/nginx-rtmp-module/archive/v${NGINX_RTMP_MODULE_VERSION}.tar.gz && \
    tar -zxf nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION}.tar.gz && \
    cd nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION} && \
    wget -O - https://raw.githubusercontent.com/gentoo/gentoo/6241ba18ca4a5e043a97ad11cf450c8d27b3079f/www-servers/nginx/files/rtmp-nginx-1.11.0.patch | patch

RUN cd /tmp/build/nginx/${NGINX_VERSION} && \
    ./configure \
      --sbin-path=/usr/local/sbin/nginx \
      --conf-path=/etc/nginx/nginx.conf \
      --error-log-path=/var/log/nginx/error.log \
      --pid-path=/var/run/nginx/nginx.pid \
      --lock-path=/var/lock/nginx/nginx.lock \
      --user=${USER} --group=${USER} \
      --http-log-path=/var/log/nginx/access.log \
      --http-client-body-temp-path=/tmp/nginx-client-body \
      --with-http_ssl_module \
      --with-http_gzip_static_module \
      --without-http_userid_module \
      --without-http_access_module \
      --without-http_auth_basic_module \
      --without-http_autoindex_module \
      --without-http_geo_module \
      --without-http_map_module \
      --without-http_split_clients_module \
      --without-http_referer_module \
      --without-http_proxy_module \
      --without-http_fastcgi_module \
      --without-http_uwsgi_module \
      --without-http_scgi_module \
      --without-http_memcached_module \
      --without-http_limit_conn_module \
      --without-http_limit_req_module \
      --without-http_empty_gif_module \
      --without-http_browser_module \
      --without-http_upstream_hash_module \
      --without-http_upstream_ip_hash_module \
      --without-http_upstream_least_conn_module \
      --without-http_upstream_keepalive_module \
      --without-http_upstream_zone_module \
      --without-http-cache \
      --without-mail_pop3_module \
      --without-mail_imap_module \
      --without-mail_smtp_module \
      --without-stream_limit_conn_module \
      --without-stream_access_module \
      --without-stream_upstream_hash_module \
      --without-stream_upstream_least_conn_module \
      --without-stream_upstream_zone_module \
      --with-threads \
      --with-ipv6 \
      --add-module=/tmp/build/nginx-rtmp-module/nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION} && \
    make -j $(getconf _NPROCESSORS_ONLN) && \
    make install && \
    mkdir /var/lock/nginx && \
    mkdir /tmp/nginx-client-body && \
    rm -rf /tmp/build

RUN apk del build-base openssl-dev && \
    rm -rf /var/cache/apk/*

RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log

COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY build /var/www/build

RUN chmod 444 /etc/nginx/nginx.conf && \
    chown ${USER}:${USER} /var/log/nginx /var/run/nginx /var/lock/nginx /tmp/nginx-client-body && \
    chmod -R 770 /var/log/nginx /var/run/nginx /var/lock/nginx /tmp/nginx-client-body

EXPOSE 80
EXPOSE 1935
CMD ["nginx"]
```

#### NGINX の configuration を設定する

イメージにコピーする `nginx.conf` は下記のように設定します。 `http` コンテキストに加えて `nginx-rtmp-module` で使用できるようになった `rtmp` コンテキストに設定を追加しています。

```
user nginx;
worker_processes 1;
daemon off;

events {
  ...
}

http {
  ...
}

rtmp {
  server {
    listen 1935;
    listen [::]:1935 ipv6only=on;

    application live {
      live on;
      record off;
    }
  }
}
```

この `rtmp` コンテキストの設定により、ローカルに Docker コンテナを立ち上げたとき `rtmp://localhost:1935/live` という URL で RTMP サーバに接続が可能になります。

### RTMP プレイヤーを実装する

次に RTMP プレイヤーとそれを表示する HTML を作成します。RTMP プレイヤーは Flash アプリケーションとして実装するので ActionScript で書きます。まず新規ファイル `Player.as` を作成します。

```sh
$ touch Player.as
```

`Player.as` に `Player` クラスを作成します。動画を表示するための `Video` オブジェクトも追加したいので、 `Sprite` クラスを継承しておきます。

```
package {

  import flash.display.Sprite;
  import flash.display.StageScaleMode;
  import flash.display.StageAlign;
  import flash.events.Event;
  import flash.events.NetStatusEvent;
  import flash.net.NetConnection;
  import flash.net.NetStream;
  import flash.media.Video;
  import flash.external.ExternalInterface;

  [SWF(backgroundColor="0x000000")]
  public class Player extends Sprite {

    private var nc: NetConnection;
    private var ns: NetStream;
    private var video: Video;
    
    function Player() {
    }

  }

}
```

次に `Stage` の設定します。Flash コンテンツを左上に整列する設定だけします。

```
function Player() {
  setupStage();
}

private function setupStage(): void {
  stage.scaleMode = StageScaleMode.NO_SCALE;
  stage.align = StageAlign.TOP_LEFT;
}
```

`NetConnection` クラスを使い、クライアントとサーバー間の双方向の接続を作成するための準備をします。 `NetConnection` オブジェクトのステータスが変化したタイミングで、メディアサーバーからのデータを再生できるように、 `NetStream` クラスを使ってストリームチャネルを開きます。開いた後ライブストリームを再生するために `ns.play()` メソッドを実行します。このときストリーム名として `"test"` を渡していますが、これは後程ライブストリームを作成する際にも使う名前になります。

```
function Player() {
  ...
  setupNetConnection();
}

private function setupNetConnection(): void {
  nc = new NetConnection();
  nc.addEventListener(NetStatusEvent.NET_STATUS, onChangeNCStatus);
}

private function onChangeNCStatus(e: NetStatusEvent): void {
  const code: String = e.info.code;
  if (code === "NetConnection.Connect.Success") {
    setupNetStream();
    ns.play("test");
  }
}
```

作成したストリームチャネルからの動画を表示するために `Video` オブジェクトに取り付けます。

```
private function setupNetStream(): void {
  ns = new NetStream(nc);
  ns.addEventListener(NetStatusEvent.NET_STATUS, onChangeNSStatus);

  video.attachNetStream(ns);
}
```

この `Video` オブジェクトもコンストラクト時に作成し、ステージに追加してきます。

```
function Player() {
  ...
  ssetupVideo();
}

private function setupVideo(): void {
  video = new Video(stage.width, stage.height);
  addChild(video);
}
```

`NetConnection` の準備が整ったので、最後にメディアサーバーに接続します。URL は先程の NGINX の RTMP メディアサーバーの `live` application に向けています。

```
function Player() {
  ...
  nc.connect("rtmp://localhost:1935/live");
}
```

### ActionScript のコンパイル

これで RTMP サーバーの実装はできたので、次は ActionScript をコンパイルします。コンパイルには Apache/Adobe Flex SDK の Node.js モジュール版である [node-flex-sdk](https://github.com/JamesMGreene/node-flex-sdk) を使用します。まずは NPM でインストールします。

```sh
$ npm i flex-sdk --save-dev
```

無事インストールできたら、`mxmlc` というコマンドを使って、 `Player.as` から `Player.swf` をコンパイルします。

```sh
$ $(npm bin)/mxmlc --output=Player.swf Player.as
```

### HTML の作成

作成された `Player.swf` を表示する HTML を作成します。

```
<object data="./Player.swf" type="application/x-shockwave-flash"></object>
```

### Docker コンテナの起動

Docker コンテナの構成に必要なファイルが揃ったので、これでイメージを作成します。

```sh
$ docker build -t rtmp .
```

イメージが作成できたか確認しましょう。

```sh
$ docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
rtmp                         latest              ************        About a minute ago          180.7 MB
```

無事に作成できたら、そのイメージから Docker コンテナを起動します。

```sh
$ docker run -p 1935:1935 -p 80:80 --name rtmp -t rtmp
```

Web ブラウザで [http://localhost/](http://localhost/) にアクセスしてみましょう。

![RTMP プレイヤー](/images/live-streaming-and-rtmp-for-frontend-engineers/browser-sc-black.png)

黒いボックスが表示されたと思います。このボックスが RTMP プレイヤーなのですが、今は配信するストリームが存在していないため、何も再生できず黒い状態です。ですので、次は再生するストリームを作成します。

### ストリームの作成

ここでは、[Open Broadcaster Software](https://obsproject.com/) （OBS）というオープンソースのライブストリーミング用のツールを使用してストリームを作成します。

OBS を起動して、「Settings」ボタンをクリックします。

![OBS](/images/live-streaming-and-rtmp-for-frontend-engineers/obs-sc-settings.png)

「Settings」ダイアログが表示されるので、左側のペインから「Stream」を選択します。
すると「URL」と「Stream key」を入力する画面に切り替わりますので、「URL」に NGINX の RTMP サーバーの `live` application の URL を入力し、「Stream key」には先程 RTMP プレイヤーを実装したときにストリーム名として指定した `test` を入力します。

![OBS Settings](/images/live-streaming-and-rtmp-for-frontend-engineers/obs-sc-stream.png)

「Settings」ダイアログで「OK」をクリックしたら、次に「 Sources」の「+」をクリックして適当なメディアソースを追加します。プルダウンメニューが表示されるので「Media Source」を選択して任意の動画ファイルを追加します。

メディアソースが追加されたら、「Start Streaming」ボタンをクリックしてストリーミングを開始します。

![OBS Sources](/images/live-streaming-and-rtmp-for-frontend-engineers/obs-sc-start.png)

ストリーミングが開始されたら、Web ブラウザに戻ります。すると OBS でストリームしている動画がブラウザの方でも再生されていることが確認できます。

![RTMP 再生](/images/live-streaming-and-rtmp-for-frontend-engineers/browser-sc.png)

## RTMP でメディアサーバーのメソッドを呼ぶ

ここまでで RTMP でサーバー側からプッシュされたデータをクライアントで再生する実装をしてきました。しかし、RTMP は双方向のデータ通信が可能なので、クライアント側からサーバー側のメソッドを呼ぶことも可能です。 `nginx-rtmp-module` では難しいですが、メディアサーバーに Adobe Media Server や Wowza Media Server を利用して開発をした場合、クライアント側からサーバー側のメソッドを呼ぶことが可能です。

ActionScript の場合、クライアントとサーバー間の双方向の接続が作成した後（ `NetConnection` オブジェクトが `nc.connect()` して、 `NetStatusEvent` が `"NetConnection.Connect.Success"` になった後）であれば、 `nc.call()` でサーバー側のメソッドを呼ぶことができます。 `nc.call()` の第１引数がメソッド名なので、 `nc.call("doSomething")` のようにクライアントから実行した場合、メディアサーバーに実装した該当のメソッドが実行されます。

![NetConnection.call](/images/live-streaming-and-rtmp-for-frontend-engineers/call.png)

たとえば Wowza Media Server の場合であれば、実装は Java なので下記のようなメソッドを実装することで、クライアントから Wowza Media Server のコンソールに `doSomething is called` と表示させることが可能です。

```
public void doSomething(IClient client, RequestFunction function, AMFDataList params) {  
  getLogger().info("doSomething is called");  
  // do something
}
```

AbemaTV の生放送番組では、RTMP の双方向通信を利用して、Web ブラウザから Wowza Media Server のメソッドを呼ぶことで、番組の進行具合に合わせて CM 入りのタイミングや視聴者参加型のインタラクションコンテンツのトリガーを最小限の遅延で放送に挿し込んでいます。

## RTMP で受け取った動画を HLS でもストリーミング

メディアサーバーはエンコーダーから RTMP 通信で送られた動画をそのまま RTMP でクライアントにプッシュ送信する以外に HLS や MPEG-DASH でストリーミングできるように変換することも可能です。たとえが nginx-rtmp-module の場合は先程作成した `nginx.conf` を編集して、 `application live` コンテキストに HLS に関する以下のディレクティブを追加します。

```
application live {
  live on;
  record off;

  hls on;
  hls_path /usr/local/nginx/html/hls;
  hls_fragment 1s;
  hls_type live;
}
```

すると、Live セッションの m3u8 プレイリストと 1 秒感覚のセグメントファイルを `/usr/local/nginx/html/hls/live` に出力してくれます。この `nginx.conf` を反映した Docker コンテナが起動している状態で、[http://localhost/hls/test.m3u8](http://localhost/hls/test.m3u8) に Safari でアクセスすると HLS でストリーミング再生ができます。（Safari でアクセスする理由は[前回](/posts/streaming-technology-basics-for-frontend-engineers/)書いた通り、HLS をネイティブサポートしている Web ブラウザが Safari だけだからです。）

![HLS を Safari で再生](/images/live-streaming-and-rtmp-for-frontend-engineers/safari-hls.png)

## まとめ

普段の Web フロントエンドの開発では、RTMP や ActionScript を扱う必要があることはあまりありません。しかし、こと動画やストリーミング領域となるとまだ Flash テクノロジーの安定性にお世話になることも多いように思います。WebRTC や WebSocket などの技術の組み合わせでこのあたりの事情もどんどん変化していきそうです。

## 参考

* [RTMP Protocol [DRAFT]](http://joy2world.tistory.com/attachment/ek8.pdf)

* [RTMP in the Age of HTTP Video Streaming: Don't Count it Out](
http://www.streamingmedia.com/Articles/Editorial/Featured-Articles/RTMP-in-the-Age-of-HTTP-Video-Streaming-Dont-Count-it-Out-100909.aspx)

* [Adobe Media Server概要](http://www.programming-knowledge.com/Adobe_Media_Server%E6%A6%82%E8%A6%81)

* [各種 RTMP サーバーでのライブストリーミングの実現](http://www.itoyanagi.name/temp/fuck/d20071109.html)

* [Red5 (media server)](https://en.wikipedia.org/wiki/Red5_(media_server))

* [リアルタイム動画配信コトハジメ](https://gist.github.com/voluntas/076fee77f30a0ca7a9b9)

* [RTMFPとRTMP](http://flashcafe.jp/kazari/k_m_clo/real.html)

* [Real Time Media Flow Protocol](https://ja.wikipedia.org/wiki/Real_Time_Media_Flow_Protocol)

* [Testing Tips For Today’s Tech: HTML5, WebSockets, RTMP, Adaptive Bitrate Streaming](http://www.neotys.com/blog/testing-tips-for-todays-tech-html5-websockets-rtmp-adaptive-bitrate-streaming/)

* [5-1.クライアントからサーバーのメソッドを呼び出す](http://coelacanth.heteml.jp/site/flash_wowza/article_5)

* [DvdGiessen/nginx-rtmp-docker](https://github.com/DvdGiessen/nginx-rtmp-docker)

* [Building Next Generation Real-Time Web Applications using Websockets](http://www.slideshare.net/chintal75/building-next-generation-realtime-web-applications)

* [WebSocket / WebRTCの技術紹介](http://www.slideshare.net/mawarimichi/websocketwebrtc)

