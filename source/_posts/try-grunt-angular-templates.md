title: grunt-angular-templatesを使ってみた
tags:
  - AngularJS
id: 8
categories:
  - JavaScript
date: 2013-10-31 21:42:13
---

### AngularJSのElement Directiveを使っていて

AngularJSのElement Directiveは便利でHTMLも綺麗になるので好きなんですが、テンプレートをDirectiveの中に書くのだけ好きになれません。

JavaScriptファイル内に文字列として書くためエディタのカラーリングも効かず、書くにくいのが嫌でした。

もちろんtemplateの代わりにtemplateUrlで外部HTMLを使うこともできますが、別途AJAXしてしまうので無駄にファイルのリクエスト数が増えてしまうのも好ましくありません。

**よく使うテンプレートたちは外部HTMLで書いておいて、リクエスト数は抑えたい。**

それの1つの解決方法として、今回は**grunt-angular-templates**を使いました。

### grunt-angular-templatesとは

[grunt-angular-templates](https://github.com/ericclemmons/grunt-angular-templates)は、Eric Clemmons氏のGruntプラグインで、指定したHTMLファイル郡をミニファイ・結合して1つのJavaScriptファイルとして出力してくれます。

そのJavaScriptファイルには、指定したHTMLファイルを$templateCache.putするように記述してあるので、後はそのURLをtemplateUrlで指定すると別途AJAXすることなくキャッシュしたデータを使用してくれます。

### インストール

インストールはnpm。

``` bash
$ npm install grunt-angular-templates --save-dev
```

そしてGruntfile.jsでロードを有効化。

``` JavaScript
grunt.loadTasks('grunt-angular-templates');
```

### 設定

まずは一番シンプルな形を試します。
AngularJSのテンプレートとして書いたHTMLファイル全てを
$templateCacheにキャッシュさせるJavaScriptを出力させます。

Gruntfile.jsで以下の最低限のオプションだけ記述します。

``` JavaScript
ngtemplates:  {
  myApp: { // angularのモジュール名に合わせます
    src: 'templates/**/*.html',
    dest: 'js/templates.js'
  }
}
```

"myApp"はテンプレートを使うモジュール名に合わせます。

### 実行

実行します。

``` bash
$ grunt ngtemplates
```

もし下記のような名前のHTMLファイルを入れていれば

``` bash
templates
└── btnLike.html
└── btnFollow.html
```

js/templates.jsには以下のようなコードが出力されます。

``` JavaScript
angular.module('myApp').run(['$templateCache', function($templateCache) {
  $templateCache.put('templates/btnFollow.html',
    "<a href=# class=\"btn btn-large btn-follow\">Follow</a>" 
  );
  $templateCache.put('templates/btnLike.html',
    "<a href=# class=\"btn btn-primary\">Like</a>" 
  );
}]);
```

あとはこのjs/templates.jsを

``` HTML
<script type="text/javascript" src="js/templates.js"></script>
```

と読み込めば、btnLike.htmlとbtnFollow.htmlの内容は読み込まれてキャッシュされるので、
使うときは$routeProviderやDirectiveのtemplateUrlで

``` JavaScript
angular.module('myApp')
.directive('btnLike', function() {
    return {
      restrict: 'E',
      templateUrl: 'templates/btnLike.html'
    };
  });
```

という感じで使うか、またはHTMLに直接

``` HTML
<div ng-include="’templates/btnLike.html’"></div>
```

とngIncludeで指定して使うことができます。

### その他オプションを試してみる

もしテンプレートが置いてあるパスと実際にアプリ内で指定するURLが違う場合は、
cwdオプションを使って下記のようにカレントディレクトリを別途指定します。

``` JavaScript
// Gruntfile.js
ngtemplates:  {
  myApp: {
    cwd: 'app', // 実際にtemplatesディレクトリが入っているパス
    src: 'templates/**/*.html', // アプリで使うURL
    dest: 'app/js/templates.js'
  }
}
```

また、テンプレート用JSに変換する際に**htmlmin**を利用することができるので、

``` JavaScript
// Gruntfile.js
ngtemplates:  {
  myApp: {
    src: 'templates/**/*.html',
    dest: 'js/templates.js'、
    options: {
      htmlmin: { // htmlminと同じオプションを指定できる
        collapseBooleanAttributes:      true,
        collapseWhitespace:             true,
        removeAttributeQuotes:          true,
        removeComments:                 true,
        removeEmptyAttributes:          true,
        removeRedundantAttributes:      true,
        removeScriptTypeAttributes:     true,
        removeStyleLinkTypeAttributes:  true
      }
    }
  }
}
```

オプションでhtmlminを指定して出力時のファイルサイズを最適化できました。

grunt-angular-templatesのおかげでテンプレートの外部HTML化が実現し、リクエスト数を抑えることもできました。

**grunt-angular-templates**
[https://github.com/ericclemmons/grunt-angular-templates](https://github.com/ericclemmons/grunt-angular-templates "grunt-angular-templates")
