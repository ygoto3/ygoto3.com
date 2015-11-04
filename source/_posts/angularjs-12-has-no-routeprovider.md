title: $routeProviderがない - AngularJS 1.2.0-rc にアップデートする際の注意
tags:
  - AngularJS
id: 38
categories:
  - JavaScript
date: 2013-11-01 00:25:55
---

2013年11月1日現在、AngularJS最新のstable versionは「1.0.8」、「1.2.0」はRC扱いになっています。

通常はstableな「1.0.8」を使用すればいいのですが、たとえばngAnimateなどの機能は「1.2.0」から提供されているので、ngAnimateを使いたくてアップデートすることもあるかと思います。

### 1.2.0ではngRouteモジュールが本体から分離されている

SPAで開発する場合など$routeProviderサービスを使用してルーティング処理を行っていると思いますが、1.0.8では本体に組み込まれている$routeProviderが**1.2.0ではngRouteモジュールのサービスとして分離されています**。

なので、$routeProviderを使用しているアプリで「1.2.0-rc」にアップデートした場合、下記のようなエラーがコンソールに表示されます。

`Uncaught Error: [$injector:modulerr] Failed to instantiate module angularApp due to:
Error: [$injector:unpr] Unknown provider: $routeProvider`

### 修正

対応方法は簡単で、ngRouteモジュールを提供する**angular-route.js**を別途インストールします。例えばBowerで

``` bash
$ bower install angular-route --save
```

でインストールした後、HTMLファイルでangular-routeを読み込み、

``` HTML
<script src="bower_components/angular-route/angular-route.js"></script>
```

依存するモジュールとして**ngRoute**を追加します。

``` JavaScript
angular.module('myApp', ['ngRoute'])
.config(['$routeProvider', function($routeProvider) {
  // ...
}]);
```

これでエラーは消えて正常な動作に戻るはずです。
