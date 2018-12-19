# レッスン8. エラー処理

## 目的

- エラー処理の目的を知る。
- エラーオブジェクトについて理解する。
- try...catchを利用したエラー処理を実装する。

## はじめに

実際のアプリケーションでは、予期できるものから予期できないものまで様々なエラーが起こります。Javascriptはシングルスレッドで動いているため、一度エラーが起こるとアプリケーション全体が止まってしまいます。そのため、エラーが起こった場合でも、適切に処理を行いプログラムが動き続けるようにする必要があります。Promiseオブジェクトの持つ、catch関数はこのエラー処理の仕組みの一つです。ここでは、こうしたエラー処理についてより詳しく見ていきましょう。

## strictモード

strictモードは、プログラムをブラウザがより厳しくチェックするモードです。ES5のコードでは`"use strict"`という文字がファイルの先頭やブロックの先頭に書かれていることがよくあります。これはstrictモードを適用したい場合の呼び出し文です。strictモードでは、通常モードではエラーが出ず、後々問題が発生しやすいような事柄に対してエラーを出します。

strictモード適用時にエラーが出るプログラムの例: 

```javascript
function test(test, test) {} // VM601:1 Uncaught SyntaxError: Identifier 'test' has already been declared
/* 
同じ名前の引数をファンクション定義で使っているため、シンタックスエラーが返ります。
*/
```

ES6のモジュールにはデフォルトでstrictモードが適用されるため、呼び出し文を書く必要はありません。ES5のプログラムを見ていて"use strict"の文字を見たら、このstrictモードを適用しているのだ、とだけ理解出来れば大丈夫です。詳しい違いを知りたい方はMDNの[ドキュメント](https://developer.mozilla.org/ja/docs/Web/JavaScript/Strict_mode)をご覧ください。


## throw文

throw文を使うと、エラーが起こった際に自分で定義したオブジェクトや、文字列、ファンクションなど任意のオブジェクトを例外として投げることが出来ます。throw文は通常、この後説明するtry...catch文内で利用します。

- **throw文の使い方**

```javascript
throw "エラーが発生しました。" // 文字列を例外として投げる例

class myException {
  constructor(message) {
    this.message = message;
    this.name = "myException";
  }
  get errorMessage() {
    return `${this.name}: ${this.message}`;
  }    
}

throw new MyException("値が上限値を超えています。"); // クラスインスタンスを例外として投げる例
```

## Errorオブジェクト

Errorオブジェクトは、Javascriptの実行時にエラーが起きた際に例外として投げられます。また上記の例ではmyExceptionというクラスを定義しましたが、このErrorオブジェクトをthrow文で投げることも多いです。

- **Errorオブジェクトの基本的な使い方**

```javascript

let err = new Error("値が上限値を超えています。"); // Errorインスタンスの生成

console.log(err.message); // "値が上限値を超えています。"

throw err； // Errorインスタンスを例外として投げる。
```

## try...catch文

try...catch文は、ある処理に対して、例外が起こった時の処理を併せて定義するための文です。

- **try..catch文のシンタックス**

```javascript
try {
  何らかの処理
} catch(err) {
  例外発生時の処理
} finally {
  成功時も例外発生時も必ず最後に呼び出される処理
}
```

以下では、実際にtry...catch文を使ってみます。

```javascript
function sayName(name) {
  if (typeof name !== "string") {
    throw new Error("名前のフォーマットが異なります。");
  } else {
    return `私の名前は${name}です。`
  }
}

let name = 1
try {
  sayName(name)
} catch(e) {
  console.log(e.message);
} finally {
  console.log("処理完了")
}
/* 実行結果
名前のフォーマットが異なります。
処理完了
*/
```

このエラー処理を使うことで、例えばユーザーがオフラインになって処理が完了しなかった場合は、そのことを画面に表示し、数秒後に自動的に再度処理を行う。というようなことが出来るようになります。

## Async/Awaitファンクションでのエラーハンドリング

Promiseでは、reject状態のPromiseオブジェクトの処理をcatch内で行いました。Async/Awaitの場合は、同じ処理をtry...catch文を利用して行うことが出来ます。今回はレッスン6で作成したPasswordValidatorをAsync/Awaitを利用して書き換えてみましょう。

- **Promiseの例**

```javascript
import BaseValidator from './BaseValidator';

class PasswordValidator extends BaseValidator {
  constructor(val) {
    super(val, 'パスワード');
    this._checkLength = this._checkLength.bind(this);
  }
  validate() {
    return super._cannotEmpty()
      .then(this._checkLength)
      .then((res) => {
        return { success: true }; // Promise.resolve({ success: true })と同一
      })
      .catch(err => {
        return err; // Promise.resolve(err)と同一
      });
  }
  _checkLength() {
    if (this.val.length >= 8) {
      return Promise.resolve();
    } else {
      return Promise.reject({
        success: false,
        message: `パスワードが短すぎます。`
      });
    }
  }
}
```
- **Async/Awaitの例**

```javascript
import BaseValidator from './BaseValidator';

class PasswordValidator extends BaseValidator {
  constructor(val) {
    super(val, 'パスワード');
  }
  async validate() {
    let result;
    try {
      result = await super._cannotEmpty();
    } catch(e) {
      return e;
    }
    try {
      result = await this._checkLength();
    } catch(e) {
      return e;
    }
    return { success: true }
  }
  _checkLength() {
    if (this.val.length >= 8) {
      return Promise.resolve();
    } else {
      return Promise.reject({
        success: false,
        message: `パスワードが短すぎます。`
      });
    }
  }
}
```

Async/Awaitの場合、try...catch文を2回書く必要があるため冗長に感じられた方もいるかもしれません。catchした場合の処理がメソッドによって異なるという場合などでは、try...catch文を使う方が良い場合もありますので状況に分けて使い分けるのが良いかと思います。

## チャレンジ

- [チャレンジ8](./challenge/README.md)

## 更に学ぼう

### 記事で学ぶ

- [制御フローとエラー処理 - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)
- [async function - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function)
- [非同期関数 - Google Developers](https://developers.google.com/web/fundamentals/primers/async-functions?hl=ja)