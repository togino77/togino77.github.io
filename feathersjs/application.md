# FeatchersJS の ソースコードを読んでみる。application 編

## 概要

`Application` インスタンスは、サービスやフック、プラグインの設定やオプション設定を管理し、初期設定されたアプリケーションは `app object` として参照される。最小限の `app object` は以下のようにして用意することができる。

```Javascript
const feathers = require('@feathersjs/feathers');
const app = feathers();
```

今回は、この `app` が生成されるまでのソースコードを読んでみる。

## 流れ

## ソースコード

```javascript
const feathers = require('@feathersjs/feathers')
```

この `require` により [feathers/lib/index.js](https://github.com/feathersjs/feathers/blob/master/packages/feathers/lib/index.js) でデフォルトエクスポートされる関数を定数 `feathers` に代入する

```javascript
$ less feathers/lib/index.js
const Proto = require('uberproto');
const Application = require('./application');
const baseObject = Object.create(null);

function createApplication () {
  const app = Object.create(baseObject);
  // Mix in the base application
  Proto.mixin(Application, app);
  app.init();
  return app;
}

module.exports = createApplication;
```

つまり、`feathers` は `createApplication` 関数であり、これは `app object` を返すものである。`app` オブジェクトは

1. `Object.create()` で生成された空のオブジェクトに
2. `Application` を mixin し
3. mixin された `init()` を呼び出して初期化したもの

といえる。ちなみに、最初に生成された空のオブジェクトというのは、`Object.create(null)` で生成されたものをプロトタイプとして `Object.craete` したものなので、いわゆる一般的な`Object` からの継承を断ち切っている。

また、`Application` の mixin は、`uberproto` ライブラリを用いて行なっている。

アプリケーションの核は、mixin した `Application` であり、これは `require('./application')` したものである。

```javascript
$ less feathers/lib/application.js
const application = {
  init () {
    Object.assign(this, {
      version,
      methods: [
        'find', 'get', 'create', 'update', 'patch', 'remove'
      ],
      mixins: [],
      services: {},
      providers: [],
      _setup: false,
      settings: {}
    });

    this.configure(hooks());
    this.configure(events());
  },
  ......
};
module.exports = application;
```

つまり、この `application` がアプリケーションの本体となる。空のオブジェクトに mixin された後、初期化するために呼び出される `init()` で、アプリケーションオブジェクトのプロパティーとなる `methods` や `mixins`、　` services` などを設定し、フックやイベントの初期設定を行なっている。