
# __proto__ なしでの プロトタイプ メソッド, オブジェクト

このセクションの最初の最初の章では、プロトタイプ をセットアップする現代のメソッドを説明します。

`__proto__` は古く、やや非推奨とされています（JavaScript標準のブラウザのみの部分で）。

現代のメソッドは次のものです:

- [Object.create(proto[, descriptors])](mdn:js/Object/create) -- 与えられた `proto` を `[[Prototype]]` として、また任意のプロパティディスクリプタで空のオブジェクトを作ります。
- [Object.getPrototypeOf(obj)](mdn:js/Object.getPrototypeOf) -- `obj` の `[[Prototype]]` を返します。
- [Object.setPrototypeOf(obj, proto)](mdn:js/Object.setPrototypeOf) -- `obj` の `[[Prototype]]` に `proto` をセットします。

これらは `__proto__` の代わりに使用します。

例:

```js run
let animal = {
  eats: true
};

// animal をプロトタイプとして新しいオブジェクトを作成する
*!*
let rabbit = Object.create(animal);
*/!*

alert(rabbit.eats); // true

*!*
alert(Object.getPrototypeOf(rabbit) === animal); // rabbit のプロトタイプを取得
*/!*

*!*
Object.setPrototypeOf(rabbit, {}); // rabbit のプロトタイプを {} に変更
*/!*
```

`Object.create` は任意で2つ目の引数を持ち、これはプロパティディスクリプタです。次のように、新しいオブジェクトに追加のプロパティが指定できます :

```js run
let animal = {
  eats: true
};

let rabbit = Object.create(animal, {
  jumps: {
    value: true
  }
});

alert(rabbit.jumps); // true
```

ディスクリプタはチャプター <info:property-descriptors> で説明したのと同じフォーマットです。

`for..in` でプロパティをコピーするよりも、より強力にオブジェクトをクローンするのに `Object.create` が使用できます。:

```js
let clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj));
```

この呼び出しはすべてのプロパティを含む `obj` の本当の正確なコピーを作ります。: 列挙型、非列挙型、データプロパティ、setter/getter -- すべてと、正確な `[[Prototype]]` です。

## 略史 

`[[Prototype]]` を管理する方法はたくさんあります! 同じことをするたくさんの方法があります!

なぜでしょう？

それは歴史的な理由によるものです。

- コンストラクタ関数の `"prototype"` プロパティはとても古くから機能しています。
- 2012年後半: `Object.create` が標準に登場しました。これは与えられたプロトタイプでオブジェクトを作りますが、それを取得/設定することはできませんでした。そのため、ブラウザはいつでもプロトタイプの取得/設定ができる非標準の `__proto__` アクセサを実装しました。
- 2015年後半: `Object.setPrototypeOf` と `Object.getPrototypeOf` が標準に追加されました。`__proto__` はどこでも実行されているデファクトであったため、標準の付録Bに追加されています。これはブラウザ以外の環境ではオプションです。

今現在、私たちはこれらのすべての方法を自由に使うことができます。

なぜ `__proto__` は関数 `getPrototypeOf/setPrototypeOf` に置き換えられたのでしょうか？これは興味深い質問であり、なぜ `__proto__` ではダメなのかを理解する必要があります。答えを知るには続きを読んでください。

```warn header="速度が重要な場合は、既存のオブジェクトの `[[Prototype]]` を変更しないでください"
技術的には、いつでも `[[Prototype]]` の取得/設定が可能です。しかし、通常はオブジェクト作成時に一度だけ設定を行い、変更はしません。`rabbit` は `animal` から継承しており、それは変更しません。

また、JavaScriptエンジンは高度に最適化されています。`Object.setPrototypeOf` または `obj.__proto__=` で "その場で" プロトタイプを変更することは、可能ですがとても遅い操作になります。そのため、何をしようとしているかわかっている場合、または JavaScript の速度が全く問題にならない場合以外は避けてください。
```

## "非常にシンプルな" オブジェクト 

ご存知の通り、オブジェクトはキー/値ペアを格納するための連想配列として使うことができます。

...しかし、もしその中で *ユーザから提供された* キーを格納しようとした場合(例えばユーザが入力した辞書)、興味深い問題が起こります。: すべてのキーは `"__proto__"` を除いてうまく動作します。

例を確認してみましょう:

```js run
let obj = {};

let key = prompt("What's the key?", "__proto__");
obj[key] = "some value";

alert(obj[key]); // [object Object], "some value" ではありません!
```

ここでは、ユーザが `__proto__` を入力した場合、その代入は無視されます!

それは驚くことではありません。`__proto__` プロパティは特別です: それはオブジェクトまたは `null` であり、文字列はプロトタイプにはなれません。

しかし、このような振る舞いを実装するつもりはありませんでした。私たちはキー/値ペアを格納したいですが、キー名が `"__proto__"` の場合は正しく保存されませんでした。なので、これはバグです。

ここでの結果はひどくはありませんが、他のケースでは、プロトタイプは実際に変更される可能性があるため、処理が予期しない方向で間違ってしまう可能性があります。

最悪なのは、通常開発者はこのような可能性について全く考えません。そのため、このようなバグに気付きにくくなり、特にJavaScriptがサーバー側で使用されている場合には、それらを脆弱性に変えることさえあります。

また、デフォルトの関数である、`toString` や他の組み込みメソッドに代入する場合も予期しないことが起こる可能性があります。

どうやってこの問題を回避しましょう？

まず、通常のオブジェクトの代わりに、格納域として `Map` を使用するよう切り替える方法があります。それですべて問題ありません。

ですが、`Object` もまたここでは上手く機能します

`__proto__` はオブジェクトのプロパティではなく、`Object.prototype` のアクセサプロパティです。:

![](object-prototype-2.svg)

なので、もし `obj.__proto__` が読み込まれたり代入された場合、該当の getter/setter がそのプロトタイプから呼ばれ、それは `[[Prototype]]` の取得/設定をします。

最初に言ったとおり、`__proto__` は `[[Prototype]]` にアクセスする方法であり、`[[Prototype]]` 自身ではありません。

今、連想配列としてオブジェクトを使いたい場合、リテラルのトリックを使ってそれを行う事ができます。:

```js run
*!*
let obj = Object.create(null);
*/!*

let key = prompt("What's the key?", "__proto__");
obj[key] = "some value";

alert(obj[key]); // "some value"
```

`Object.create(null)` はプロトタイプなし(`[[Prototype]]` が `null`)の空オブジェクトを作ります。:

![](object-prototype-null.svg)

したがって、`__proto__` に対する継承された getter/setter はありません。今や通常のデータプロパティとして処理されますので、上の例は正しく動作します。

このようなオブジェクトを "非常にシンプルな" または "純粋な辞書オブジェクト" と呼びます。なぜなら、それらは通常のオブジェクト `{...}` よりもシンプルなためです。

欠点は、そのようなオブジェクトには組み込みのオブジェクトメソッドがないことです。 `toString` など:

```js run
*!*
let obj = Object.create(null);
*/!*

alert(obj); // Error (no toString)
```

...しかし、連想配列ではそれは通常問題ありません。

ほとんどのオブジェクトに関連したメソッドは `Object.keys(obj)` のように `Object.something(...)` であることに注目してください。それらはプロトタイプにはないので、このようなオブジェクトで機能し続けます。:


```js run
let chineseDictionary = Object.create(null);
chineseDictionary.hello = "ni hao";
chineseDictionary.bye = "zai jian";

alert(Object.keys(chineseDictionary)); // hello,bye
```

## サマリ 

プロトタイプをセットアップしたり直接アクセスするための現代のメソッドは次の通りです:

- [Object.create(proto[, descriptors])](mdn:js/Object/create) -- 与えられた `proto` を `[[Prototype]]` (`null` もOK)として、また任意のプロパティディスクリプタで空のオブジェクトを作ります。
- [Object.getPrototypeOf(obj)](mdn:js/Object.getPrototypeOf) -- `obj` の `[[Prototype]]` を返します( `__proto__` の getter と同じです)。
- [Object.setPrototypeOf(obj, proto)](mdn:js/Object.setPrototypeOf) -- `obj` の `[[Prototype]]` に `proto` をセットします( `__proto__` の setter と同じです)。

組み込みの `__proto__` の getter/setter はユーザが作成したキーをオブジェクトに入れる場合に安全ではありません。ユーザがキーとして `"__proto__"` を入力する可能性があり、その場合エラーが発生し、通常は予期せぬ結果になるでしょう。

なので、`Object.create(null)` を使用して `__proto__` をもたない "非常にシンプルな" オブジェクトを作成するか、`Map` オブジェクトを使用します。

また、`Object.create` はオブジェクトのすべてのディスクリプタの浅いコピー（shallow-copy）を作成する簡単な方法です。

```js
let clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj));
```

さらに、`__proto__` は `[[Prototype]]` の getter/setterであり、他のメソッドと同様に `Object.prototype` に存在することも明らかにしました。

私たちは、`Object.create(null)` によってプロトタイプなしのオブジェクトを作ることができます。このようなオブジェクトは "純粋な辞書" として使われ、キーとして `"__proto__"` の問題はありません。

他のメソッドです:

- [Object.keys(obj)](mdn:js/Object/keys) / [Object.values(obj)](mdn:js/Object/values) / [Object.entries(obj)](mdn:js/Object/entries) -- 列挙可能な自身の文字列プロパティ名/値/キー値ペアの配列を返します。
- [Object.getOwnPropertySymbols(obj)](mdn:js/Object/getOwnPropertySymbols) -- すべての自身のシンボリックプロパティ名の配列を返します。
- [Object.getOwnPropertyNames(obj)](mdn:js/Object/getOwnPropertyNames) -- すべての自身の文字列プロパティ名を返します。
- [Reflect.ownKeys(obj)](mdn:js/Reflect/ownKeys) -- すべての自身のプロパティ名の配列を返します。
- [obj.hasOwnProperty(key)](mdn:js/Object/hasOwnProperty): それは `obj` 自身が `key` という名前のプロパティをもつ(継承でない) 場合に `true` を返します。

オブジェクトプロパティ(`Object.keys` など)を返すすべてのメソッドは "自身の" プロパティを返します。もし継承されたものが欲しい場合は、`for..in` を使います。
