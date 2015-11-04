title: AngularJS の Controller / Service / Directive / Filter 役割のポイント
id: 91
categories:
  - JavaScript
date: 2014-08-29 13:24:13
tags:
  - AngularJS
---

会社内で AngularJS の Working Group を作り活動している中で、よく上がる質問の１つが AngularJS における Controller / Service / Directive / Filter に書く処理をどう分けたらいいのか、でした。

本記事では、Controller / Service / Directive / Filter の役割のポイントを整理したいと思います。

### 基本ポイント

他の MVC デザインパターンと同様、ビジネスロジックとプレゼンテーションロジックを分離することが基本です。

ビジネスロジックを Service が担当し、Controller でそれを紐付けてテンプレートに共有します。プレゼンテーションロジックは Directive と Filter が担当し、DOM 操作処理やデータ整形処理をテンプレートに共有します。

#### Controller

Controller ではビューで表示するデータとユーザーアクションに対するメソッドを定義します。

AngularJS では `$scope` オブジェクトを介してデータやメソッドをテンプレートで共有することができます。Controller には、ビジネスロジックを `$scope` オブジェクトに書き込んでいき、テンプレートでは `$scope` オブジェクトで共有されたデータとメソッドを参照します。

共有される `$scope` オブジェクトがカオスな状態になるのを避けるために、Controller では `$scope` オブジェクトを書き込み専用として、テンプレートでは読み取り専用として扱うと良いです。

また、直接 DOM を参照することなどは行わなないようにします。DOM 操作処理が Controller に入ってしまうと、デザイン変更などでテンプレートの HTML に変更を行わなければいけない場合に、Controller も書き変える必要が出てくる可能性があります。

そのため、DOM 操作処理は Controller 内では極力行わなず、Directive にその役目を渡しましょう。

ビジネスロジックとプレゼンテーションロジックを Service、Directive、Filter に分離して Controller を簡潔に保つことができると理想的です。

#### Service

ビジネスロジック担当です。ビューに依存しない処理を記述します。

また、各 Service はシングルトンとして存在するため、異なる Controller や Directive 間で共有するモデルとして使用できます。

#### Directive

HTML を拡張する機能です。前述の DOM 操作が必要になる場合を含み、プレゼンテーションロジックを記述します。

自身で Controller を持ち、単一で完結するコンポーネントを作ることもできますし、属する scope で公開されているデータと振舞いをテンプレートに紐付ける役割を担います。

#### Filter

データを整形する処理を記述します。

Directive と同様にプレゼンテーションロジックを記述しますが、Directive と違い、直接 Scope にアクセスすることはできません。モデルを変更することなく表示フォーマットのみを変更します。

### サンプル

ここでは、フォームから ユーザーデータを追加する処理を例に説明します。

この例では、`FormCtrl` という Controller、`noHyphen` という Directive、`User` という Service を組み合わせて実装しています。

まずモジュールを宣言します。

``` JavaScript
var app = angular.module('app', []);
```

#### テンプレート

``` HTML
<body ng-app="app">
  <form name="registrationForm" ng-controller="FormCtrl" novalidate>
    <input type="text" ng-model="user.nickname" required no-hyphen />
    <button type="submit" name="nickname" ng-click="submit()"
  ng-disabled="registrationForm.$invalid">送信</button>
  </form>
</body>
```

このようなテンプレート用意した場合、各々の役割は以降のようになります。

#### Controller に書く実装

``` JavaScript
app.controller('FormCtrl', function ($scope, $log, User) {
  $scope.submit = function () {
    User.addUser($scope.user)
    .then(
      function (resource) {
        $log.log(resource);
      },
      function (err) {
        $log.warn(err);
      }
    );
  };
});
```

ここでのポイントは `$scope` オブジェクトの設定だけを記述している点です。DOM にイベントハンドラを紐付ける `$('button').on('click', function () { ... })` などの処理はビルトインの Directive である `ngClick` に任せてあります。

また、バリデーション機能は、特定の DOM の値を取得する必要があるため、Controller 内には記述しません。後述する `noHyphen` という Directive を実装して機能を実現します。

このフォームは送信ボタンを押された時に Ajax 処理も実行しますが、その処理も後述する `User` という Service に実装を切り分けています。

#### Directive に書く実装

``` JavaScript
app.directive('noHyphen', function () {
  return {
    require: 'ngModel',
    link: function (iScope, iElem, iAttr, ngModelCtrl) {
      ngModelCtrl.$parsers.push(function (viewVal) {
        var _isValid = true;

        if (~viewVal.indexOf('-')) {
          _isValid = false;
        }

        ngModelCtrl.$setValidity('noHyphen', _isValid)
      });
    }
  };
});
```

ここでのポイントは、`ngModel` を介して DOM 操作をしている点です。Controller で必要だった DOM にイベントハンドラを紐付ける処理は Directive に記述します。

処理した結果を `ngModel` ディレクティブを介して `FormCtrl` の `$scope` に渡しています。

#### Service に書く実装

今回は、`factory` メソッドを使用します。

``` JavaScript
app.factory('User', function ($http) {
  var _onSuccess = function (res) {
        return res.data;
      },
      _onError = function (res) {
        return $q.reject('an error occured.');
      },
      _addUser = function (user) {
        var request = $http({
          method: 'post',
          url: '/api/something',
          params: {
            action: 'add'
          },
          data: {
            nickname: user.nickname
          }
        });
        return request.then(_onSuccess, _onError);
      };

   return {
     addUser: _addUser
   };
});
```

ここでのポイントは、ビューに依存しない処理のみを記述している点です。

ここでは、新規ユーザーデータの送信に使われる Ajax 処理を実装しているので、`FormCtrl` はこの `User` Service をインジェクトすることで自身の `$scope` オブジェクトに持っているデータを送信することができます。

特有の概念が多いため AngularJS での役割の分担は分かりづらいですが、自分は上記のように処理を分けるようにしています。
