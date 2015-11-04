title: Gilgamesh を使って UI コンポーネントを拡張してみる
id: 145
categories:
  - JavaScript
  - Design
date: 2015-02-23 01:12:12
tags:
  - AngularJS
---

UI をコンポーネント・ベースで開発していると、コンポーネントを開発した当初は予期していなかったカスタマイズが必要になることがあります。何でもカスタマイズできるような汎用性を持たせられれば、それが一番良いですが、過剰な汎用性はアプリケーションを不必要に重くするだけです。

しかし、アプリケーション固有の UI コンポーネントをある程度の大きさの粒度で作っている場合、必要十分な汎用性を予測することは非常に難しいことが多いため、必要になった時点で機能的な分岐を追加することが多いです。多くの場合、既存でそのコンポーネントを使っている箇所に影響を与えないように、分岐するためのフラグを要素の属性値として渡します。

ある程度の大きさの粒度というのは、例えば投稿フォームのような複数の input 、button 要素をテンプレートの中に持ち、バリデーションや Ajax などの機能を提供するくらいの粒度を想定しています。例えば、この投稿機能を提供する UI を下記のように使っている場合、

``` HTML
<post-form />
```

[![postForm コンポーネント](/images/try-gilgamesh/postForm.png)](/images/try-gilgamesh/postForm.png)

`label` という属性の値として変更したいラベルのテキストを渡すことで、属性値が設定されていた場合だけデフォルトとは違うラベルに変更できるようにします。

``` HTML
<post-form label="まずは試してみる" />
```

[![ラベルを変更した postFomr コンポーネント](/images/try-gilgamesh/SampleLabel.png)](/images/try-gilgamesh/SampleLabel.png)

ただし、変更が必要になったときにこれを繰り返し続けると、次のようなことにもなりかねません。

``` HTML
<post-form
  label="最初の投稿"
  size="small"
  skin="dark"
  is-followed="true"
  stars="12"
  limit="1000"
  …
  …
/>
```

さすがにこの例のような場合はデザイン的にも問題があるとは思いますが、運用を長く続けていると起こり得る自体です。

### Gilgamesh

[Gilgamesh](http://sskyy.github.io/Gilgamesh/ "Gilgamesh") は、[Zhenyu Hou](https://github.com/sskyy "sskyy (Zhenyu Hou)") 氏が開発している JavaScript フレームワークを拡張するライブラリ集です。

JavaScript フレームワークを拡張する、とありますが、現在のところサポートしているフレームワークは、AngularJS のみです。今後 Polymer と React もサポートされる予定みたいです。

Gilgamesh には今のところ大きく２つの機能があります。

1.  テンプレート拡張ライブラリ
2.  データソースライブラリ

今回は、１つ目の「テンプレート拡張ライブラリ」としての機能を利用して AngularJS の `Directive` で作った UI コンポーネントの拡張を試してみます。

### Gilgamesh を試す準備

[Gilgamesh のリポジトリ](https://github.com/sskyy/Gilgamesh.git "sskyy/Gilgamesh") を clone してきて、任意の HTML で jQuery と AngularJS を読み込ませた上で下記の script 追加します。

``` HTML
<script src="./adapters/angular/adapter.js"></script>
<script src="./build/Gilgamesh.js"></script>
<script src="./adapters/angular/directives.js"></script>
```

ちなみに、jQuery を外して実行してみたところ、エラーが出たので現状は jQuery に依存しているのかもしれません。（あまり深く追ってないです。）

### Gilgamesh でコンポーネントを作成する

AngularJS の Directive 機能でコンポーネントを作るのとほぼ同様に、Gilgamesh のコンポーネントを作ることができます。Angular モジュール・オブジェクトの `directive` メソッドを `component` に置き替えるだけです。

``` JavaScript
angular.module('demo')
.component('postForm', function () {
  return {
    templateUrl: './template.html',
    link: function () {}
  };
});
```

これで下記のように `&lt;div post-form&gt;&lt;/div&gt;` をマークアップに配置するとこのコンポーネントを使うことができます。

``` HTML
<div post-form></div>
```

[![postForm コンポーネント](/images/try-gilgamesh/postForm.png)](/images/try-gilgamesh/postForm.png)

本当は、`restrict: 'E'` で、Element Derective にしたかったのですが、現時点では `&lt;div post-form&gt;&lt;/div&gt;` を配置するとエラーが出てうまく動作しませんでした。

### コンポーネントのパーツを書き替える

Gilgamesh で作ったコンポーネントに対して、マークアップ側から変更を加えていきます。中止ボタンだけ別の見た目のボタンに変更します。

まず、コンポーネントで使っているテンプレートの中止ボタン要素に役割名を指定します。ここでは `cancel` という名前を指定します。

``` HTML
<div class="col-xs-2">
  <button class="btn btn-block" ng-click=“user.cancel()"
   gm-role="cancel"
  >中止</button>
</div>
```

次にコンポーネントを使用する側で、中止ボタンだけ置き替えたい要素に上書きする記述をします。コンポーネントの要素に `gm-tpl-partial` という属性を書き加えて、子要素として `gm-role="cancel"` という属性を付けた要素を記述します。これが中止ボタンを上書きする要素になります。 

``` HTML
<div post-form gm-tpl-partial>
  <button class="btn btn-danger btn-block" ng-click=“user.cancel()"
   gm-role="cancel"
  >解除</button>
</div>
```

[![部分的にパーツを上書きした postForm コンポーネント](/images/try-gilgamesh/postFormPartial.png)](/images/try-gilgamesh/postFormPartial.png)

また、部分的にではなく、テンプレート自体を全体的に別のものにしたい場合は、`gm-tpl-partial` 属性を設定しないで、下記のようにコンポーネント要素の中身を上書きするだけで UI 全体が上書きされます。

``` HTML
<div post-form>
  <!-- ここの記述でテンプレートをそっくり書き替える -->
</div>
```

### コンポーネントのパーツを取り除く

部分的に上書きもできれば、部分的に取り除くこともできます。今度は中止ボタンをコンポーネントから取り除きます。`gm-tpl-exclude="cancel"` という属性をコンポーネントの要素に追加するとテンプレートで `gm-role="cancel"` 属性を与えられた要素だけ除外されます。

``` HTML
<div post-form
 gm-tpl-exclude="cancel"
></div>
```

[![postForm からパーツだけ除外](/images/try-gilgamesh/postFormExclude.png)](/images/try-gilgamesh/postFormExclude.png)

### コンポーネントの外にある要素をコンポーネントのパーツとして扱う

例えば、中止ボタンをテンプレートのマークアップ外に配置したい場合は、Gilgamesh のコンポーネントの外にある要素をコンポーネントのパーツとして扱うことができる機能が有効です。まずコンポーネントの要素に `id` を設定します。

``` HTML
<div post-form id="postForm"></div>
```

そして HTML の任意の場所に中止ボタンとして機能させたい要素を `gm-import="postForm"` 属性を加えて配置します。属性値を、先程の id 属性の値と同じにすることで、postForm コンポーネントの外にある要素をコンポーネントの中のものとして扱うことができるようになります。

``` HTML
<button class="btn btn-danger btn-block"
 gm-import="postForm"
></button>
<div post-form id=“postForm”></div>
```

コンポーネントの外にあっても、`gm-import` 属性で紐付けられた要素内は postForm コンポーネントの scope に紐付きます。なので、要素内に書いた Angular 式は postForm のコンテキストで展開されます。

[![postForm コンポーネントに外部から要素を追加](/images/try-gilgamesh/postFormImport.png)](/images/try-gilgamesh/postFormImport.png)

### コンポーネントを拡張する

あるコンポーネントの `link` に設定した機能を継承して別のコンポーネントを作ることができます。`component` メソッドで新しいコンポーネントを作るときに `extend` キーを追加します。値には継承したい親コンポーネントの名前を指定します。

``` JavaScript
angular.module('demo')
.component('postFormSubscribe', function () {
  return {
    extend: 'postForm',
    templateUrl: './template.html',
    link: function (iScope) {
      // postForm の link が先に実行される
      iScope.user.subscription = true;
    }
  };
});
```

これで postForm コンポーネントを継承して追加で `iScope.user.subscription = true;` という処理を postFormSubscribe コンポーネントにだけ実行できます。`iScope.user.subscription = true;` を実行したことにより、postFormSubscribe コンポーネントでは、購読チェックボックスがデフォルトでオンになるようにしました。

[![postForm コンポーネントを拡張して作った postFormSubscribe コンポーネント](/images/try-gilgamesh/postFormExtend.png)](/images/try-gilgamesh/postFormExtend.png)

この拡張機能は残念ながら、現時点で自分が試した範囲では `link` を継承することしかできないようでした。親のテンプレートを継承できるともっと使い道が広がりそうです。

### まだプロダクトでは使えなさそうだが...

Gilgamesh のテンプレート拡張機能を試してみました。少しまだバグが多い印象なのでプロダクトにはまだ導入できないと思っています。しかし、予期できなかったけれど、必要になった汎用性を復活してくれるライブラリとして、機会があれば使ってみたいと思っています。

また本記事の内容については、[こちらのスライド](http://www.slideshare.net/ygoto3q/componentization-with-gilgamesh "Componentization with Gilgamesh")でも同様の内容を話しています。

**Gilgamesh**
[http://sskyy.github.io/Gilgamesh/](http://sskyy.github.io/Gilgamesh/ "grunt-angular-templates")

**Gilgamesh: bring Angular to the next level**
[http://www.reddit.com/r/programming/comments/2s5exu/gilgamesh_bring_angular_to_the_next_level/](http://www.reddit.com/r/programming/comments/2s5exu/gilgamesh_bring_angular_to_the_next_level/ "grunt-angular-templates")
