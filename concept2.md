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