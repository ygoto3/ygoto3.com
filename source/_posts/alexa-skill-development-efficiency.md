title: Alexa アプリ（スキル）開発効率化メモ：ローカル開発／継続的インテグレーション／多言語対応
categories:
  - smart speaker
date: 2017-10-08 09:05:02
tags:
  - Alexa
    smart speaker
    VUI
    AWS
---

![Amazon Echo](/images/alexa-skill-development-efficiency/amazon-echo-devices.jpg)

担当している [AbemaTV](https://abema.tv/) が [Amazon Alexa に対応しました](https://www.cyberagent.co.jp/news/detail/id=20957)。今回 Alexa スキル（Alexa に機能を追加するためのアプリをスキルと呼びます）の開発にあたって、課題感があったチームによる平行開発、継続的インテグレーションおよびデプロイ、多言語対応についてのメモを残したいと思います。

## Web 技術を使って開発できるけど動作確認が大変

最近は Web で使われてきた技術がさまざまなデバイス用のプラットフォームでも利用できるようになり、Web エンジニアがこういった PC やスマートフォン以外のデバイス向けのアプリを開発することも多くなってきました。たとえば Amazon Echo などの Alexa 搭載端末や Google Chromecast 用アプリも Web 技術を使って開発できるので Web エンジニアが参入する障壁はかなり低いのですが、既存の Web アプリケーションと大きく違うのが、**実機で動作確認するのが一苦労**な点です。

Web アプリケーションであれば、実装している PC 上でビルドして同じ PC で Web サーバを立てるだけでほかの PC やスマートフォンからアクセスして動作確認できます。しかし、Amazon Echo や Google Chromecast のようなデバイスは、各々のアプリ開発者ポータルに登録されたアプリにしかアクセスできないようになっています。もちろん PC に直接デバイス接続するような機能もプラグもありません。両者とも物理的なインターフェースがシンプルすぎます。

実機で自作アプリの動作確認をするには毎回オンラインでアクセスできるどこかにデプロイする必要があります。アプリを実装する場所（PC）から動作テストを行える場所（オンラインのどこか）までがやたら遠いので、どうしても開発効率が悪くなりますし、複数人で平行して開発しようとすると人数分のテスト環境をどこかにホストする必要があり、そういった環境を別途管理するなど煩雑さも増します。

## Alexa スキルをチームで開発するときの課題

そんな中、現在サービス開発を担当している AbemaTV で、いわゆるスマート・デバイス向けのアプリ開発を専門で担当するチームを立ち上げました。Alexa スキルをチームで開発するとなっても、平行開発しづらいこの環境ではマンパワーのメリットが活かせません。平行開発を可能にするために以下の要素を満たす必要があります。

- 開発に使っているローカル PC 上である程度の動作確認が可能なこと
- 継続的インテグレーションが可能なこと

この 2 点を実現する手段について紹介するのですが、Alexa スキルの開発が初めての方のために一般的な開発手順について認識がない方のために、まず Alexa スキルの概要と開発手順をざっくり説明します。

## Alexa スキルの開発手順

Alexa スキルと呼ばれるものは、家電製品などを制御するためのスマートホームスキル、ニュースなどを読み上げるためのフラッシュブリーフィングスキルなど用途が特化しているものもありますが、今回は用途が汎用的なサービスを作ることができるカスタムスキルについてお話します。

カスタムスキルの開発手順は大まかに以下のようになります。

1. Voice User Interface (VUI) 作成
1. サービス・ロジック実装
1. VUI からサービス・ロジックへの連携
1. 実機やシミュレータによるテスト
1. 申請
1. 審査
1. 公開

具体的な実装手順自体は Amazon Alexa が公式に公開しているチュートリアル [Build An Alexa Fact Skill](https://github.com/alexa/skill-sample-nodejs-fact) を一通りやると大体分かります。あと、昨日クラスメソッドさんが投稿している [【祝Alexa日本上陸】とりあえず日本語でスキルを作ってみる](https://dev.classmethod.jp/etc/first-step-of-making-alexa-custom-skills/) もスキルの作り方がとても分かりやすいのでオススメです。

Alexa スキルを開発するには、普段聞き慣れない Alexa 特有の概念をいくつか理解する必要があります。個人的には最初分かりづらかったので、補足がてら簡単に説明します。

### Voice User Interface - VUI

Voice User Interface（以後 VUI）というのは、その名前の通り、声で操作するユーザー・インターフェースです。PC デスクトップ・アプリやスマートフォン・アプリでいうところのボタンとかテキスト入力ボックスなどにあたります。PC やスマートフォン上のアプリケーションはマウスやタッチパッドで操作するので、ボタンがクリックされたりテキストが入力されたときにアプリは特定の処理を実行します。しかし、Alexa の場合は操作手段が声です。どのように話しかけたときにどんな処理につなげるかを橋渡しする存在が VUI です。

VUI の少し深い話は[こちらの記事](https://website-usability.info/2017/10/entry_171019.html)など参考になりますが、とても抽象性が高い概念なので、ここでは以下の Alexa の VUI を構成する具体的な要素 3 つがどのようにユーザーの声とサービスをつなげているかを見ていきます。

- Invocation Name（呼び出し名）
- Intent（意図）
- Sample Utterance（発話サンプル）

#### Invocation Name

Invocation Name はいわゆる**スキルの呼び出し名**です。つまり、Alexa から特定のスキルを使いたいときに指定する名前です。スマホだとホームスクリーンでアプリを起動するときにアイコンをタップすると思いますが、Invocation Name に相当します。たとえば、AbemaTV スキルを呼び出したい場合であれば、**AbemaTV** が Invocation Name なので、「Alexa、**AbemaTV** を開いて」と話しかけると AbemaTV スキルが起動します。

#### Sample Utterance

Sample Utterance は、**発話のサンプル**です。ユーザーが実際に発話した文言をどんな意図（Intent）として受けるかを判断するための要素です。たとえばユーザーがランキングを知りたいときに、質問の仕方は何パターンもあります。ある人は「ランキングを教えてー」と言うかもしれませんし、別の人はランキングという言葉を使わず「いま人気の番組は何？」と尋くかもしれません。ただ、厳密な発話の仕方が異なってもユーザーが聞きたいことは結果同じだったりします。こういった異なる発話パターンをスキルがどんな意図として解釈するのかをマッピングするのが Sample Utterance の役割です。

```
RankingIntent ランキングを教えて
RankingIntent ランキングが知りたい
RankingIntent いま人気の番組は何
```

上記のような Sample Utterance を書いた場合、**「Alexa、AbemaTV のランキングを教えて」** と言っても、 **「Alexa、AbemaTV のランキングが知りたい」** と話しかけても **「Alexa、AbemaTV でいま人気の番組は何？」** と訊いても全てランキングを知りたいという意図と解釈して処理するようになります。意図には名前が付けることができ、ここでは `RankingIntent` という名前にしています。この名前を指定することは、次に説明する Intent に処理を接続するために重要です。

#### Intent

Intent は Sample Utterance によってマッピングされたユーザーの意図に対して実際のどんな処理を行う部分です。例の `RankingIntent` の場合はユーザーがランキングを知りたいという意図に対する処理なので、実際に現在のランキング・データを取得して、それをユーザーに回答するための文章を作成します。

Amazon Echo などの Alexa 対応デバイスはユーザーの発話音声をクラウド上の Alexa サービスに送り音声からユーザーの意図を解釈した後は、具体的なサービス・ロジックの処理依頼を AWS Lambda のような別サービスにリクエストします。Alexa が Amazon のプロダクトなので、チュートリアルにあるように AWS Lambda 上に処理を実装すると連携も簡単ですし、Amazon が用意している SDK の恩恵にあずかることができます。このフローは Alexa 公式ブログの記事 [Alexaスキル開発トレーニングシリーズ 第1回 初めてのスキル開発](https://developer.amazon.com/ja/blogs/alexa/post/6e716e5c-55b0-445b-b936-9cfac4712e7b/training-1) の下記スキル実行仕組みの図が分かりやすいです。

![Alexa スキル実行仕組みの図](/images/alexa-skill-development-efficiency/alexa-skill-flow.png)

### Intent に対するサービス・ロジックの実装

Alexa サービスから Intent が指定されて AWS Lambda にリクエストが飛んできます。その Intent に対するサービス・ロジックを Lambda Function として実装します。Amazon が提供する [Alexa Skills Kit SDK for Node.js](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs) では、Alexa をトリガーに呼び出された AWS Lambda のイベント情報を SDK を通じて Intent ごとのハンドラに振り分けてくれます。

```js
const Alexa = require('alexa-sdk');

// Intent ごとの処理を書いていく
const intentHandlers = {
  'RankingIntent': function () {
    this.response.speak(`本日のランキングは${getRanking()}です。`);
    this.emit(':responseReady');
  },
  'AMAZON.HelpIntent': function () { ... },
  'AMAZON.CancelIntent': function () { ... },
  'AMAZON.StopIntent': function () { ... },
  'LaunchRequest': function () { ... },
};

// Lambda のイベントとコンテキスト情報を SDK に渡して、Intent のハンドラにつなげる
exports.handler = function(event, context, callback) {
  const alexa = Alexa.handler(event, context);
  alexa.appId = APP_ID;
  alexa.registerHandlers(intentHandlers);
  alexa.execute();
};
```

この Lambda Function のエンドポイントを自作のスキルから接続するように設定することで Alexa にユーザーと対話させることができるようになります。

余談ですが、2017 年 11 月現在 AWS Lambda の Node.js サポート・バージョンが `v6.10` なので、本記事での JavaScript コードは全て `v6.10` 用になっています。

```sh
$ n
  ο node/6.10.0
    node/9.0.0
```

### Alexa スキルの開発手順を効率化したい

ここまでが [Build An Alexa Fact Skill](https://github.com/alexa/skill-sample-nodejs-fact) チュートリアルに載っている内容です。この内容をカスタマイズすれば、Alexa スキルを開発することはできます。しかし、前述したように、このスキルを動作確認するには毎回 AWS Lambda 上にコードをデプロイする必要があります。これでは開発効率も悪いです。しかも Alexa スキルは Lambda Function に紐づけて管理するので、チームで開発するとなると人数分のスキル設定と Lambda Function も別途用意しておく必要があります。しばらく開発してると、少々手間がかかりすぎるのでちょっと辛くなり、次のようなことができる環境が欲しいなと思い始めます。

- Amazon Skills Kit やAWS Lambda にデプロイすることなくローカルで気軽にテストしたい
- スキル設定と Lambda Function をまとめて継続的にインテグレーションしたい

結論を言うと、前者は **alexa-app というサードパーティーの Alexa スキル用フレームワークを利用することで解決**して、後者は **Amazon が提供する ASK-CLI という Alexa スキル管理のためのコマンドラインツールを CI ツールで走らせることで解決**しました。

### ローカルで Alexa スキルを開発する

ローカルで Alexa スキルを動作確認したいと思っている人は多いだろうなと思い、ネット上で記事を漁っていると、やはり同じことを考えている人がちょこちょこいるようです。書かれたのが 2016 年 3 月と少し古いですが、ローカルで Alexa スキルを開発するための詳しい手順が書かれた [Big Nerd Ranch](https://www.bignerdranch.com/) さんによる [Developing Alexa Skills Locally with Node.js: Implementing an Intent with Alexa-app and Alexa-app-server](https://www.bignerdranch.com/blog/developing-alexa-skills-locally-with-nodejs-implementing-an-intent-with-alexa-app-and-alexa-app-server/) という記事を見つけました。

#### alexa-app

[alexa-app](https://github.com/alexa-js/alexa-app) は、Alexa スキルを開発するためのサードパーティ製のフレームワークです。基本機能としては Alexa からの JSON リクエストを簡単に扱うための API や Alexa へ返すレスポンスを簡単に生成するための API を提供してくれます。実装が少し楽になるにはなるのですが、ただそれだけのメリットだと、サードパーティ製ということもあり将来的なメンテナンスとか考慮すると使うのを躊躇するところです。しかし、これを使いたく一番大きな理由は、今回の課題である「ローカルでのスキル・テスト」と「スキル設定と Lambda Function の継続的インテグレーション」を実現するために必要な次の 2 つの機能を提供してくれるからです。

- 実装したスキルを Express にも連結できる
- Intent と Sample Utterance もフレームワークで管理できる

#### スキルを Express アプリとしてテスト可能

実機である Alexa 端末から Intent 処理のリクエストを AWS Lambda で受ける必要があるので、alexa-app で実装したスキルを AWS Lambda のハンドラとして連結することは当然可能ですが、このフレームワークは同じ実装コードを任意の Express サーバに接続することも可能です。これで Alexa スキルを Express アプリのようにテストできます。

たとえば簡単な Alexa スキルを alexa-app で書いてみます。

```js
const alexa = require('alexa-app');

const app = new alexa.app('sample-alexa-skill');

function LaunchRequest(req, res) {
  const prompt = '人気の番組は何？、と訊いてください';

  res
    .say(prompt)
    .reprompt(prompt)
    .shouldEndSession(false);
};

app.launch(LaunchRequest);

module.exports = app;
```

これは「Alexa、AbemaTV を開いて」と話しかけたときに Alexa に「人気の番組は何？、と訊いてください」と答えさせる処理を alexa-app で書いたものです。このテストを Jasmine で書くと：

```js
const express = require('express');
const request = require('supertest');
const app = require('./app');

describe('Sample Alexa Skill', () => {
  var server;

  beforeEach(() => {
    // alexa-app で書いたスキルを任意の Express アプリと接続する
    const expressApp = express();
    app.express({
      expressApp,
      debug: true,
      checkCert: false,
    });
    server = expressApp.listen(3000);
  });

  afterEach(() => {
    server.close();
  });

  it('responds to a launch intent', () => {
    return request(server)
      .post('/sample-alexa-skill')
      .send({
        request: {
          type: 'LaunchRequest',
        }
      })
      .expect(200).then(res => {
        const actual = res.body.response.outputSpeech.ssml;
        const expected = '<speak>人気の番組は何？、と訊いてください</speak>'; 
        return expect(actual).toBe(expected);;
      });
  });
});
```

`const app = require('./app');` で読み込んでいるのが alexa-app のインスタンスです。これを Express に接続することで [SuperTest](https://github.com/visionmedia/supertest) などを使って通常の Express HTTP サーバをテストするのと同じ感覚でテストを書くことができます。

一点、注意なのですが、alexa-app インスタンスはそのままだと AWS Lambda に接続できないので、別途 `index.js` などのエントリーポイントを作成して以下のようにハンドラに渡すようにしておきます。

```js
module.exports.handler = require('./app').lambda();
```

#### alexa-app-server でデバッグ

alexa-app の Express 用のインターフェースを利用すればローカルでのデバッグ作業もかなり楽になります。

任意のリクエストに対するデバッグを行うときに、毎回リクエストを生成するコードを書くのは手間です。そこで alexa-app で書いたスキル用の Web サーバ [alexa-app-server](https://github.com/alexa-js/alexa-app-server) を使うと Web ブラウザから GUI で簡単にリクエストを生成できます。

![alexa-app-server のスクリーンショット](/images/alexa-skill-development-efficiency/alexa-app-server.png)

alexa-app-server の設定は簡単です。まずプロジェクトに alexa-app-server を `npm` か `yarn` でインストールします。

```sh
$ yarn add alexa-app-server
```

プロジェクトのルート・ディレクトリ直下に `apps` というディレクトリを作成して、そこに作成した alexa-app アプリのプロジェクトを移動します。（シンボリックリンクでも構いません。）

```sh
$ mkdir apps && mv ./somewhere/sample-alexa-skill apps
```

alexa-app-server は alexa-app インスタンスのモジュールを探すときに `package.json` の `main` プロパティの値をパスとして確認します。もしプロジェクトに `package.json` がない場合や `main` プロパティの値が alexa-app インスタンスのファイル・パスを指していない場合は変更します。

```json
{
  ...
  "main": "app.js",
  ...
}
```

サーバの設定を記述します。alexa-app-server をインストールした方のプロジェクトのルート・ディレクトリに `index.js` という名前でファイルを作成し次のように記述します。

```js
const AlexaAppServer = require('alexa-app-server');

AlexaAppServer.start({
  server_root: __dirname,     // サーバ・ルートへのパス
  public_html: 'public_html', // 静的コンテンツ
  app_dir: 'apps',            // alexa-app を置くディレクトリ。複数の alexa-app アプリを置くことができます
  app_root: '/alexa/',        // サービスのルート。これ以下に各 alexa-app のエンドポイントが作られる
  port: 8080                  // 使用するポート
});
```

保存したら起動してみます。

```sh
$ node index.js
```

[http://localhost:8080/alexa/sample-alexa-skill](http://localhost:8080/alexa/sample-alexa-skill) にアクセスすると自分が作ったスキル向けの JSON リクエストを生成できるインターフェースが表示されます。ここでスキルに実装済のインテントをプルダウンで設定したり任意の値を入力できるので、ローカルで効率的にスキルをデバッグすることが可能です。

![alexa-app-server でのデバッグ](/images/alexa-skill-development-efficiency/alexa-app-server-debug.png)

### 多言語対応

スキルを多言語対応する場合、Alexa からのリクエストにロケール情報が入っているので、それを使って地域／言語別にレスポンスを変更できます。

![alexa-app-server のロケール別のリクエスト切り替え](/images/alexa-skill-development-efficiency/alexa-app-server-locale.png)

alexa-app-server はロケール別のリクエスト切り替えがとても簡単なため多言語対応に関しても重宝します。通常、ロケールを頻繁に変更しながらのテストは大変です。Amazon Echo などの端末は [Alexa の管理コンソール](https://alexa.amazon.com/) での登録時にしかロケールを変更できないように見えますし、Amazon 開発者コンソールのシミュレータも言語ごとに分けられているため、ロケールを頻繁に変更するテストには向いていません。

alexa-app-server のインターフェース上、まだ `ja-JP` ロケールがオプションから選択できません。[フォーク](https://github.com/ygoto3/alexa-app-server.git#feature/ja-jp)して `ja-JP` をオプションに追加したものを使っています。こちら[プル・リクエスト中](https://github.com/alexa-js/alexa-app-server/pull/100)。

### 継続的インテグレーション

alexa-app を使うことで Express に連結してローカルで擬似的にテスト・デバッグできる範囲が広がり、複数人での平行チーム開発も可能になりました。しかし、チームで平行開発できるようになると今度はインテグレーションが問題になってきます。特に Alexa スキルの場合、Lambda Function とは別にスキル設定を Amazon 開発者コンソールで管理しているので、Lambda Function 用の最新コードにスキル設定が一致しないことが発生します。そういった不一致を発生させないために：

- スキル情報と Lambda Function の最新コードを常に同期する
- 同期タイミングは開発コードがメイン・レポジトリへ統合するタイミング

などが実現できれば嬉しいです。前者については、スキル情報とスキルに紐づいた Lambda をまとめて操作できるコマンドラインツールの [Alexa Skill Kit Command-line Interface (ASK CLI)](https://developer.amazon.com/docs/smapi/ask-cli-command-reference.html) を使うことで同期を取ることができます。後者については、ソースコードのバージョン管理に Git/GitHub を使っているのであれば、GitHub との連携が簡単な CI ツールで ASK CLI を走らせれば実現できます。本記事では、CircleCI を使います。

#### ASK CLI を使うための認証

ASK CLI を使うために Amazon Developer アカウント と AWS ユーザーの認証が必要です。今回は AWS Lambda 用のコードも含めてスキル管理したいので、ASK CLI から AWS を使える状態にする必要があります。

#### AWS CLI のユーザー認証

ASK CLI の認証時に AWS の認証情報を紐づけたいので、先に AWS ユーザーを認証します。（AWS CLI を既に使ったことがある方で認証済のプロファイルがある場合は、ここは読み飛ばしていただくのが良いでしょう。）

まず [AWS CLI](https://github.com/aws/aws-cli) をインストールします。Python パッケージで提供されているので、 `pip` などでインストールします。

```sh
$ pip install awscli
```

インストール完了後、 `which aws` などでパスが表示されることを確認できたら、次に AWS のユーザー認証を行います。AWS のアカウントがまだない場合は [アマゾン ウェブ サービス（AWS）](https://aws.amazon.com/jp/about-aws/) で作成します。AWS のアカウントを持っている場合は、ログインして [IAM Management Console](https://console.aws.amazon.com/iam/home#/home) サービスの [Users](https://console.aws.amazon.com/iam/home#/users) で AWS CLI 用のユーザーを作成します。作成時に表示される **AWS Access Key ID** と **AWS Secret Access Key** をメモしておきます。

Alexa スキルに紐づける Lambda Function を作成したり、IAM の操作も許可する必要があるので、作成したユーザーに下記のポリシーを追加します。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt000001",
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:PassRole",
                "lambda:CreateFunction",
                "lambda:AddPermission",
                "lambda:GetFunction",
                "lambda:UpdateFunctionCode"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

作成した AWS CLI 用のユーザーで AWS CLI を認証します。認証は `aws configure` コマンドで行います。

```sh
$ aws configure
AWS Access Key ID [None]: メモした AWS Access Key ID
AWS Secret Access Key [None]: メモした AWS Secret Access Key
Default region name [None]: us-east-1
Default output format [None]:
```

`Default region name` ですが、[Alexa Skills Kit のドキュメント](https://developer.amazon.com/docs/custom-skills/host-a-custom-skill-as-an-aws-lambda-function.html)に以下のように書いてあり、限られたリージョンでしか AWS Lambda の Alexa Skills Kit のサポートをしていないので注意が必要です。

> Lambda functions for Alexa skills can be hosted in either the US East (N. Virginia) or EU (Ireland) region. These are the only regions the Alexa Skills Kit supports.

これで AWS CLI が認証できました。認証情報が `~/.aws/config` と `~/.aws/credentials` に保存されていれば OK です。

```sh
$ cat ~/.aws/config
[default]
region = us-east-1
```

```sh
$ cat ~/.aws/credentials
[default]
aws_access_key_id = XXXX
aws_secret_access_key = XXXXXXXX
```

#### ASK CLI のアカウント認証

次に ASK CLI を Amazon Developer アカウントに認証します。 `ask init` コマンドを使って AWS CLI でユーザー認証したプロファイルと紐づけながら Amazon Developer アカウントへの認証手順が進みます。

```sh
$ ask init
-------------------- Initialize CLI --------------------
Setting up ask profile: [default]
? Please choose one from the following AWS profiles for skill's Lambda function deployment.
 (Use arrow keys)
❯ default
  ──────────────
  skip AWS credential for ask-cli
  ──────────────
```

初めて実行する場合は下記のようなダイアログが表示されますが、既にデフォルトのプロファイルがある場合は、新規プロファイルを作成するのか、既存プロファイルを上書くのかを尋かれるダイアログが表示されます。

ここで紐づけたい AWS のプロファイルを選択します。例では、先程 `aws configure` でユーザー認証したときプロファイル指定をしていないので、 `default` という名前で保存されているので、 `default` を選択します。

Web ブラウザが起動し、「Login with Amazon」のページで表示されるのでログインします。

![Login with Amazon のスクリーンショット](/images/alexa-skill-development-efficiency/amazon-sign-in.png)

次に権限の確認をされるので、問題なければ「Okay」ボタンをクリックします。

![Amazon 権限確認画面のスクリーンショット](/images/alexa-skill-development-efficiency/amazon-consent.png)

無事ログインが成功すると、Sign in was successful. Close this browser and return to the command line interface. というメッセージでブラウザを閉じろと言われるので閉じます。

ASK CLI の認証情報に関しては `~/.ask/cli_config` に保存されています。

```sh
$ cat ~/.ask/cli_config
{
  "profiles": {
    "default": {
      "aws_profile": "default",
      "token": {
        "access_token": “XXXXXXX”,
        "refresh_token": “XXXXXXX”,
        "token_type": "bearer",
        "expires_in": 3600,
        "expires_at": "20XX-XX-XXTXX:XX:XX.XXXZ"
      },
      "vendor_id": "XXXXXXXXXXXXXX"
    }
  }
}
```

#### ASK プロジェクトを作成

ユーザー認証が通ったので、ASK CLI でスキル全体を管理できるようにプロジェクトを新規作成します。

```sh
$ ask new --skill-name sample-alexa-skill --lambda-name sample-alexa-skill
```

これで新規の Alexa スキル・プロジェクトの雛形が作成されます。tree コマンドを実行すると次のようなディレクトリ・ツリーが表示されるはずです。

```sh
$ tree
.
└── sample-alexa-skill
    ├── lambda
    │   └── custom
    │       ├── index.js
    │       ├── node_modules
    │       │   └── ...
    │       ├── package-lock.json
    │       └── package.json
    ├── models
    │   └── en-US.json
    └── skill.json
```

この中で重要な各要素の役割はざっくりと次の通りです。

| 要素 | 役割 |
| --- | --- |
| lambda | Lambda 用のコードを格納するディレクトリ |
| models | Intent Schema や Sample Utterance などを格納するディレクトリ |
| skill.json | スキルの申請に必要な情報を記述するファイル |

雛形ができたので、スキルに必要な情報を設定していきます。

まず Alexa は Apple の App Store などと同様にスキルを公開するのに申請が必要なので、`skill.json` にこのスキルの名前やこのスキルを使うためのフレーズ例など、申請に必要な情報を記述します。

次に `lambda` ディレクトリには、今回 alexa-app で作った AWS Lambda 用モジュールを格納します。生成された `lambda` ディレクトリ以下の雛形は必要ないので `custom` ディレクトリごと削除してしまい、代わりに AWS Lambda 用モジュールのディレクトリを `custom` という名前でここに移動します。

```sh
$ rm -rf lambda/custom
$ mv somewhere/sample-alexa-skill lambda/custom
```

最後に `models` ディレクトリにスキルの Intent のデータ構造を示した Intent Schema と Sample Utterance の情報を格納する必要があるのですが、これらは alexa-app フレームワーク上の実装コード内に記述されています。なので、フレームワークの API を使って JSON ファイルとして出力するスクリプトを書きます。

```js
const app = require('../index');

// alexa-app で実装したアプリ・オブジェクトは schemas.askcli で
// ASK プロジェクト用の Interaction Model JSON を出力できる
// 引数に Invocation Name 呼び出し名を渡す
const interactionModel = app.schemas.askcli('Sample Alexa Skill');

// JSON を標準出力に流します
process.stdout.write(interactionModel);
```

このスクリプトを実行した出力をロケール ID をファイル名にした JSON にパイプします。日本語であれば `models/ja-JP.json` にパイプします。

```sh
$ node ./scripts/gen-interaction-model.js > ./models/ja-JP.json
```

`ja-JP.json` の中身はこんな感じです。

```json
{
   "interactionModel": {
      "languageModel": {
         "intents": [
            {
               "name": "SayNumber",
               "samples": [
                  "数字の {number} を言って"
               ],
               "slots": [
                  {
                     "name": "number",
                     "type": "AMAZON.NUMBER",
                     "samples": []
                  }
               ]
            }
         ],
         "types": [],
         "invocationName": "Sample Alexa Skill"
      }
   }
}
```

多言語対応する場合は、必要な分、別のロケール ID の JSON ファイルにパイプします。ロケール ID をファイル名にした JSON ファイルを複数 `models` ディレクトリに入れておくことにより、ASK CLI が言語別のスキル情報として登録してくれます。ここでは日本語と英語に対応するために `ja-JS.json` とは別に `en-US.json` を書き出します。

```sh
$ APP_LOCALE=en-US node ./scripts/gen-interaction-model.js > ./models/en-US.json
```

ここでは、環境変数 `APP_LOCALE` に応じて Sample Utterance が切り替わるように alexa-app の Intent を実装しました。 `en-US.json` の Sample Utterance 部分などが差し替わって出力されます。

```json
{
   "interactionModel": {
      "languageModel": {
         "intents": [
            {
               "name": "SayNumber",
               "samples": [
                  "say the number {number}"
               ],
               "slots": [
                  {
                     "name": "number",
                     "type": "AMAZON.NUMBER",
                     "samples": []
                  }
               ]
            }
         ],
         "types": [],
         "invocationName": "Sample Alexa Skill"
      }
   }
}
```

言語に関する設定は `models` ディレクトリのほかに `skill.json` ファイルにも記述する必要があるので、必要に応じてロケール情報を追加しましょう。

```json
{
  "skillManifest": {
    "publishingInformation": {
      "locales": {
        "en-US": {
          "summary": "Sample Alexa Skill's Short Description",
          "examplePhrases": [
            "Alexa open Sample Alexa Skill",
            "Alexa tell Sample Alexa Skill say the number 1",
            "Alexa tell Sample Alexa Skill say the number 3"
          ],
          "name": "sample-alexa-skill",
          "description": "Sample Alexa Skill's Full Description"
        },
        "ja-JP": {
          "summary": "Sample Alexa Skill の説明",
          "examplePhrases": [
            "アレクサ、Sample Alexa Skill を開いて",
            "アレクサ、Sample Alexa Skill で数字の1を言って",
            "アレクサ、Sample Alexa Skill で数字の3を言って"
          ],
          "name": "sample-alexa-skill",
          "description": "Sample Alexa Skill の詳細な説明"
        }
      },
      "isAvailableWorldwide": true,
      "testingInstructions": "Sample Alexa Skill's Testing Instructions.",
      "category": "EDUCATION_AND_REFERENCE",
      "distributionCountries": []
    },
    "apis": {
      "custom": {
        "endpoint": {
          "sourceDir": "lambda/custom"
        }
      }
    },
    "manifestVersion": "1.0"
  }
}
```

これで alexa-app で実装した多言語対応スキルを ASK プロジェクトとして管理できるようになりました。

#### Alexa スキルをデプロイする 

作成した ASK プロジェクトを Amazon Echo などの実機で試すためには、Alexa スキルとしてデプロイする必要があります。ASK CLI の `deploy` コマンドを使うだけです。

```sh
$ ask deploy
```

Alexa スキルがデプロイされたことを確認するため、[Amazon 開発者コンソール](https://developer.amazon.com/) に行き、Alexa Skills Kit のスキル・リストに `sample-alexa-skill` が登録されているか確認します。

![Alexa Skills Kit のスキル・リスト](/images/alexa-skill-development-efficiency/amazon-apps-developer-portal.png)

#### GitHub 連携で CI

ASK CLI で Alexa スキルをデプロイできるところまで来たので、あとはこの手順を CI ツールに設定すれば継続的にテストしたりデプロイしたりすることができます。もちろん CI ツールは何でも構いませんが、ここでは GitHub に簡単に連携ができる CircleCI を使って、GitHub Flow ベースで単純で DevOps な感じの運用ができればいいなというイメージ。

#### デプロイ時に必要な処理の依存関係を Makefile にまとめる

CircleCI に処理を書いていく前に、スクリプトの実行手順に若干の依存関係ができてしまったので、明示的に手順を示す意味で Makefile にまとめます。

```basemake
.PHONY: interaction_model
interaction_model:
	node ./lambda/custom/scripts/gen-interaction-model.sh > models/ja-JS.json
	APP_LOCALE=en-US node ./lambda/custom/scripts/gen-interaction-model.sh > models/en-US.json

.PHONY: deploy
deploy: interaction_model
	ask deploy
```

先程も説明した通り、多言語対応する場合は、環境変数別に `gen-interaction-model.sh` を走らせて言語別の Model を書き出す処理もまとめておきます。

#### CircleCI にインテグレーション／デプロイ処理を追加する

デプロイ時に必要な処理もまとまったので、CircleCI プロジェクト用にインテグレーション処理とデプロイ処理を書いていきます。ASK プロジェクト・ディレクトリ直下に `.circleci/config.yml` ファイルを作り、以下のような YAML でジョブを記述します。

```yaml
defaults: &defaults
  working_directory: ~/repo
  docker:
      # Use the same Node version as that of AWS Lambda's
    - image: circleci/node:6.10

version: 2
jobs:
  build:
    <<: *defaults
    steps:
      - checkout

      - restore_cache:
          key: v1-dependencies-{{ checksum "package.json" }}

      - restore_cache:
          key: v1-dependencies-{{ checksum "lambda/custom/package.json" }}

      - run:
          name: Install dependencies
          command: yarn install

      - run:
          name: Install dependencies for Lambda function
          working_directory: lambda/custom
          command: yarn install

      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}

      - save_cache:
          paths:
            - lambda/custom/node_modules
          key: v1-dependencies-{{ checksum "lambda/custom/package.json" }}

      - persist_to_workspace:
          root: .
          paths:
            - node_modules
            - lambda/custom/node_modules

  test:
    <<: *defaults
    steps:
      - checkout
      - attach_workspace:
          at: .

      - run:
          name: Run tests
          command: |
            yarn test

      - run:
          name: Report code coverage
          command: $(yarn bin)/codecov

  deploy:
    <<: *defaults
    steps:
      - checkout
      - attach_workspace:
          at: .

      - deploy:
          name: Deploy Skill
          command: |
            sudo apt-get -y -qq install python3-pip gettext
            sudo pip3 install awscli
            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
            aws configure set default.region us-east-1

            sudo npm i ask-cli -g
            mkdir -p ~/.ask
            echo ${ASK_CLI_CONFIG} | base64 -d > ~/.ask/cli_config

            make deploy 

workflows:
  version: 2
  test_and_deploy:
    jobs:
      - build
      - test:
          requires:
            - build
      - deploy:
          requires:
            - build
            - test 
          filters:
            branches:
              only: master
```

テストの実行やらコード・カバレッジの取得やらしていますが、前述したようにデプロイに関しては

- スキル情報と Lambda Function の最新コードを常に同期する
- 同期タイミングは開発コードがメイン・レポジトリへ統合するタイミング

のように自動化したかったので、CircleCI で GitHub の `master` ブランチへのマージのタイミングで下記 3 点の処理を実行するようにタスクを記述しています。

- AWS CLI のインストールおよび設定
- ASK CLI のインストールおよび設定
- スキル設定と AWS Lambda コードのデプロイ

ローカル同様、CircleCI 上でも AWS と ASK の認証が必要です。それぞれ CircleCI 上でインストールして認証情報を設定します。

まず ASK CLI で AWS Lambda をデプロイするために必要なので AWS CLI をインストールします。AWS CLI を認証するために環境変数 `AWS_ACCESS_KEY_ID` と `AWS_SECRET_ACCESS_KEY` を参照しています。なので、CircleCI 側の環境変数に両者を登録しておきます。

ASK CLI に関しては、先程の認証情報が `~/.ask/cli_config` に記述してあるので Base64 にエンコードしてクリップボードにコピーします。Mac 系の OS なら `pbcopy` できるのでこんな感じです。

```sh
$ base64 ~/.ask/cli_config | pbcopy
```

クリップボードの中身を CircleCI 側の環境変数 `ASK_CLI_CONFIG` として設定します。これで GitHub にホストした `master` ブランチに変更をマージする度に Amazon 開発者コンソールの Alexa スキル情報と紐づいた AWS Lambda Function コードが最新状態に更新されるようになりました。

Alexa スキルを実際に公開するためには、スキルを申請して審査を通過する必要があります。ASK CLI は申請もコマンドラインで送信できるので、それも自動化したい人は `ask api submit` コマンドなど必要な手順を CI プロセスに追加してもいいかもしれません。

### まとめ

Alexa スキルは Web 技術を使って簡単に開発を始めることができます。新しいデバイスなので勝手が掴めず、動作確認等々が大変な部分もあって最初はちょっととまどいましたが、探せば開発を助けるツールの恩恵を受けることができ、少しずつ開発しやすい環境を構築できるようになってるなと感じます。本記事がこれから Alexa スキルを開発をする人の効率化の参考になれば幸いです。

本記事の内容のサンプルコードは下記に上げてあります。

- [ygoto3/sample-alexa-skill](https://github.com/ygoto3/sample-alexa-skill)
- [ygoto3/alexa-app-workspace](https://github.com/ygoto3/alexa-app-workspace)

### 参照

- [ASK CLI Command Reference](https://developer.amazon.com/docs/smapi/ask-cli-command-reference.html)
- [Alexa Parrot](https://github.com/dblock/alexa-parrot)

