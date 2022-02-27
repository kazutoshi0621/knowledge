# Rustの日本語ドキュメント
https://doc.rust-jp.rs/

# Cargo
```plaintext
USAGE:
    cargo [+ツールチェイン] [オプション] [コマンド]
```

参考：https://doc.rust-lang.org/cargo/index.html

## コマンド

### ビルド関連
```plaintext
bench       #[bench]が付与されている関数の実行時間を計測
build       ビルド
check       コンパイルできるかをチェック（実行ファイルは生成しない）
clean       ビルド生成物を削除
doc         ドキュメント生成
                記法：
                    //!     ファイルコメント、モジュールコメント
                    ///     関数コメント
                
                ポイント：
                    ・コメント内にマークダウンが使える。
                    ・# Examples 見出し内にサンプルコードを記述しておくと、cargo testでテストを自動実行してくれる

                参考：
                    https://qiita.com/simonritchie/items/87d3743e138763ff3e85
fetch       Cargo.lockに従い、依存ファイルをダウンロード
fix         コンパイラが検出したlint警告を自動的に修正
run         ビルド＆実行
rustc       ビルド（任意のオプションをコンパイラに渡せる）。
rustdoc     ドキュメント生成（任意のオプションをコンパイラに渡せる）。
test        ユニットテストと結合テストを実行
report      非互換性のレポートを表示
```

### マニフェスト関連
```plaintext
generate-lockfile   Cargo.lockを生成もしくは最新に更新。
locate-project      Cargo.tomlをJSONとして出力
metadata            現在のパッケージに関する機械可読メタデータ
pkgid               完全に就職されたパッケージ仕様を印刷する
tree                依存関係グラフのツリー視覚化を表示する
update              ローカル録ファイルに記録されている依存関係を更新する
vendor              すべての依存関係をローカルでベンダー
verify-project      クレートマニフェストの正確性を確認する
```

### パッケージ関連
```plaintext
init        既存のディレクトリに新しいCargoパッケージを作成します
install     Rustバイナリをビルドしてインストールします
new         新しいパッケージを作成します
search      creates.ioでパッケージを検索
uninstall   Rustバイナリを削除する
```

### コマンドの公開
```plaintext
login       レジストリからAPIトークンをローカルに保存
owner       レジストリでクレートの所有者を管理する
package     ローカルパッケージを配布可能なtarballにアセンブルする
publish     パッケージをレジストリにアップロード
yank        押し出されたクレートをインデックスから削除
```


## Cargo.lock
依存解決後の依存グラフのスナップショットを保存しているファイル。
Cargo.tomlには直接の依存関係に関する制約しか書いていないため再現性のあるビルドのためには不十分。
バイナリクレートの場合は、`Cargo.lock`をVCS(git等)に置き、ライブラリクレートの場合は置かないのが一般的。