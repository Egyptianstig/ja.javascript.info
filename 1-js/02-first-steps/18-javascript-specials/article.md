# JavaScript スペシャル（これまでのおさらい）

このチャプターでは、微妙なケースに注意を払いながら、私たちが今まで学んだJavaScriptの機能を簡単に再確認します。

## コード構造 

文はセミコロンで区切られます:

```js run no-beautify
alert('Hello'); alert('World');
```

通常、行の終わりは区切りとして扱われますので、これは動作します:

```js run no-beautify
alert('Hello')
alert('World')
```

これは "自動セミコロン挿入" と呼ばれます。ときどき、これは動作しません。例えば:

```js run
alert("There will be an error after this message")

[1, 2].forEach(alert)
```

ほとんどのコードスタイルのガイドは、各文の後にセミコロンを置くことに賛同しています。

セミコロンはコードブロック `{...}` や、ループのような構文構造の後では必要ありません:

```js
function f() {
  // 関数宣言のあとにセミコロンは不要です
}

for(;;) {
  // ループの後にセミコロンは不要です
}
```

...しかし、"余分な" セミコロンを任意の場所に置いたとしても、それはエラーではありません。それは無視されます。

より詳細はこちら: <info:structure>.

## Strict モード 

現在のJavaScriptのすべての機能を完全に有効にするには、`"use strict"` でスクリプトを始める必要があります。

```js
'use strict';

...
```

そのディレクティブはスクリプトの先頭、もしくは関数の最初である必要があります。

`"use strict"` がなくても動作しますが、幾つかの機能が古いスタイル, "互換の方法" で振る舞います。私たちは一般的に現代の振る舞いを好みます。

言語に最新の機能のいくつか（今後学ぶクラスなど）は暗示的に strict モードを有効にします。

より詳細はこちら: <info:strict-mode>.

## 変数 

これらを使って定義できます:

- `let`
- `const` (定数, 変更できない)
- `var` (古いスタイル, あとで見ます)

変数の名前は次を含むことができます:
- 文字と数字、しかし1文字目に数字は指定できません。
- 文字 `$` と `_` は普通の文字です。
- 非ラテンのアルファベットや象形文字も使えますが、一般的には使用されません。

変数は動的に型付けされます。 それらは任意の値を格納することができます:

```js
let x = 5;
x = "John";
```

7つのデータ型があります:

- `number` 浮動少数点と整数値両方
- `bigint` 任意の長さの整数値
- `string` 文字列
- `boolean` 論理値: `true/false`
- `null` -- 単一の値 `null` を持つ型。"空白", "存在しない" を意味する
- `undefined` -- 単一の値 `undefined` を持つ型。"未割り当て" を意味する
- `object` と `symbol` -- 複雑なデータ構造やユニークな識別子です。私たちはまだそれらは学んでいません。

`typeof` 演算子は値の型を返します。2つ例外があります:
```js
typeof null == "object" // 言語の間違い
typeof function(){} == "function" // 関数は特別に扱われます
```

より詳細はこちらです: <info:variables> and <info:types>.

## インタラクション 

私たちは動作環境としてブラウザを使っているので、基本のUI関数は次の通りです:

[`prompt(question[, default])`](mdn:api/Window/prompt)
: `question` を訪ね、訪問者の入力もしくは、"cancel" が選択されたときは `null` を返します。

[`confirm(question)`](mdn:api/Window/confirm)
: `question` を訪ね、OKかキャンセルを選択させます。選択は `true/false` として返却されます。

[`alert(message)`](mdn:api/Window/alert)
: `message` を出力します。

それらの関数はすべて *モーダル* であり、コードの実行を止め、訪問者が回答するまでそのページとのやり取りを防ぎます。

例えば:

```js run
let userName = prompt("Your name?", "Alice");
let isTeaWanted = confirm("Do you want some tea?");

alert( "Visitor: " + userName ); // Alice
alert( "Tea wanted: " + isTeaWanted ); // true
```

より詳細はこちらです: <info:alert-prompt-confirm>.

## 演算子 

JavaScriptは次のような演算子をサポートします:

算術
: 通常の四則演算の `* + - /`、また剰余として `%`、冪乗として `**`。

    二項演算子プラス `+` は文字列を連結します。また、オペランドのいずれかが文字列であれば、もう一方も文字列に変換されます:

    ```js run
    alert( '1' + 2 ); // '12', string
    alert( 1 + '2' ); // '12', string
    ```

代入
: 単純な代入の `a = b` と `a *= 2` のような他の演算子と組み合わせたものがあります。

ビット単位
: ビット演算子はビットレベルで整数を扱います。必要なときに、[docs](mdn:/JavaScript/Reference/Operators/Bitwise_Operators)を見てください。

3項
: 3つのパラメータを持つ唯一の演算子です: `cond ? resultA : result B`. `cond` が真の場合、`resultA` を返し、そうでなければ `resultB` を返します。

論理演算子
: 論理積 `&&` と 論理和 `||` は短絡評価を行い、それが停止したところの値(`true`/`false` である必要はありません)を返します。論理否定 `!` はオペランドをブール型に変換し、その逆の値を返します。

NULL合体演算子
: `??` 演算子は変数のリストから定義済みの値を選択する方法として提供されています。`a ?? b` の結果は `a` が `null/undefined` の場合は `b`、そうでなければ `a` です。

比較
: 異なる型の値のための等価チェック `==` は、それらを数値に変換します(`null` と `undefined`を除きます。それらは、お互いに等しく、他とは等しくなりません)。従って以下は等価です。:

    ```js run
    alert( 0 == false ); // true
    alert( 0 == '' ); // true
    ```

    他の比較も同様に数値に変換します。

    厳密等価演算子 `===` は変換を行いません: 異なる型は常に異なる値を意味します。:

    値 `null` と `undefined` は特別です: それらはお互いに等価 `==` であり、それ以外と等しくありません。

    より大きい/少ない演算子は文字列を1文字ずつ比較し、他の型は数値に変換します。

その他
: 他にもカンマ演算子などがあります。

より詳細はこちらです: <info:operators>, <info:comparison>, <info:logical-operators>.

## ループ 

- 私たちは3つのタイプのループを説明しました:

    ```js
    // 1
    while (condition) {
      ...
    }

    // 2
    do {
      ...
    } while (condition);

    // 3
    for(let i = 0; i < 10; i++) {
      ...
    }
    ```

- `for(let...)` ループの中で宣言された変数はループの内側でのみ見えます。しかし、`let` を省略することができ、既存の変数を再利用することも出来ます。
- ディレクティブ `break/continue` はループ全体/現在のイテレーションを終了させることができます。ネストされたループを停止する場合にはラベルを使ってください。

詳細はこちらです: <info:while-for>.

今後、オブジェクトを扱うためのより多くの種類のループを学びます。

## "switch" 構造 

"switch" 構造は複数の `if` チェックに置換できます。それは比較に `===` を使います。

例えば:

```js run
let age = prompt('Your age?', 18);

switch (age) {
  case 18:
    alert("Won't work"); // プロンプトの結果は文字列であり、数値ではありません

  case "18":
    alert("This works!");
    break;

  default:
    alert("Any value not equal to one above");
}
```

詳細はこちらです: <info:switch>.

## 関数 

私たちは、JavaScriptで関数を作る3つの方法をカバーしました。:

1. 関数宣言: メインコードフローの中の関数

    ```js
    function sum(a, b) {
      let result = a + b;

      return result;
    }
    ```

2. 関数式: 式のコンテキストにある関数

    ```js
    let sum = function(a, b) {
      let result = a + b;

      return result;
    }
    ```

3. アロー関数:

    ```js
    // 右側の式です
    let sum = (a, b) => a + b;

    // もしくは { ... } を使った複数行の構文で、return が必要です:
    let sum = (a, b) => {
      // ...
      return a + b;
    }

    // 引数なし
    let sayHi = () => alert("Hello");

    // 1つの引数
    let double = n => n * 2;
    ```


- 関数はローカル変数を持ちます: それらはその関数本体の中で宣言されます。このような変数は関数の中でだけ見えます。
- パラメータはデフォルト値を持つことが出来ます。: `function sum(a = 1, b = 2) {...}`.
- 関数は常に何かを返します。もしも `return` 文がない場合は　`undefined` を返します。

詳細はこちら: <info:function-basics>, <info:function-expressions-arrows>.

## これからが本番です 

ここまではJavaScriptの機能の簡単な一覧でした。今のところ、私たちは基本だけを学びました。このチュートリアルではさらに、JavaScriptのより特別で高度な機能について説明していきます。
