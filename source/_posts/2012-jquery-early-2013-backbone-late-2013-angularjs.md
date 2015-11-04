title: 2012 jQuery→Early 2013 Backbone→Late 2013 AngularJSな自分がハマった10のこと
tags:
  - AngularJS
id: 55
categories:
  - JavaScript
date: 2013-12-22 23:55:46
---

こちらは[Frontrend Advent Calendar 2013](http://www.adventar.org/calendars/62 "Frontrend Advent Calendar 2013") 22日目の記事です。

本記事は、2012年までjQueryだけで開発していたフロントエンドエンジニアの自分が、[Frontrend](http://frontrend.github.io/ "Frontrend")な方々に影響を受けたのをきっかけに、とあるコミュニティ系WebサービスでAngularJSを導入するまでの過程で影響された記事やイベントや人を時系列で紹介すると共に導入後AngularJSの開発でハマった10のことを紹介します。

### 2012.07：Anatomy of Backbone.jsで学ぶ

[AngularJS](http://angularjs.org/ "AngularJS")と言えば、MVC的なアーキテクチャをJavaScript開発に取り入れるためのフレームワークで現在は業務でも使用させていただいています。

しかし、振り返ってみると、2012年はMVなんちゃらなフレームワーク等には、全く無縁の生活を送っていました。

そんな2012年の7月頃に、Frontrendの[Yuya Saito](https://twitter.com/cssradar "Yuya Saito")さんに[Code School](https://www.codeschool.com/ "Code School")の[Anatomy of Backbone.js](https://www.codeschool.com/courses/anatomy-of-backbonejs "Anatomy of Backbone.js")を紹介していただいて、[Backbone](http://backbonejs.org/ "Backbone.js")をはじめとするJS開発におけるMVC的な発想を知ります。

ご存知の方も多いと思いますが、[Code School](https://www.codeschool.com/ "Code School")はステップごとに用意されている講義形式の動画で学び、そのまま出題される課題をブラウザ上のエディタでコードを書いて解答して、実践的にプログラミングを学ぶことができるサービスです。

[Anatomy of Backbone.js](https://www.codeschool.com/courses/anatomy-of-backbonejs "Anatomy of Backbone.js")を受講した印象としては、とても初心者が学習しやすいチュートリアルです。JSライブラリと言えば[jQuery](http://jquery.com/ "jQuery")しか使ったことがなかった自分はここでBackboneの基本の基本を学べたと思います。

### 2013.02：Frontrend Vol.4でBackbone導入決意

基本のチュートリアルをこなしたとは言え、実際のプロジェクト(リリース済)で使うことに敷居の高さとリスクを感じ、なかなか導入することもできないまま2013年になってしまいました。

しかし、2月9日に開催された[Frontrend Vol.4](http://frontrend.github.io/events/04/r "Frontrend Vol.4")での[ahomu](http://aho.mu/ "ahomu")さんセッション「[jQuery to Backbone – アーキテクチャを意識したJavaScript入門](http://vimeo.com/album/2260782/video/59558632 "jQuery to Backbone – アーキテクチャを意識したJavaScript入門")」で、まさに「自分みたいにjQueryくらいしかライブラリ触ったことない人でもBackboneとか使えるかもー」と思い始めます。セッションの内容を参考にさっそく自分がフロントエンドを担当しているWebサービスでBackboneの導入を始めました。

このセッションは、当時のJS開発におけるjQueryが解決しない問題とBackboneを導入することで得られるメリットが分かりやすく説明されている上に、jQueryベースで記述されたコードをBackboneの構造に徐々に移していく具体的なコーディング手順まで紹介されています。自分はまさにその手順に従って、プロジェクトにBackboneを導入できたように思います。

### 2013.09：Backbone Is Not EnoughでEmberJSとAngularJSが気になる

Backboneを使い始めて半年くらい経ち、自分が担当しているサービスでは巡るめくアップデートとプロモーション施策の実装をしていかなければいけませんでした。そしてフロントエンドエンジニアが自分だけ、という状況だったこともあり「何か劇的に開発を高速化できる方法はないかな」と日々模索していました。

そして、[Shine Technologies](http://www.shinetech.com/ "Shine Technologies")のブログ記事「[Backbone Is Not Enough](http://blog.shinetech.com/2013/09/06/backbone-is-not-enough/ "Backbone Is Not Enough")」を読み、AngularJSに興味を持ち始めます。

記事では、Backboneでの大規模なSPA開発において、ネストされるViewをうまく構成する難しさ、Viewをテストする難しさ、メモリ管理の難しさ、容易に遅くなってしまうレンダリングへの対策やデータバインディングの重要さなどに関して[EmberJS](http://emberjs.com/ "EmberJS")やAngularJSと比較して書かれていますが、中でも「Backboneと比較したEmberJSとAngularJSのコード記述量がかなり少ない」という1点に惹かれて、EmberJSとAngularJSが気になり始めます。

### 2013.10：A comparison of the two-way binding in AngularJS, EmberJS and KnockoutJSでAngularJSが一番良いような気がしてくる

EmberJSとAngularJSが気になり始めましたが、これらのフレームワークはそれぞれ何が違うのかは正直よく分からないでいました。Backbone単体では自分でこつこつ設定するデータバインディングを簡単に設定できる点に関しては、EmberJSもAngularJSも同じです。

その疑問に関しては、2013年10月に公開された[JSConf EU 2013](http://2013.jsconf.eu/ "JSConf EU 2013")（9月開催）のMarius Gundersen氏セッションの動画「[A comparison of the two-way binding in AngularJS, EmberJS and KnockoutJS](http://www.youtube.com/watch?v=mVjpwia1YN4 "A comparison of the two-way binding in AngularJS, EmberJS and KnockoutJS")」で解決されます。

このセッションでは、AngularJS、EmberJS、[KnockoutJS](http://knockoutjs.com/ "KnockoutJS")の双方向データバインディングにおける挙動の違いが分かりやすくまとめられています。

このセッションにおいてAngularJSは、Dirty Checkingという仕組みと非同期でデータをバインドをしているため、

*   リストの単純なレンダリングに関しては速い
*   Modelが複雑で巨大になってくるとレンダリングが遅い
*   コンピューティング処理が挟まれるプロパティに関しては重くなる

などの特徴が説明されています。

担当サービスにおいて、Modelはそこまで複雑かつ巨大にならないと思った点と単純なレンダリングの速さが気に入り、AngularJSの導入を決めました。

### 2013.10：A Better Way to Learn AngularJSで学ぶ

導入を決めたら、次はAngularJSについて学習しなければいけません。

「[Anatomy of Backbone.js](https://www.codeschool.com/courses/anatomy-of-backbonejs "Anatomy of Backbone.js")」のような効率が良い（そしてできれば無料の）ラーニングリソースを探していたら（残念ながら、[Code School](https://www.codeschool.com/ "Code School")にAngularJSのコースはありませんでした。）、「[A Better Way to Learn AngularJS](http://www.thinkster.io/pick/GtaQ0oMGIl/ "A Better Way to Learn AngularJS")」というラーニングリソースを見つけました。

[Code School](https://www.codeschool.com/ "Code School")のようにコーディングで課題を問いていくリッチな機能は無いですが、初心者にも分かりやすく説明されたAngularJSのチュートリアルを動画で見ることができます。無料で学べるのですが、これを一通りこなすだけでAngularJSの基本的な使い方に関しては網羅できるように思います。

### 2013.10：AngularJSで開発を始めていろいろとハマる

もちろん基本的な使い方しか学んでいない自分は、AngularJSでサービスの開発を始めると、いろんなところでハマりました。Backboneを利用していた時と比べ、確かにコード記述量は減りましたが、AngularJSについて調べている時間は増えました。

そんなわけで「[Backbone Is Not Enough](http://blog.shinetech.com/2013/09/06/backbone-is-not-enough/ "Backbone Is Not Enough")」に載っていたAngularJSの[ラーニングカーブ](http://shinetechblog.files.wordpress.com/2013/09/learning_curves.png)をしみじみ実感しましたが、同時にノウハウが溜まった後は劇的に楽になるはず、という期待でいっぱいでした。

#### AngularJSで開発を始めてハマった10のこと

本記事のタイトルにある通り、ここからはAngularJSでの開発で最初の頃に自分がハマった点を回避策とともにリストしていきたいと思います。

##### 1\. JSファイルをminifyしたら動かなくなった

AngularJSでは、Controllerの書き方にパターンがいくつか存在しますが、そのパターンのうちminifyするとJSが正しく動作しなくなるものがあります。

AngularJSにはDI (Dependency Injection)と呼ばれる仕組みがあります。Controllerで使用するserviceを指定する際に、functionの引数に指定された変数名から自動的に必要なserviceを決定できるすごい機能を持っているために、起こってしまうのがこの問題です。

回避策は2点。

*   引数にわたすserviceの名前を文字列で明示する
*   ngminを使用する

1点目の回避策は、引数にわたすserviceの名前を文字列で明示することです。

minifyすると動かない書き方（functionの引数だけでserviceを指定）

``` JavaScript
angular.module('myApp', [])
  .controller('MyCtrl', function ($scope, $http) {
    // ...
  });
```

minifyしても動く書き方（functionの引数の前に文字列でserviceを指定）

``` JavaScript
angular.module('myApp', [])
  .controller('MyCtrl', ['$scope', '$http', function ($scope, $http) {
    // ...
  }]);
```

2点目の回避策は、使用するpre-minifierを[ngmin](https://github.com/btford/ngmin "ngmin")にすることです。

こちらだと動かない方の書き方をしても、ngminが動く方の書き方に変えてminifyしてくれます。これでキーの打数は減り、楽して開発できるでしょう。

##### 2\. RequireJSなどで遅延ロードするとAngularJSが正しく動かない

Backboneを使っているときは、モジュールごとに処理を分けてRequireJSで遅延ロードさせたりしていましたが、AngularJSで同様のことをしようと思うとモジュールのロードが完了する前にAngularの起動が行われて、意図した挙動をしなくなることがあります。

回避策は、手動で起動することです。

``` JavaScript
require(['require', 'exports', 'app'], function (require, angular) {
  // 起動前のいろんな処理...

  // 手動で起動
  angular.bootstrap(document, ['myApp']);
});
```

##### 3\. AngularJS起動前だとng-hideなどで隠れるはずの要素が一瞬表示されてしまう

SPAで作っている場合には問題ないかもしれませんが、そうでない場合、先述した手動での初期化などを行うとAngularJSの起動が遅くなり、ページロード時にngHideディレクティブなどを指定しているDOMが一瞬表示されてしまうことがあります。

[angular.css](https://github.com/angular/angular.js/blob/master/css/angular.css "angular.css")を読み込んでおき、対象の要素のclass属性に**ng-hide**を足しておくことで回避できます。

ngHideなどの処理は、内容的には**display:none;**を適用するために、**ng-hide**というクラスを要素に付けているだけです。

``` CSS
.ng-hide {
  display: none !important;
}
```

[angular.css](https://github.com/angular/angular.js/blob/master/css/angular.css "angular.css")を読み込まなくても、上記のようなcssが入っていれば大丈夫です。

##### 4\. AngularJS起動前に{{ expression }}が一瞬表示されてしまう

上記と同様に、AngularJS起動前の評価されていないexpressionも生のテキストとしてページロード時に一瞬表示されてしまいます。

``` HTML
<p>{{comment}}</p>
```

このような場合は、

``` HTML
<p ng-bind="comment"></p>
```

で回避することができます。
もしくは、

``` HTML
<p ng-cloak></p>
```

でも可能ですが、ngCloakを使用する場合はngHide同様[angular.css](https://github.com/angular/angular.js/blob/master/css/angular.css "angular.css")を読み込んでおく必要があります。

この問題を回避するという用途的には、ngCloakを使う方が正しいようです。

##### 5\. textarea要素にデフォルト値が設定できない

textarea要素にデフォルト値を設定したいと思い、

``` HTML
<textarea ng-model="comment">コメント</textarea>
```

上のように設定してもテキストエリア内にもcomment Modelにも反映されません。

回避策は、ng-initで明示的にcomment Modelをそのデフォルト値で初期化することです。

``` HTML
<textarea ng-init="comment = ‘コメント’" ng-model="comment"></textarea>
```

##### 6\. 表示テキストが2重エスケープされる

AngularJSでHTMLにバインドすると自動的にエスケープがかかってしまいます。サーバサイドでエスケープ処理を施している場合は、場合によっては2重エスケープ状態が発生してしまいます。

そんなときは、ngSanitizeをインストールして、

``` JavaScript
angular.module('myApp', ['ngSanitize']);
```

DOMには

``` HTML
<span>{{item.content}}</span>
```

と書く代わりに

``` HTML
<span ng-bind-html="item.content"></span>
```

のようにしてエスケープ処理をしないようにできます。

##### 7\. Directiveの命名規則がややこしい

ここからは、AngularJSで一番素敵な機能だと思っているDirectiveについてのネタが続きます。

``` HTML
<x-switch></x-switch>
```

このようなElement DirectiveをJS側で定義したいとき、

``` JavaScript
angular.module('myApp', [])
  .directive('xSwitch', function () {
    // ...
  });
```

のように、ハイフンつなぎ（x-switch）→キャメルケース（xSwitch）とする必要があります。

##### 8\. カスタムDirectiveの内側のコンテンツが消える

当然と言えば当然なのですが、テンプレートを指定したDirective DOMの内側にあらかじめコンテンツを入れておいてもテンプレートに置き変えられてしまいます。最初はこれに気づきませんでした。

Directiveの内側に入れたコンテンツをテンプレート内の特定の場所で利用したい場合は、JS側のDirective定義でtranscludeをtrueに設定し、template側のコンテンツを利用したい要素にng-transclude属性を設定する。

``` JavaScript
angular.module('myApp', [])
  .directive('someDirective', function () {
    return {
      transclude: true,
      template: '<div class="well"><p ng-transclude></p></div>',
      link: function() {
        // …
      }
    };
  })
```

##### 9\. カスタムDirectiveをたくさん作っていたらtemplate用HTMLファイルのリクエストでいっぱいになる

カスタムDirectiveはとても便利で、HTMLを綺麗にしておけるし、処理も独立させられるのでついついたくさん作ってしまいます。そのときに使用するテンプレートHTMLを外部においてtemplateUrlで読み込ませると、そのテンプレートの数分だけリクエストが別途走ってしまいます。

回避策は、外部テンプレートHTMLをtemplateCacheを使用してJSにキャッシュしてくれる[grunt-angular-templates](https://github.com/ericclemmons/grunt-angular-templates "grunt-angular-templates")を使用することです。

こちらに関しては、[別記事](http://www.ygoto3.com/?p=8 "grunt-angular-templatesを使ってみた")参照。

##### 10\. AngularJS Batarangの存在を知るのが遅かった

これは、ハマったことでも何でもないですが、AngularJSのデバッグをする際にDev ToolsとDOM自体にプロパティ表示用のコードを書いたりしていました。

AngularJSのデバッグ用Chrome Extensionに「[Angular Batarang](https://github.com/angular/angularjs-batarang "Angular Batarang")」というものがあります。これの存在をもっと早く知っていたらデバッグがもっと楽だっただろうと思います。

BackboneにもFirebug Extensionの「[Backbone-Eye](https://addons.mozilla.org/en-US/firefox/addon/Backbone-Eye/ "Backbone-Eye")」などがありますし、やはり専用のデバッグツールがあると開発も快適です。

### 2013.10：ブログを始める（余談）

開発にハマっては調べハマっては調べしている日々の中、Frontrendの[Hiroki Tani](http://inkdesign.jp/ "Hiroki Tani")さんに何気なく「ブログとか書いてみたらどうですか」と言われたのをきっかけに、どうせならAngularJSで調べたことでも書いてみようと思い、このブログを始めました。あまり継続して書けていないので、来年はもっと頑張ろうと思います。

### 終わりに

[Frontrend Advent Calendar 2013](http://www.adventar.org/calendars/62 "Frontrend Advent Calendar 2013")という場を借りて、とても個人的な振り返りをさせていただきました。今日この記事を書けるのも、先に書いた通り[Frontrend](http://frontrend.github.io/ "Frontrend")の方々にいただいたきっかけが大きく影響しています。

明日は、[ysugimoto](http://blog.wnotes.net/blog/article/webrtc-beginning "ysugimoto") さんです。
