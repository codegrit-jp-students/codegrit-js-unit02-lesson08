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

## 更に学ぼう

### 記事で学ぶ

- [制御フローとエラー処理 - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)
- [async function - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function)
- [非同期関数 - Google Developers](https://developers.google.com/web/fundamentals/primers/async-functions?hl=ja)