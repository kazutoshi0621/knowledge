# 11章 トレイトとジェネリクス

トレイトは、インターフェイス・抽象基底クラスのようなもの。

バイト列を書き出すトレイトは、std::io::Write 

```rust
use std::io::Write;

fn hoge(out: &mut dyn Write) -> std::io::Result<()> {
    out.write_all(b"hello world\n");
    out.flush()
}

```

上の意味。
Writeトレイトオブジェクトを受け取って、書き込んで、flush()する。
どこに何を書くとかは定められていない。標準出力かもしれないし、ネットワークかもしれない。
TODO dynってなに？ 動的ディスパッチを示すキーワード。。。うーん。
TODO Result<()>の()ってなに？


トレイトを使えば、strやboolのような組み込み型にもメソッド追加できる。

ジェネリクスで表すと以下のような感じ。
fn hoge<T: Wirte>(out: T) 

## 11.1 トレイトの使い方

トレイトは、何らかの機能。つまりその型ができることを、示す。
なのでC++のインターフェイスというよりかは、抽象基底クラスのほうが近いかも。

トレイトのメソッドを使うには、そのトレイトのuseが必要。
トレイトを使うと任意のメソッドを追加できる。サードパーティの型や標準の型などにも。
そのため、重複することがあるので、明示する仕様になっている。

CloneとIteratorのメソッドは何もインポートしなくても使える。
これはすべてのモジュールに自動的にインポートされる標準のプレリュード（prelude）に含まれているから。

TODO プレリュードって何？

dyn Writeは トレイトオブジェクトと呼ばれる。 
TODO トレイトオブジェクトってなに？


### 11.1.1 トレイトオブジェクト

トレイト型への参照をトレイトオブジェクトと呼ぶ

Write                         ← トレイト型
let hoge: &mut dny Write;     ← トレイト型への参照＝トレイトオブジェクト

トレイト型の変数を生成することはできない
let hoge: dny Write = xxx;    // これはコンパイルエラー
なぜかというと、Rustではコンパイル時に変数サイズが確定していないといけないという制約がある。
以下のように参照ならOK。
let hoge: &mut dny Write = xxx;    // これはコンパイルエラー

#### 11.1.1.1 トレイトオブジェクトのメモリ配置

トレイトオブジェクトはファットポインタである。
（ファットポインタとは、アドレス以外の情報も持っているポインタのこと）。

具体的には、
・データそのものを指すアドレス
・トレイト型の情報（仮想テーブルと呼ぶ。関数ポインタとか）を指すアドレス
の2つの情報を持つ

#### 11.1.2 ジェネリック関数を型パラメータ

```rust
fn hoge(out: &mut dyn Write)            // 普通の関数（トレイト利用）
fn hoge<T: Write>(out: &mut T)          // ジェネリック関数
```

複数のトレイトが実装された型を要求する場合は、`+`を使う
```rust
fn hoge<T: Write + Hash + Debug>(out: &mut T)          // ジェネリック関数
```

whereを使うと、制約を後ろに書くことができる。
```rust
fn run_query<M: Mapper + Serialize, R: Reducer + Serialize>(data: &DataSet, map: M, reduce: R) -> Results
fn run_query<M, R>(data: &DataSet, map: M, reduce: R) -> Results
    where M: Mapper + Serialize,
          R: Reducer + Serialize
```

生存期間を明示するときは、生存期間を先に書く
```rust
fn hoge<'a, T>()
```

定数を受け取ることも可
```rust
fn hoge<const N: usize>(foo: [i32, N], bar: [i32, N])   // fooとbarには、i32型の同じ長さの配列しか渡せない
```
