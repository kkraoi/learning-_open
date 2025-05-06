# Rubyに関するメモ書き

## rbenv
### インストール
1. `brew install rbenv`: homebrewでインストールするため「/opt/homebrew/opt/」にできる。`brew --prefix rbenv`にて検証。
2. 「Users/ユーザー名/.zshrc」に、
      ```
      export PATH="$HOME/.rbenv/bin:$PATH"
      eval "$(rbenv init -)"
      ```
      を書き込む。
3. `exec $SHELL` でターミナルを再読み込み・更新し設定を反映。

### Rubyをインストールする
1. `rbenv install -l` でンストール可能な最新バージョンを確認する。
2. `rbenv install X.X.X` で任意のバージョンをインストールする。
3. `rbenv versions` でインストールしたrubyを確認。
4. `rbenv local X.X.X` | `rbenv global X.X.X` でバージョンを切り替える。前者はカレントディレクトリのスコープで適用され、後者はグローバルスコープで適用される。
5. `exec $SHELL` でターミナルを再読み込み・更新し設定を反映。

## 正規表現
### iで大文字小文字区別せずマッチングを行う
```
/Ruby/i =~ "ruby"
```

### /^\s+/ = 行頭の空白
 ^ : 行頭（文字列の先頭）

 \s : 白文字（スペース・タブ・改行 など）

 + : 「1回以上の繰り返し」（空白が1つ以上連続する部分を対象）

### /^ABC$/ = ABCとマッチ、ABCDはマッチしない
　$ : 行末が前の文字と一致するか

### 文字の先頭・文字の末尾
 ※^や$は行頭・行末であって文字の先頭や文字の末尾ではない！

 \A : 文字の先頭

 \z : 文字の末尾

### マッチしたい文字を範囲で指定する
[0AB] : 0またはAまたはB

[A-Z] : アルファベと大文字

[a-z] : アルファベ小文字

[0-9] : 0から9までの数字

[A-Za-z_] : [A-Z]と[a-z]と「_」

[A-Za-z0-9_-] : アルファベットと数字と「_」と「-」

[^ABC] : ABC以外

### 任意の文字とマッチ
. : 任意の1文字とマッチする

/A..D/ : 「ABCD」とマッチ。「ABD」はマッチしない。

/^...$/ : 三文字の文字とマッチ

### バックスラッシュを使ったパターン
\s : 空白、たぶ、改行文字、改ページ文字

\d : 0~9の数字

\w : 英数字

\A : 文字列の先頭

\z : 文字列の末尾

\メタ文字 : `\[`の場合、`[`とマッチ。`\^`の場合、`^`とマッチ。

### 繰り返し
 * : ０回以上の繰り返し

 + : 1回以上の繰り返し

 ? : 0または１回の繰り返し。つまり直前の文字があってもなくても良い。

 {n} : n回の繰り返し

 {n, m} : n~m回の繰り返し

 ### 選択 「|」
 `/^(ABC|DEF)$/`: ABC => マッチする。DEF => マッチする。AB => マッチしない。ABCDEF => マッチしない。

 ### キャプチャ
 ```
 /@/ =~ "ローカルパート@ドメイン名"
 local_part = $` 対象より前方
 dimain = $' 対象より後方

 または
 /\A(.*)@(.*)\z/ =~ "ローカルパート@ドメイン名"
 local_part = $1
 dimain = $2

 =>
 p local_part #=> "ローカルパート"
 p dimain #=> ドメイン名"
 ```

### URlからサーバーのアドレスをキャプチャできる
`%r|https?://([^\/]*)/|`

`%r||` : メタ文字「\」をエスケープ

`s?` : sがあってもなくても良い。

`()` : キャプチャする範囲

`[^/]` : 「/」を含まない

`*` : 前の文字がなくてもよく、また複数の文字。

⇩フル(スキーム名、サーバーアドレス、パス、クエリ名、フラグメント)でキャプチャできる正規表現
`%r|^(([^:?#]+):)?(//([^/?#]*))?([^?#]*)(\?([^#]*))?(#(.*))?|`

## p pp
print puts のように出力するが、ppは、オブジェクトの構造を見やすくしてくれる。コンテナを確認する時に便利。

## 多重代入
```
array  = [1,2]
a, b = array
=> a, b = 1, 2
```

```
array  = [1,2]
a,  = array
#↑ 明示的にする場合、a, _ = array と書くと良い
=> a = 1
```

## 条件判断

### unless文
```
unless 条件 then
      処理
end
```
条件が偽であれば処理を実行。

### case文
JSのswhitch文みたいなもの。<br>
比較したいオブジェクトが一つだけで、そのオブジェクトの値によって場合分けしたい場合に使う。


例:
```
tags = ["A", "IMG", "PRE"]
tags.each do |tagname|
  case tagname
  when "P", "A", "I", "B", "BLOCKQUOTE"
    puts "#{tagname} has a child"
  when "IMG", "BR"
    puts "#{tagname} has no child"
  else
    puts "#{tagname} cannot be used"
  end
end
```

例: オブジェクトの型で処理を場合分け
```
array = ["A", 1, nil]
array.each do |item|
  case item
  when String
    puts "#{item} is String"
  when Numeric
    puts "#{item} is Numeric"
  else
    puts "#{item} is something"
  end
end
```

例: 電子メールのヘッダ解析
```
text.each_line do |line|
  case String
  when /^From:/i
    puts "送信者 の情報を見つけました"
  when /^To:/i
    puts "宛先 の情報を見つけました"
  when /^Subject:/i
    puts "件名 の情報を見つけました"
  when /^$:/
      #^$は空行を表す。ヘッドと本文の間に必ず空行がある。
    puts "ヘッダの解析が終了しました"
  else
   # 読み飛ばす
  end
end
```

### if修飾子　unless修飾子
```
puts "aはbよりも大きい" if a > b
```
ifブロックと同じことがコンパクトに書ける


## メソッド

### メソッド命名ルール
trueかfalseを返す場合は、メソッド名の末尾に?をつける

### 不定数の引数を受け取る場合
```
def meth(first_arg, *args, last_arg)
      [first_arg, args, last_arg]
end

p meth(1,2,3,4,5) #=> [1, [2,3,4], 5]
p meth(1,2) #=> [1, [], 2]
```

### キーワード引数
↑「不定数の引数を受け取る場合」のように、引数を個数と順番に従って与える必要はない書き方。
```
def hyo_menseki(x: 0, y: 0, z: 0, **args)
  unless args.empty?
    puts "定義に存在しないキーワード引数#{args}は無効です"
  end

  xy = x * y
  yz = y * z
  xz = x * z

  return (xy + yz + xz) * 2
end

p hyo_menseki( x: 2, y: 4) # 個数 問題ない
p hyo_menseki(z: 3, x: 2, y: 4) # 順番 問題ない
p hyo_menseki(z: 3, x: 2, y: 4, v: 5, w: 6) # 定義に存在しないキーワード引数

config = {z: 3, x: 2, y: 4, v: 5, w: 6}
p hyo_menseki(**config) # 3系では**（キーワード引数展開）をつけないとだめ
```
「キーワード引数展開」はスプレッド構文みたいなもので、キーワード引数をハッシュに展開する。

### 配列を展開して引数に渡す
```
def method_name(a, b, c)
  puts a + b + c
end

args = [1, 2, 3]
method_name(*args)
```
「*args」で展開する。

## 演算子

### 配列が空じゃなければ、値を取り出す
```
item = ary && aray.first
⇩ 改良
item = ary.&first
```

### 値がfalsyであれば、値を代入(デフォルト値設定)
```
var = var || 1
⇩ 改良
var ||= 1
```

### 範囲演算子「..」「...」
```
p (5..10).to_a
#=> p (5..10).to_a

p (5...10).to_a
[5, 6, 7, 8, 9]
```

## rescue修飾子
```
number = Integer(var) rescue 0
```
Integerメソッドは、引数が`"123"`のような数値らしき文字列である場合は数値にして返し、`"abc"`のような、どうみても文字列の場合は例外処理を発生させる。

## 定番テク
### ファイルを開くとき
```
File.open("sample.txt") do |file|
  file.each_line do |line|
    # 行の処理
  end
end
```
これは
```
file = File.open("sample.txt")
begin
  file.each_line do |line|
    # 行の処理
  end
ensure
  file.close
end
```
と同等だが、スマートだね。ファイルの閉じ忘れを防ぐことができる。

### テキストをハッシュに
```
def str_to_hash(str)
  hash = Hash.new()
  array = str.split(/\s+/)
  while key = array.shift
    value = array.shift
    hash[key] = value
  end
  return hash
end
p str_to_hash("キー バリュー キー バリュー\nキー バリュー")
```

## グローバル変数
### $!
「最後に発生した例外オブジェクト」**を保持している。

## イデオム
### 繰り返し新しい行を読むとき
```
f.each_line do |line|
  line.chomp!
  lineを処理する
end
```
`chomp!`で破壊的に行末の`\n`を削るのが定番

### getsメソッドを使う際の定番
io.gets は1行を返す。
chomp! は改行コードを消す
```
while line = io.gets
  line.chomp!
  # lineに対する処理
end
io.eof? #入力の終わりまで読み込んだか？
```

## バージョンを変える
```
bundle install
↓
Your Ruby version is x.x.x, but your Gemfile specified x.x.x
```
のようなエラーが出ている場合、
```
ruby 'x.x.x'
```
でバージョンを変え、
```
bundle install
```
しなおす。

## TIPS
### マジックコメント
rbの1行目に`# encoding: Shift_JIS`のように書くことをマジックコメントといい、文字コードを指定することで、
```
method-print_puts_p.rb:3: invalid multibyte char (Shift_JIS)
```
のようなエラーを解消できる。<br>
※Shift_JISはwindows<br>

また、
```
ruby -E UTF-8 スクリプトファイル名
```
と実行すると、文字化けを解消できる可能性がある。<br>
-E = --encoding

### コマンド：　ネットからダウンロード
```
curl -o ファイル名 ファイルパス
```
このコマンドによって、カレントディレクトリにダウンロードできる。<br>
-o: ダウンロードした内容を指定したファイル名で保存するために使います。

例
```
curl -o NEWS-3.4.0.md https://raw.githubusercontent.com/ruby/ruby/refs/heads/master/doc/NEWS/NEWS-3.4.0.md
```

### vscode スニペット登録
`Cmd + Shift + P` を押して `Preferences: Configure Snippets` を検索。

jsonに、
```
  "スニペットタイトル": {
    "prefix": "スニペットキー",
    "body": [
      "1行コード",
      "  1行コード",
      "1行コード"
    ],
    "description": "　を展開。"
  },
```
を書く。

### コマンド: ファイルをコピペ
```
cp コピー元　コピーファイル名
```

### ファイル移動
```
mv ファイル名　移動先ディレクトリ
```

### 圧縮されたファイルを展開して内容を表示する
```
zcat file.txt.gzなど圧縮ファイル
```

### ファイルの最後の部分を表示させる
```
tail filename.txt #=> ファイルの最後の10行を表示
tail -n 20 filename.txt => -n でファイルの最後の[-nに続く数字]行を表示
```

### ファイルに書き込む
```
echo "書き込む内容" > 書き込むファイル
```

### gemがどこにインストールされているのか
```
gem env gemdir
```

### インストール先を調べる
```
which nodebrewやgemなど
```

### ruby: インストール可能のバージョンを調べる
```
rbenv install -l
```

### ruby: バージョンをインストール
```
rbenv install 3.3.0
```

### ruby: 持っているバージョンを調べる
```
rbenv versions
```

### ruby: バージョンを切り替える
```
rbenv global 3.3.0
rbenv loacal 3.3.0
ruby -v
```

```
gem install sqlite3 -v 1.4.2  --install-dir vendor/gems

$LOAD_PATH.unshift File.expand_path("vendor/gems/lib")

require "sqlite3"
```


## 用語

### シンタックスシュガー
直感的にわかりやすい構文にすること。<br>
`3.add(2)`を`3 + 2`とするとわかりやすいよね。これがシンタックスシュガー

### レシーバ
メソッドの前のオブジェクトのこと。<br>
メソッドを実行することは、オブジェクトにメッセージを送り、そのメッセージの返答を受け取るから。