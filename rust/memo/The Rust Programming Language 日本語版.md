https://doc.rust-jp.rs/book-ja/title-page.html



## 宣言

```rust
// 変数宣言
let hoge;           // 不変変数
let mut hoge;       // 可変変数

// 定数宣言
const HOGE: u32 = 100_000;    // 名前はすべて大文字にすること。型注釈は必須。数値は _ で区切ることができる。
```

## 型

```rust
//整数型
let hoge: i8;
let hoge: u8;
let hoge: i16;
let hoge: u16;
let hoge: i32;
let hoge: u32;
let hoge: i64;
let hoge: u64;
let hoge: isize;
let hoge: usize;

// 浮動小数点型
let hoge: f32;
let hoge: f64;

// 論理型
let hoge: bool;

// 文字、文字列
let hoge: char;
let hoge: &str;

// タプル
let hoge: (bool, i32);

// 配列
let hoge: [u32; 5];
```

## 関数

```rust
fn hoge(foo: i32) -> bool {
    true                // 関数の最後にセミコロン無しで戻り値を書く。
                        // return true; と書いても同じ意味だが、rustでは関数末尾はこの書き方にするのが普通。
}
```

## 制御構文
if, loop, while , for

# 4 所有権

## 4.1 所有権とは

所有権は、ヒープ上のデータを安全に管理するための仕組み。

| メモリ   | 特徴                                                                                                             |
| -------- | ---------------------------------------------------------------------------------------------------------------- |
| スタック | ・高速（キャッシュヒットしやすい。静的なメモリ確保と解放）<br>・サイズがコンパイル時に決定している必要がある     |
| ヒープ   | ・低速（キャッシュミスヒットしやすい。動的なメモリ確保と解放）<br>・サイズがコンパイル時に決定していなくても良い |

所有権のルール

  - 変数は値の所有権を持つ
  - 変数の値を別の変数に代入すると、所有権は移動する
  - 所有権を失った変数は破棄される

#### 所有権におけるスタックとヒープの動作の違い

代入時に、値のコピーが走るのか、所有権の移動が走るのかが違う。

```rust
// スタックのみを使う型の場合（値のコピー）
let x = 5;              // 変数x は 値5 の所有者になる
let y = x;              // 値5 がコピーされ、変数y は 新たな値5 の所有者になる
                        // コピーされるのは、プリミティブ型にはCopyトレイトが実装されているから


// ヒープを使う型の場合（所有権の移動）
let s1 = String::from("hello");
    /*
        ヒープ上にメモリを確保し"hello"で初期化。そこを指すファットポインタをs1に格納。
        ファットポインタとはアドレス以外の情報も持つポインタのこと。

        以下は実行後のメモリ構造。
        （capacityはヒープ上に確保したメモリサイズのこと）
        -------------------------------+------------------------------
                    stack              |            heap            
        -------------------------------+------------------------------
                                       |                            
                    s1                 |
            +==========+=========+     |                            
            | name     |  value  |     |   
            +==========+=========+     |          +---------+
            | ptr      |    .-------------------> |    h    |
            +----------+---------+     |          +---------+
            | len      |    5    |     |          |    e    |
            +----------+---------+     |          +---------+
            | capacity |    5    |     |          |    l    |
            +----------+---------+     |          +---------+
                                       |          |    l    |
                                       |          +---------+
                                       |          |    o    |
                                       |          +---------+
                                       |                            
        -------------------------------+------------------------------
    */
let s2 = s1;
    /*
        s1が所有するヒープ上の値を、s2に所有権移動。
        （所有権が移動するのは String型が `Copy`トレイトを実装していないから）
        s1は所有権を失うため、stackは解放される。

        以下は実行後のメモリ構造。
        -------------------------------+------------------------------
                    stack              |            heap            
        -------------------------------+------------------------------
                                       |                            
                      s1               |
            +==========+=========+     |                            
            | name     |  value  |     |   
            +==========+=========+     |          +---------+
            |                    |     |    +---> |    h    |
            |                    |     |    |     +---------+
            |       invalid      |     |    |     |    e    |
            |                    |     |    |     +---------+
            |                    |     |    |     |    l    |
            +----------+---------+     |    |     +---------+
                                       |    |     |    l    |
                                       |    |     +---------+
                      s2               |    |     |    o    |
            +==========+=========+     |    |     +---------+
            | name     |  value  |     |    |
            +==========+=========+     |    |
            | ptr      |    .---------------+
            +----------+---------+     |
            | len      |    5    |     |
            +----------+---------+     |
            | capacity |    5    |     |
            +----------+---------+     |
                                       |                            
        -------------------------------+------------------------------
    */
println!("{}, world!", s1);     // s1が未初期化なのでエラーになる
```

値のコピーが走るかどうかは、その型がCopyトレイトを実装しているかどうかによる。Copyトレイト実装している型の例。
- 整数型（u32など）。
- bool型。
- 浮動小数点型（f64など）。
- char。
- Copyトレイトを実装した型だけを含むタプル。(char, bool) など。

#### clone (deep copy)
明示的にclone()を呼べば、ヒープ上のデータもコピーできる。ただし実行コストは高い。

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);   // エラーにならない
```

### 関数
関数に変数を渡した場合も、代入(`=`)と同様のことが起きる。

```rust
fn hoge(bar: String) {
    println!("{}", bar);
} // barがスコープを抜ける。所有者がスコープを抜けたため、drop()が呼ばれヒープが解放される。

fn main() {
    let foo = String::from("hello");
    hoge(foo);           // fooが持っている  "hello"の所有権  が関数側に移動する。
                         // ← ここではもうfooは有効ではない

} // fooがスコープを抜ける。しかしすでにムーブされているのでdrop()は呼ばれない。
```

戻り値についても同様。

```rust
fn main() {
    let bar = hoge();         // gives_ownershipは、戻り値をs1に

} // barがスコープを抜ける。所有者がスコープを抜けたため、drop()が呼ばれヒープが解放される。

fn hoge() -> String {
    let foo = String::from("hello");
    foo                       // 所有権が移動
}
```

## 参照と借用

`&` で参照を渡すことができる。意味的にはCのポインタと同じで先頭アドレスが格納される。

```rust
let s1 = String::from("hello");
let s2 = &s1;       // s2 は &String 型
                    // 厳密に言うと、s2 は、「スタックに積まれたString型のファットポインタ」の先頭アドレス、になる。
```

参照を代入する場合は、値の所有権は移動しない（これを、借用という）。
所有していないので、変数がスコープアウトしてもdropされない。
ただし、所有権を持つ変数が「先に」スコープアウトするとコンパイルエラーになる（だって参照先が無くなるから）。

### 可変参照と不変参照

参照している値を変更したい場合は、mut をつける
```rust
let mut s = String::from("hello");     // まず、そもそもの値を可変にしないといけない
let s2 = &mut s;                       // &mut で可変参照を表す。s2 は &mut String 型。
```

メモリ安全性を守るため、同時に一人しか可変参照をすることはできない。
（これを許すと、2つ以上のポインタが同一データに同時アクセスできてしまうことに）。

```rust
let mut s = String::from("hello");
let r1 = &mut s;
let r2 = &mut s;    // これはコンパイルエラー
```

メモリ安全性を守るため、誰かが不変参照をしている間は、可変参照をすることはできない。
（これを許すと、変更中の値を参照できてしまうことに）。

```rust
let mut s = String::from("hello");

let r1 = &s;        // 
let r2 = &mut s;    // これはコンパイルエラー
```

当然ながら、不変参照はいくつでもOK。

```rust
let s = String::from("hello");

let r1 = &s;        
let r2 = &s;        // これはOK
```

## 4.3 スライス型

### 文字列スライス

文字列データへの参照。`&str`型になる。

```rust
    let hoge = String::from("foobar");
    let fuga = &hoge[1..4];     // "oob"

    // 省略可能
    let fuga = &hoge[..4];      // "foob"       
    let fuga = &hoge[1..];      // "oobar"
```

### 配列のスライス

`&[xxx]`型になる。

```rust
    let hoge = [1, 2, 3, 4, 5];
    let fuga = &hoge[1..4];     // [2, 3, 4]      &[i32]型
```

# 5 構造体
```rust
// 宣言
struct Hoge {
    foo: i32,
    bar: bool,
}

// インスタンス生成
let hoge = Hoge {
    foo: 0,
    bar: true,
};
let fuga = Hoge {
    foo: 0,
    ..hoge            // 構造体更新記法という。明示的にセットされていない残りのフィールドを初期化する。
};
```

タプル構造体

```rust
// 宣言
struct Hoge(i32, bool);

// インスタンス生成
let hoge = Hoge(0, true);

// アクセス
let foo = hoge.0;
let Hoge(bar, baz) = hoge;      // barとbazにそれぞれの値が入る
```

ユニット構造体
`()`のこと
フィールドを持たない構造体。トレイトだけを実装するときに使う。

# 10 ジェネリック型、トレイト、ライフタイム

ジェネリック型にトレイトを組み合わせることで、特定の振る舞いのある型のみに制限できる。
ライフタイムとは、参照の関係性をコンパイラに与えるジェネリクスの一種。

## 10.1 ジェネリックなデータ型
関数、構造体、enum、メソッドでジェネリクスを使用することができる。

### 関数
```rust
fn hoge<T>(foo: T) {
```

### 構造体
```rust
struct Hoge<T> {
    x: T,
    y: T,
}
```

### enum
```rust
enum Option<T> {
    Some(T),            // T型の値を保持する、Someという名前の列挙子
    None,               // なんの値も持たない、Noneという名前の列挙子（実は数値を持っているが・・・）
}

enum Result<T, E> {
    Ok(T),              // T型の値を保持する、Okという名前の列挙子
    Err(E),             // E型の値を保持する、Errという名前の列挙子
}
```

### メソッド
```rust
struct Hoge<T> {
    x: T,
    y: T,
}

// impl<T> という構文になる
impl<T> Hoge<T> {
    // xが保持している値の参照を返すメソッド
    fn x(&self) -> &T {
        &self.x
    }
}

// Hoge<f32>だけにメソッドを実装することもできる。
// ※このときは、impl<T>である必要はない（implブロックの中にTが登場しないため）。
impl Hoge<f32> {
    fn fuga(&self) -> f32 {
        self.x + self.y
    }
}

// 別の型引数Uを取ることもできる
impl<T> Hoge<T> {                   // ここでimpl<T, U> のように U を宣言する必要はない。
    fn fuga<U>(&self, foo T) {      // ここで U を宣言すれば良い（関数の中でしか使わないため）。
        :
    }
}
```


## 10.2 トレイト: 共通の振る舞いを定義する
ざっくりいうと、インターフェイスや抽象基底クラスのようなもの（厳密には違う）。
メソッドのみ定義でき、変数は定義できない。

### トレイトを定義する

```rust
pub trait Summary {         // pubは、このトレイトを公開する、という意味
    // このトレイトを実装する型は、summarize関数を定義する必要がある。
    fn summarize(&self) -> String;
}
```

### トレイトを型に実装する

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

// Summaryトレイト を NewsArticle型 に実装する
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

// Summaryトレイト を Tweet型 に実装する
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```


トレイトを実装後、普通のメソッド同様にNewsArticleやTweetのインスタンスに対してこのメソッドを呼び出せる。
```rust
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            // もちろん、ご存知かもしれませんがね、みなさん
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
```

### トレイト実装の注意点

トレイト か 対象の型 のいずれかが自分のクレートで定義されている必要がある。
言い換えると、外部のトレイトを外部の型に対して実装することはできないということ。

この制限により、他の人のコードが自分のコードを壊したり、その逆が起きないことが保証される。もしこの制限がなければ、2つのクレートが同じ型に対して同じトレイトを実装できてしまい、コンパイラはどちらの実装を使うべきかわからなくなってしまう。

### デフォルト実装 

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        // "（もっと読む）"
        String::from("(Read more...)")
    }
}

// 型への実装はこれだけでよい。もしくは明示的に関数定義してオーバーライドすることも可。
impl Summary for NewsArticle {}

```

### ジェネリクスとトレイトの組み合わせ

トレイト境界を用いて、ジェネリクスに制約を設けることができる。

```rust
pub fn notify<T: Summary>(item: &T) {       // T: のあとの Summary 部分をトレイト境界と呼ぶ
    println!("Breaking news! {}", item.summarize());
}

// impl Trait 構文（上記のシンタックスシュガー）
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

#### 複数のトレイト境界（AND条件）

```rust
pub fn notify<T: Summary + Display>(item: &T) {     // T は Summary と Display 両方のトレイトを実装している必要がある

// impl Trait 構文（上記のシンタックスシュガー）
pub fn notify(item: &(impl Summary + Display)) {

// これも同じ意味。where句を使うことで、関数名と引数リストが近づいて見やすい。
pub fn notify<T>(item: &T) {
    where T: Summary + Display
{
```

### トレイトを実装している型を返す
以下のように、impl Trait構文を戻り値型のところで使う。

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

ただし、impl Traitは一種類の型を返す場合にのみ使える。
たとえば、以下のように NewsArticle か Tweet のいずれかを返すようなコードはコンパイルエラー。
（このような振る舞いをする関数を書く方法は、17章のトレイトオブジェクトで異なる型の値を許容する節で学びます）。

```rust
This code does not compile!
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
```

### トレイト境界ごとに、メソッドを実装しわける

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

// 常にこのメソッドは実装される
impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// Tが Display + PartialOld 境界を満たしているときのみ、このメソッドは実装される
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

### ブランケット実装

あるトレイト境界を満たしたすべての型に特定のトレイトを実装することを、ブランケット実装(blanket implementation)と呼ぶ。Rustの標準ライブラリで広く使用されている。

例えば、
```rust
// Displayトレイトが実装されたすべての型に、ToStringトレイトを自動的に実装。
// （つまりDisplayトレイトを実装すれば、ToStringトレイトで定義されたto_string()などが呼び出せる）
impl<T: Display> ToString for T {
    （省略）
}
```

ブランケット実装は、トレイトのドキュメンテーションの「実装したもの」節に記載がある。


















































## 10.3 ライフタイムで参照を検証する

Rustの参照は全てライフタイム（その参照が有効となるスコープ）を持つ。

ライフタイムは基本的にコンパイラが自動で推論するが、参照のライフタイムがいくつか異なる方法で関係することがある場合は、ジェネリックライフタイム引数を使用して、ライフタイムを明示的にアノテーションすることが必要。

この章では、ライフタイム記法が必要となる最も一般的な場合について説明する。詳細は、第19章の「高度なライフタイム」節を参照のこと。

### ライフタイムによる、ダングリング参照の回避
ライフタイムの主目的は、ダングリング参照（宙ぶらりんな参照）を回避すること。

```rust
{
    let r;
    {
        let x = 5;
        r = &x;
    }
    println!("r: {}", r);       // ダングリング参照（参照先の値が消えている）。コンパイルエラー。
}
```

### ボローチェッカー
ボローチェッカーは以下であることをチェックするプログラムである。
「参照される変数（上の例で言うと 'b ）のライフタイム」＞＝「参照する変数（上の例で言うと 'a ）のライフタイム」

ボローチェッカーにより、以下のコードが不正であることが検出できる。
```rust
{
    let r;                // -----------+
                          //            | <-'a (r のライフタイム)
    {                     //            |
        let x = 5;        // -+         |
        r = &x;           //  | <- 'b (x のライフタイム)
    }                     // -+         |
                          //            |
    println!("r: {}", r); //            |
}                         // -----------+
```

### 関数のジェネリックなライフタイム
以下のように、2つの文字列スライスを渡し長い方を返す関数longest()を定義する。

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("{}", result);
}
```

ただし上記のコードはコンパイルエラーになる。

```plaintext
error[E0106]: missing lifetime specifier
(エラー: ライフタイム指定子が不足しています)
 --> src/main.rs:1:33
  |
1 | fn longest(x: &str, y: &str) -> &str {
  |                                 ^ expected lifetime parameter
  |                                   (ライフタイムパラメータが必要です)
  |
  = help: this function's return type contains a borrowed value, but the
signature does not say whether it is borrowed from `x` or `y`
  (ヘルプ: この関数の戻り値の型は借用値を含んでいるが、シグネチャには `x` と `y` のどちらから借用したかは書かれていない。)
```

x または y のどちらかが戻り値であるが、x も y もライフタイムが分からないので、ボローチェッカーが戻り値のライフタイムを決定できない。そのためエラーになる。ライフタイムパラメータを追加し、ボローチェッカーが分析を実行できるようにする。



### ライフタイムアノテーション
ライフタイムアノテーションを使いライフタイムの関係を記述する。アノテーションにより参照の寿命が変わることはない。
アノテーションは、アポストロフィ（'）＋名前、で指定。ほとんどの人は、'a や 'b という名前を使う。

```rust
&i32        // 参照
&'a i32     // 明示的なアノテーションをした 参照
&'a mut i32 // 明示的なアノテーションをした 可変参照
```

### 関数におけるライフタイムアノテーション
ジェネリック<>の中にライフタイムパラメータを宣言する。

```rust
/*
  すべての引数と戻り値に同じライフタイム ('a) を明示

  この関数を呼んだ場合に、ジェネリクスの 'a（ライフタイム）は具体的にどう推論されるのか？
  x の 'a と y の 'a を同時に満たそうとすると、'a は x と y のライフタイムの重なる部分にならざるを得ない。
  つまり、'a は x と y のライフタイムのうち、longest()呼び出し以降のライフタイムがより短いもの、と等しくなる。

  x と y のより小さいライフタイムが 'a なので、戻り値の参照のライフタイムも 'a になる。
*/
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

関数やメソッドの引数のライフタイムは、入力ライフタイムと呼ばれ、 戻り値のライフタイムは出力ライフタイムと呼ばれる。

アノテーションにより渡された参照のライフタイムが変わるのではない。
この制約に従わない場合にボローチェッカがエラーにするということ。
たとえば、longest()の例だと、x と y のライフタイムが重ならない場合は 'a が存在しないためエラーになる。

アノテーションがあることで、Rustコンパイラが行う解析がよりシンプルになり、コンパイラのエラーは、コードのその部分と制約をより正確に指摘することができる。


ボローチェッカが弾く例

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());   // result のライフタイムは、string2と同じ
    }
    println!("The longest string is {}", result);   // エラー
}
```

### ライフタイムの観点で思考する
何にライフタイム引数を指定する必要があるかは、関数の中身に依存する。

```rust
// 例えば、以下は y に対してライフタイムを指定する必要はない（コンパイルが通る）。
// x や 戻り値のライフタイムとは何の関係もないから
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

関数が参照を返す場合、そのライフタイムは、いずれかの引数と同じライフタイムにする必要がある。
なぜかというと、戻り値は必ずいずれかの引数と関係しているため。
もしどの引数とも無関係の値の参照を返すとしたら、それは関数内で生成した値になり、ダングリング参照になってしまうため。


### 構造体定義のライフタイム注釈
参照を含む構造体の場合、全参照にライフタイムアノテーションが必要

```rust
// このアノテーションは、ImportantExcerptのインスタンスよりも
// partフィールドが保持している参照の方が長生きすることを意味する
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')       // first_sentence は novel の文字列への参照
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };  // part も novel の文字列への参照
}
```
novelの文字列データは、ImportantExcerptがスコープを抜けるまで存在するのでコンパイルが通る。


### ライフタイム省略
引数と戻り値に参照が1つだけある場合は、省略可。

```rust
fn hoge(s: &str) -> &str {
    return s
}
```

### メソッド定義におけるライフタイム注釈
構造体にライフタイムのあるメソッドを実装する際、リスト10-11で示したジェネリックな型引数と同じ記法を使用します。 ライフタイム引数を宣言し使用する場所は、構造体フィールドかメソッド引数と戻り値に関係するかによります。

構造体のフィールド用のライフタイム名は、implキーワードの後に宣言する必要があり、 それから構造体名の後に使用されます。そのようなライフタイムは構造体の型の一部になるからです。

implブロック内のメソッドシグニチャでは、参照は構造体のフィールドの参照のライフタイムに紐づくか、 独立している可能性があります。加えて、ライフタイム省略規則により、メソッドシグニチャでライフタイム注釈が必要なくなることがよくあります。 リスト10-25で定義したImportantExcerptという構造体を使用して、何か例を見ましょう。

まず、唯一の引数がselfへの参照で戻り値がi32という何かへの参照ではないlevelというメソッドを使用します:



impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
impl後のライフタイム引数宣言と型名の後に使用するのは必須ですが、最初の省略規則のため、 selfへの参照のライフタイムを注釈する必要はありません。

3番目のライフタイム省略規則が適用される例はこちらです:



impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        // お知らせします
        println!("Attention please: {}", announcement);
        self.part
    }
}
2つ入力ライフタイムがあるので、コンパイラは最初のライフタイム省略規則を適用し、 &selfとannouncementに独自のライフタイムを与えます。それから、 引数の1つが&selfなので、戻り値型は&selfのライフタイムを得て、 全てのライフタイムが説明されました。

静的ライフタイム
議論する必要のある1種の特殊なライフタイムが、'staticであり、これはプログラム全体の期間を示します。 文字列リテラルは全て'staticライフタイムになり、次のように注釈できます:



// 静的ライフタイムを持ってるよ
let s: &'static str = "I have a static lifetime.";
この文字列のテキストは、プログラムのバイナリに直接格納され、常に利用可能です。故に、全文字列リテラルのライフタイムは、 'staticなのです。

エラーメッセージで'staticライフタイムを使用する提言を目撃する可能性があります。 ですが、参照に対してライフタイムとして'staticを指定する前に、今ある参照が本当にプログラムの全期間生きるかどうか考えてください。 可能であっても、参照がそれだけの期間生きてほしいかどうか考慮する可能性があります。 ほとんどの場合、問題は、ダングリング参照を生成しようとしているか、利用可能なライフタイムの不一致が原因です。 そのような場合、解決策はその問題を修正することであり、'staticライフタイムを指定することではありません。

ジェネリックな型引数、トレイト境界、ライフタイムを一度に
ジェネリックな型引数、トレイト境界、ライフタイムを指定する記法を全て1関数でちょっと眺めましょう！



use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    // アナウンス！
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
これがリスト10-22からの2つの文字列のうち長い方を返すlongest関数ですが、 ジェネリックな型Tのannという追加の引数があり、これはwhere節で指定されているように、 Displayトレイトを実装するあらゆる型で埋めることができます。 この追加の引数は、関数が文字列スライスの長さを比較する前に出力されるので、 Displayトレイト境界が必要なのです。ライフタイムは一種のジェネリックなので、 ライフタイム引数'aとジェネリックな型引数Tが関数名の後、山カッコ内の同じリストに収まっています。

まとめ
いろんなことをこの章では講義しましたね！今やジェネリックな型引数、トレイトとトレイト境界、そしてジェネリックなライフタイム引数を知ったので、 多くの異なる場面で動くコードを繰り返しなく書く準備ができました。ジェネリックな型引数により、 コードを異なる型に適用させてくれます。トレイトとトレイト境界は、型がジェネリックであっても、 コードが必要とする振る舞いを持つことを保証します。ライフタイム注釈を活用して、 この柔軟なコードにダングリング参照が存在しないことを保証する方法を学びました。 さらにこの解析は全てコンパイル時に起こり、実行時のパフォーマンスには影響しません！

信じるかどうかは自由ですが、この章で議論した話題にはもっともっと学ぶべきことがあります: 第17章ではトレイトオブジェクトを議論します。これはトレイトを使用する別の手段です。 第19章では、ライフタイム注釈が関わるもっと複雑な筋書きと何か高度な型システムの機能を講義します。 ですが次は、コードがあるべき通りに動いていることを確かめられるように、Rustでテストを書く方法を学びます。









