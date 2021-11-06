# 残りのパラメータ(Rest parameters)とスプレッド演算子(Spread operator)

JavaScript の組み込み関数の多くは、任意の数の引数をサポートしています。

例:

- `Math.max(arg1, arg2, ..., argN)` -- 引数の中で最大のものを返します。
- `Object.assign(dest, src1, ..., srcN)` -- `src1..N` のプロパティを `dest` にコピーします。
- など.

このチャプターでは、その方法を説明します。そして、より重要なことは、そのような関数や配列を快適に扱う方法です。

## 残りのパラメータ '...'  

関数は、どのように定義されたかに関係なく、任意の数の引数で呼ぶことができます。

このような感じです:
```js run
function sum(a, b) {
  return a + b;
}

alert( sum(1, 2, 3, 4, 5) );
```

"必要以上の" 引数でもエラーにはなりません。 しかし、結果はもちろん最初の2つだけが使われます。

残りのパラメータは、3つのドット `...` を持つ関数定義で記述できます。これは文字通り、"残りのパラメータを配列にまとめる" という意味です。

例えば、すべての引数を配列 `args` に集める場合は次のようになります:

```js run
function sumAll(...args) { // args は配列の名前です
  let sum = 0;

  for (let arg of args) sum += arg;

  return sum;
}

alert( sumAll(1) ); // 1
alert( sumAll(1, 2) ); // 3
alert( sumAll(1, 2, 3) ); // 6
```

最初のパラメータを変数として取得するようにし、残りだけを集めることもできます。

ここでは、最初の2つの引数が変数に入り、残りは `titles` 配列に格納されます:

```js run
function showName(firstName, lastName, ...titles) {
  alert( firstName + ' ' + lastName ); // Julius Caesar

  // 残りは titles 配列に入ります
  // i.e. titles = ["Consul", "Imperator"]
  alert( titles[0] ); // Consul
  alert( titles[1] ); // Imperator
  alert( titles.length ); // 2
}

showName("Julius", "Caesar", "Consul", "Imperator");
```

````warn header="残りのパラメータは最後である必要があります"
残りのパラメータはすべての残っている引数をまとめるため、下記は意味をなさず、エラーとなります。:

```js
function f(arg1, ...rest, arg2) { // arg2 after ...rest ?!
  // error
}
```

`...rest` は常に最後です。
````

## "arguments" 変数

インデックスによってすべての引数を含む `arguments` と言う特別な配列ライクなオブジェクトもあります。

例:

```js run
function showName() {
  alert( arguments.length );
  alert( arguments[0] );
  alert( arguments[1] );

  // 反復可能(iterable) です。
  // for(let arg of arguments) alert(arg);
}

// 表示: 2, Julius, Caesar
showName("Julius", "Caesar");

// 表示: 1, Ilya, undefined (2つ目の引数がないので)
showName("Ilya");
```

以前は、残りのパラメータは言語仕様に存在しませんでした。そして、`arguments` は引数の数に関係なく、関数に指定されたすべての引数を取得するための唯一の方法でした。そして、これは今でも動作するので使うことができます。

が、デメリットは `arguments` は配列ではなく、配列ライクで反復可能という点です。従って、配列メソッドをサポートしないので、 `arguments.map(...)` 呼び出しをすることはできません。

また、すべての引数が常に含まれているため、残りのパラメータのように部分的に取り込むことはできません。

したがって、これらの機能が必要な場合には "残りのパラメータ" が好ましいです。

````smart header="アロー関数は `\"arguments\"` を持ちません"
もしもアロー関数から `arguments` オブジェクトにアクセスすると、外部の "通常の" 関数からそれらを取得します。

ここではその例です:

```js run
function f() {
  let showArg = () => alert(arguments[0]);
  showArg();
}

f(1); // 1
```

以前学んだように、アロー関数は自身の `this` を持ちません。そして、特別な `arguments` オブジェクトも持っていないことが分かります。
````

## スプレッド 演算子 

先ほど、パラメータのリストから配列を取得する方法をみました。

一方、我々はその逆を正確に行う必要がある場合があります。

例えば、リストから最大値を返す組み込み関数 [Math.max](mdn:js/Math/max) です。

```js run
alert( Math.max(3, 5, 1) ); // 5
```

今、配列 `[3 ,5, 1]` を持っているとします。それを使って `Math.max` を呼び出すにはどうやるでしょう？

それを "そのまま" 渡すと上手く動きません。なぜなら `Math.max` は数値の引数のリストを期待しており、単一の配列がくることは期待していません。:

```js run
let arr = [3, 5, 1];

*!*
alert( Math.max(arr) ); // NaN
*/!*
```

...そして、おそらくコードの中で手動で項目をリストする(`Math.max(arg[0], arg[1], arg[2])`)こともできません。なぜなら ,
それがどれだけあるか分からないからです。スクリプトが動くとき、もっと多いかもしれないし、全くないかもしれません。また、その書き方は格好悪いです。

*スプレッド演算子 (Spread operator)* はその場合に役立ちます。残りのパラメータと似ており、`...` を使いますが、全く反対のことをします。

関数呼び出しで `...arr` が使われると、反復可能なオブジェクト `arr` を引数のリストに展開します。

`Math.max` の場合:

```js run
let arr = [3, 5, 1];

alert( Math.max(...arr) ); // 5 (配列を引数のリストに変換する)
```

また、この方法で複数の iterables を渡すこともできます:

```js run
let arr1 = [1, -2, 3, 4];
let arr2 = [8, 3, -8, 1];

alert( Math.max(...arr1, ...arr2) ); // 8
```

通常の値とスプレッド演算子を組み合わせることもできます。:


```js run
let arr1 = [1, -2, 3, 4];
let arr2 = [8, 3, -8, 1];

alert( Math.max(1, ...arr1, 2, ...arr2, 25) ); // 25
```

さらにスプレッド演算子は配列をマージするために使うこともできます:

```js run
let arr = [3, 5, 1];
let arr2 = [8, 9, 15];

*!*
let merged = [0, ...arr, 2, ...arr2];
*/!*

alert(merged); // 0,3,5,1,2,8,9,15 (0, then arr, then 2, then arr2)
```

上の例では、スプレッド演算子のデモをするために配列を使いましたが、任意の iterable で動作します。

例えば、ここではスプレッド演算子を使って、文字列を文字の配列にします:

```js run
let str = "Hello";

alert( [...str] ); // H,e,l,l,o
```

スプレッド演算子は内部的にはイテレータを使用して要素を集めます。これは `for..of` と同じ方法です。

従って、文字列の場合、 `for..of` は文字を返すので `...str` は `"H","e","l","l","o"` になります。文字のリストは配列初期化子 `[...str]` に渡されます。

この特定のタスクでは、`Array.from` を使うこともできます。それは iterable(文字列など) を配列に変換するからです:

```js run
let str = "Hello";

// Array.from は iterable を配列に変換します
alert( Array.from(str) ); // H,e,l,l,o
```

結果は `[...str]` と同じです。

しかし `Array.from(obj)` と `[...obj]` には微妙な違いがあります:

- `Array.from`は配列ライクと iterables の両方で動作します。
- スプレッド演算子は iterables でのみ動作します。

従って、何かを配列に変換するタスクにおいては、`Array.from` がより適しています。


## 配列/オブジェクトのコピー

[以前](info:object-copy#cloning-and-merging-object-assign) `Object.assign()` について話したのを覚えていますか？

スプレッド構文を使用して同じことをすることができます。

```js run
let arr = [1, 2, 3];

*!*
let arrCopy = [...arr]; // 配列をパラメータのリストに展開します
                        // その後、結果を新しい配列に格納します
*/!*

// 配列が同じコンテンツを持っているか？
alert(JSON.stringify(arr) === JSON.stringify(arrCopy)); // true

// 配列は等価か？
alert(arr === arrCopy); // false (同じ参照ではないため)

// 最初の配列を修正しても、コピーは修正されません:
arr.push(4);
alert(arr); // 1, 2, 3, 4
alert(arrCopy); // 1, 2, 3
```

オブジェクトのコピーを作成するために、同じことを行うことが可能であることにも注目です:

```js run
let obj = { a: 1, b: 2, c: 3 };

*!*
let objCopy = { ...obj }; // オブジェクトをパラメータのリストに展開します
                          // その後、結果を新しいオブジェクトに結果を返します
*/!*

// オブジェクトは同じコンテンツを持っているか？
alert(JSON.stringify(obj) === JSON.stringify(objCopy)); // true

// オブジェクトは等価か？
alert(obj === objCopy); // false (同じ参照ではありません))

// 最初のオブジェクトを修正しても、コピーは修正されません:
obj.d = 4;
alert(JSON.stringify(obj)); // {"a":1,"b":2,"c":3,"d":4}
alert(JSON.stringify(objCopy)); // {"a":1,"b":2,"c":3}
```

このオブジェクトコピーの方法は、`let objCopy = Object.assign({}, obj)` 、あるいは、配列の `let arrCopy = Object.assign([], arr)` よりもはるかに短いです。そのため、可能な場合はこちらの方が好まれます。


## サマリ 

コード上に `"..."` がある場合、それは残りのパラメータかスプレッド演算子です。

それらを区別する簡単な方法があります:

- `...` が関数パラメータの最後にある場合、それは "残りのパラメータ" で、リストの残りを配列に集めます。
- 関数呼び出しなどで `...` があると、それは "スプレッド演算子" と呼ばれ、配列をリストに展開します。

利用パターン:

- 残りのパラメータは、任意の数の引数を受け入れる関数を作成する際に使用されます。
- スプレッド演算子は、通常、多くの引数のリストを必要とする関数に配列を渡す際に使用されます。

共にそれらはリストとパラメータの配列を簡単にやり取りするのに役立ちます。

関数呼び出しのすべての引数は "昔のスタイル" `arguments`(配列ライクな反復可能オブジェクト) も利用可能です。
