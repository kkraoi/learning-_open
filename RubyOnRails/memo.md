# Ruby On Rails について
## Railsの雛形を作成する
```
rails new アプリケーション名
```

## アプリケーションを削除
```
rm -rf アプリケーション名
```

## 構成
### コア部分
```
|- app/
    |- assets/          => 以下3つのファイルが保存されています。
       |- images/       => 画像ファイルを保存
       |- stylesheets/  => CSSやSCSSファイルを保存
       |- config/       => 画像、CSSやJavaScriptを圧縮するための設定ファイルを保存しています。
    |- controllers/     => MVCにおけるControllerのファイルを保存しています。
    |- javascripts/     => JavaScriptファイルを保存
    |- models/          => MVCにおけるModelのファイルを保存しています。
    |- views/           => MVCにおけるViewのファイル（〜.html.erbと名のつくファイル）を保存しています。
```

### コマンドの設定ファイル
`~/bin/`

### アプリケーションに関する設定
```
|- config/
    |- initializers/  => 初期化ファイルを管理
    |- database.yml   => データベースへの接続設定を記述するファイル
    |- routes.rb      => ルーティングを設定するファイル
```

### データベース
```
|- db/
    |- migrate/  => マイグレーションファイル（データベース上のテーブルを作成・更新するために必要なファイル）を管理しています。
    |- schema.rb => 最新のデータベースの状態をキャプチャするファイルです。
```

### 公開ファイル群
`~/public/`<br>
サーバーにデプロイ（アプリケーションの公開）をした後、ウェブ上に公開されるファイルを保存しています。

### テストファイル群
`~/test/`

## Railsでサーバーを立ち上げる
```
rails s -p 8080
```
「s」は「server」の略

## 必要なプラグインを追加する
```
yarn add @babel/plugin-proposal-private-methods @babel/plugin-proposal-private-property-in-object
```
## MVCモデルとルーティング
### ルーティングの役割
URLを提供する

### コントローラの役割
ルーティングで判断された結果を元に、適切なページを表示させる。

### ビューの役割
ブラウザに表示させるHTMLを実際に組み込む。

### モデルの役割
データベースにアクセスをして、データの登録・取得・削除・更新を行う。

## コントローラを作成する
```
rails g controller コントローラ名
```
gはrails generateの略

## モデルの作成
```
rails g model モデル名
```
モデルの名称は、「List」のように先頭大文字。

## 疑問
### controllerとは？
モデルに対してデータをとってくる命令を出す、ビューに対して、とってきたデータを渡すような司令塔。

### erbファイルとは？
HTMLファイル内でRubyの構文を使えるようにしたもの。

### アクションとは？
コントローラの中の処理内容。ユーザーがこれを使う感じ<br>
コントローラ内のpublicメソッドで、ルーティングを通じてユーザーが直接呼び出せる処理

## Railsアプリケーションを作成する際の手順のまとめ

1. Railsアプリケーションのひな形を作成する
2. 作成したフォルダへ移動する
3. アプリケーションサーバを起動する
4. URLにアクセスする
5. ホスト許可の設定をする
6. アプリケーションサーバを停止する

## RuntimeErrorが返ったら
```
Rails::Application is abstract, you cannot instantiate it directly. (RuntimeError)
```
これが返ってきたら、
1. Gemfile.lockにて、 springが4.3系になっていることを確認する。なっていたら、`4.2.1`に変更する。
2. `spring stop` をする。springが起動していなかったらOK。
3. `bundle install`を実行し、パッケージを更新する。

## Rspec(DWCにて)
### Gemfile 編集
```
group :test do
  gem 'capybara', '>= 2.15'
    gem 'rspec-rails'
    gem "factory_bot_rails"
    gem 'faker'
end
```
に変更し、最後の行に
```
gem 'net-smtp'
```
を追加、その後`bundle install`。

### config/environments/test.rb 編集
```
config.active_support.deprecation = :stderr
↓
config.active_support.deprecation = :silence
```
に変更、その後`rails db:migrate RAILS_ENV=test`でマイグレート

### 実行コマンド
```
bundle exec rspec spec/ --format documentation
```
でテストを実行。

- bundle exec<br>
→ Gemfileに指定されているバージョンのRSpecを使うため。（システム全体のバージョンとズレないように）
- rspec spec/<br>
→ spec/ディレクトリ以下の全テストファイルを実行する、という指示。
- --format documentation<br>
→ テスト実行結果を見やすく整形して表示する。（例えば「〇〇ができること」というふうにツリー構造で出る）

### ちなみに
```
bundle exec rspec spec/ --format documentation -o rspec.log
```
で、テスト実施後にGemfileと同じ階層にrspec.logファイルが作成される。<br>
行が多すぎてターミナルで見れない時に使うと良い。

# EC2 について
## `rails s -p 8080` でブラウザが永遠に読み込み
pumaがうまく起動していないかもしれない。<br>
メモリ不足が原因かもしれない

### pumaのプロセスを確認する
```
ps aux | grep puma
```
a: すべてのユーザーのプロセスを表示<br>
u: ユーザー指向のフォーマット（CPU/メモリ使用量など）<br>
x: ターミナルに紐づかないバックグラウンドプロセスも含める
grep puma: puma という文字列を含む行だけを抽出。（grepのプロセスも表示される）

【正常なケース（Pumaが動作中）】
```
ec2-user 12345  0.5  2.1 1023456 51234 ?  Sl   14:20   0:02 puma 5.6.9 (tcp://0.0.0.0:8000) [your_app]
ec2-user 12350  0.0  0.0  119424   964 pts/0  S+   14:25   0:00 grep --color=auto puma
```
上: Pumaのプロセス<br>
下: grepのプロセス

【 異常なケース（Puma未起動）】
```
ec2-user 12350  0.0  0.0  119424   964 pts/0  S+   14:25   0:00 grep --color=auto puma
```
grepのプロセスしか表示されていない。

### ポート8000の使用状況を確認する
他にポート8000を使用していないか確認。
```
sudo lsof -i :8000
```
losf: プロセスが開いているファイルを表示するコマンド<br>
-i: ネットワークソケットファイルを指定する

これを実行して、戻りメッセージがなければ、他がポートを使用していることはない。

### メモリの使用状況を見る
```
free -m
```
メモリの利用状況を調べる。`-m`はメモリの量をMB単位で表示するオプション。

【正常なケース】
```
              total        used        free      shared  buff/cache   available
Mem:            952         343         277           0         331         450
```

【異常なケース】
```
              total        used        free      shared  buff/cache   available
Mem:            981         950          15          10          15          5
```
free や available が 極端に小さいことに注目。

- total:	システムの全物理メモリ量	952MB	ハードウェア制約
- used:	現在使用中のメモリ（アプリ＋OS）	499MB	過多なら要注意
- free:	完全に未使用のメモリ	252MB	単体では判断不可
- buff/cache: アプリが要求すれば即時解放可能な「疑似空き領域」
- available: 即座に利用可能なメモリ（free + キャッシュ解放可能量）	298MB	実質的な空きメモリ指標

ちなみに、「Swap」とは、物理メモリ（RAM）が不足した時に、ディスク領域を「仮想メモリ」として代用する仕組み。<br>
つまり、Swap.totalは仮想メモリの総量、Swap.usedは仮想メモリの使用量、Swap.freeは仮想メモリの使用可能な量。

### 最終手段
vscodeを閉じて、再びssh接続。(アプリの再起動)

それでもダメだったら

```
sudo reboot
```
にて再起動する（インスタンスの再起動）

## DB作成
既存のテーブルへのカラムの追加や削除を行う場合、テーブルを作成するマイグレーションファイルを直接編集するのは避ける！

### migrationファイルとは
テーブルの作成や変更を管理するファイル。このファイルで、テーブル名・カラム名を決定する。
`rails g model List`このコマンドによって、テーブル名が「lists」に自動的に決定される。

マイグレーションファイル`db/migrate/(作成日時)_create_lists.rb`のブロック内にて、
```
t.型 :カラム名
t.string :title
t.string :body
...
```
でカラム名を決定する。

```
rails db:migrate
```
でマイグレーションファイルからテーブルを作成する。<br>
この際、`db/schema.rb`が自動生成されるので、ちゃんとカラムができているか確認する癖をつけておくと良い。

### 新たに別のカラムを追加したい場合
```
rails g migration Addカラム名Toテーブル名 カラム名:型名

例 ⇩
rails g migration AddNameToLists name:string
rails db:migrate
```

### カラムを削除したい場合
```
rails g migration Removeカラム名Fromテーブル名 カラム名:型名

例 ⇩
rails g migration RemoveNameFromLists name:string
rails db:migrate
```

### マイグレーションファイルのステータスを表示する
```
rails db:migrate:status

⇩ 結果
 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20250406052431  Create lists
   up     20250406060900  Add name to lists
   up     20250406063258  Remove name from lists
```

up: マイグレーションファイルがmigrateされている状態。つまりこのマイグレーションが適用済み。<br>
down: migrateされていない状態。データベースに未適用の変更が待機している状態。

upの状態ではmigrateをすることができない。<br>
一度実行したマイグレーションファイルの中身を修正してmigrateしようとしてもうまくいかない

### 1つ前の状態に戻す（直前のマイグレーションを取り消す）
```
rails db:rollback
```

### 特定のマイグレーションファイルのタイムスタンプを指定して、その時点に戻す
```
rails db:migrate:down VERSION=20250401123456
```

### 全部リセットしてやり直す
```
rails db:migrate:reset
```
誤解ポイント: マイグレーション履歴がリセットされたり、初期のマイグレーションが適用されるのではなくて、テーブルがリセットされる。

【こんな時に使う】<br>
- 複数のマイグレーションファイルに修正が必要
- テーブルをすべて削除し、データをリセットしたい

## フラッシュメッセージ実装

### ３種のキー
- flash[:notice]	成功系のメッセージ（緑系など）
- flash[:alert]	エラー・警告系のメッセージ（赤系など）
- flash[:error]	より重大なエラー向け（alertに近い）

### 基本形
コントローラにて
```
flash[:notice] = "投稿に成功しました。"
```
ビューファイルapplication.html.erbにて
```
<%= flash[:notice] %>

<% if flash[:notice] %>
  <p class="notice"><%= flash[:notice] %></p>
<% end %>

<% flash.each do |message_type, message| %>
  <div class="flash-message <%= message_type %>">
    <%= message %>
  </div>
<% end %>
```

deviseを導入している場合、ログイン・ログアウト・サインアップなどのアクションが起こると、deviseが自動で表示される。

# ruby文法

## メソッド シンボル ブロック
```
create_table :lists do |t|
  t.string :title
  t.timestamps
end
```
これは、js風でいえば、
```
createTable("lists", (t) => {
  t.string("title");
  t.timestamps();
});
```
こんな感じ。

## form_withについて
フォームを作るための便利なヘルパーメソッド

## リンクを作る
```
<%= link_to 表示させるテキスト , リンク先URL [,オプション] %>
⇩ 例
<%= link_to list.title, list_path(list.id) %>
```

## 名前付きルート
`as: "名前"`<br>
名前付きルートがあると、その名前をredirect_toやlink_toでも使用することができます。
```
get 'lists/:id' => 'lists#show', as: 'banana'
⇩ ルートで上のように設定すると、下のように呼び出す。
banana_path(モデルのインスタンス | id)

例
banana_path(list.id)
⇩ この場合、下と同様になる
"/lists/#{list.id}"
```

## 画像のサイズ変更
「image_processing」というGemを用いて画像サイズの変更を行う。

```
# Gemfile
gem 'image_processing', '~>1.2'
```
を追記またはコメントアウト解放。その後、`bunlde install`

エラー回避のため、
```
# config/environments/development.rb
config.active_job.queue_adapter = :inline
```
を追記

## オプション
url: どのURLへフォームの情報を送信するか。<br>
method: HTTPメソッドをシンボルで指定<br>
なお、これら二つは、省略できる。<br>
model: モデルのインスタンスを入れる

###　注意事項
-  tableタグ直下、trタグ直下で使えない。

# パスについて
## ルーティングとは
このURL(パス)だったら、このプログラム(アクション)を実行する、みたいな感じ。
例えば、`http://127.0.0.1:8080/top`(/topだったら)の場合、topアクションを実行する(home#top)

/topにアクセスした場合
1. /topにアクセス。
2. /config/routes.rb で`/top`が使われているアクションを探す。
3. /app/controllers/homes_controller.rb でアクションを実行。
4. /app/views/homes/top.html.erb でHTMLをブラウザに表示。

ルーティングを指定するためには、routes.rbファイルを編集する。

## パスを調べる
```
rails routes

⇩ 実行結果
Prefix Verb   URI Pattern     Controller#Action
    top GET    /top(.:format)
```

- Prefix: パスの代わりとなる文字列。
- Verb: HTTPメソッド。GET, POST, PUT, DELETE。
- URI Pattern: ルーティングのうち「パス」を表す。(.:format) ... アドレスバー上では127.0.0.1:8080/topにアクセスした時のアクションを定義する、ということ。
- Controller#Action(アクション): ルーティングのうち「アクション」を表す。「コントローラ名#アクション」

## パスを調べるタイミング
- 新しくルーティングを作ったとき

## 注意
###　命名規則
モデルの名称は、先頭大文字: List<br>
コントローラの名称は、複数形かつ全て小文字: homes

## redirect_to と　reder の違い
redirect_toはアクションを新たに実行し、renderはアクションを新たに実行しない

エラーメッセージを扱う際にはrender、それ以外はredirect_toを使う!

### redirect_to
1. redirect_toがルーティングにURLを送る（内部的にはブラウザを介してルーティングにURLを送る）
2. ルーティングに送られてきたURLとHTTPメソッドを照らし合わせて、どのコントローラのどのアクションを実行するかを決める
3. アクションを実行する
4. ビューを表示する

### reder
renderで定義したビューファイルを表示する。

画面遷移はなく、表示されているHTMLが入れ替わるのみ。

renderするビューに必要なインスタンス変数は、あらかじめ用意しなくてはならない!

renderによって、コントローラからビューファイルにハンドルが移るイメージ。

## rails c

"rails c"では、主に下記三つのことを行うことができます。

1. 実装する前に実装を考えているメソッドの動きを確認する
2. テーブルにどんなデータが格納されているか確認する
3. createが正常に動作し、テーブルに値が格納されたか確かめる

```
rails c
```
によってプロンプトが表示される。

```
exit
```
で抜けられる。

## rspec導入
1. [ダウンロード](https://wals.s3.ap-northeast-1.amazonaws.com/curriculum/rails/rails6/spec_bookers1.zip)
2. spec_bookers1.zipを解凍し、アプリのappと同階層に置く。
3. Gemfileの group :test do の中を※1のように変更し、末行に`gem 'net-smtp'`を追加する
4. `bundle install`
5. test.rbを編集する(:stderrを:silenceに変更する)。※2
6. テスト用のデータベースを作成する。`rails db:migrate RAILS_ENV=test`
7. テストを行う。`bundle exec rspec spec/ --format documentation`

※1
```
group :test do
  gem 'capybara', '>= 2.15'
  gem 'rspec-rails'
  gem 'factory_bot_rails'
  gem 'faker'
end
```

※2
```
 # Print deprecation notices to the stderr.
  config.active_support.deprecation = :silence
```

## 追加でページを作成する手順。
途中で、Aboutページの作成するとしたら、

### 1:　コントローラを設定
既存の大元となるコントローラ(ここではhomesコントローラ)に、
```
def about
end
```
アクションを追加する。

### 2: ビューファイルを作成
app/views/homes/about.html.erb<br>
を作成する。

### 3: ルーティングを設定
```
get "homes/about", to: "homes#about", as: "about"
```
名前付きルート（パス名）をaboutにしている。

## アソシエーション 関連付け機能
テーブルとテーブルを結びつける。<br>
Railsでは「1:Nの関係」は、DBの構造の設計 + Railsの規約に従って、<br>
「モデル」に記載して、モデル間の関係性を機能として持たせる。<br>
Railsでは6種の関連付けがある。

以降は、Articleモデル（記事）とUserモデル（投稿者）を例として記載していく。

### belongs_to
記事視点でのArticleテーブルとUserテーブルの関系。<br>
記事は一人の投稿者に属している。
```
# app/models/article.rbにて
belongs_to :user
```
※belongs_toの引数は単数形！

### has_many
投稿者視点でのArticleテーブルとUserテーブルの関系。<br>
投稿者は記事をいくつも持っている。
```
# app/models/user.rbにて
has_many :articles, dependent: :destroy
```
※has_manyの第一引数は複数形！

dependent: destroy によって、userのあるデータを消すと、それに紐付いたarticlesを自動的に削除する。<br>
下記のコードでユーザーid:1を外部キーとして所持しているarticleをすべて削除できる。
```
@user = User.find(1)
@user.destroy
```

### 各テーブルが親子関係にある場合
アプリ/config/routes.rbにて
```
  resources :親テーブル, only: [:new, :create, ...など] do
    resources :子テーブル, only: [:create, ...など]
  end
```
このように書くと良い。

これによって、下記のように親子関係をURLとパラメータで明示できる。
```
POST /posts/:親_id/子s
```

また、ネストすることによって
```
<%= form_with model: [@親, @子] do |f| %>
↓ これに変換される
<form action="/親s/:id/子s" method="post">
```
のように、リストが連結される。この場合、createには親と子、両方のモデルの情報が必要になる。<br>
ネストされたリソースでは、URLに親リソースのIDが含まれる(/post_images/:post_image_id/post_comments)

ちなみに、
```
resource :favorite, only: [:create, :destroy]
```
のように単数系にすると、/:idがURLに含まれなくなります。<br>
resourceは「それ自身のidが分からなくても、関連する他のモデルのidから特定できる」といった場合に用いることが多いです。

## 正しい命名規則
Railsには以下のような命名規則がある：

| 概要                     | 名前の例               | 命名スタイル                | 備考                                                         |
|--------------------------|------------------------|-----------------------------|--------------------------------------------------------------|
| **モデル名**             | `User`                 | キャメルケース（単数形）     | クラス名なので最初大文字。データ1件を表す。                  |
| **モデルファイル名**     | `user.rb`              | スネークケース（単数形）     | `app/models`内に置かれる。                                  |
| **テーブル名**           | `users`                | スネークケース（複数形）     | DBのテーブル。マイグレーションファイルで定義される。         |
| **コントローラー名**     | `UsersController`      | キャメルケース（複数形）     | クラス名。複数のリソースを扱うため複数形。                   |
| **コントローラーファイル名** | `users_controller.rb`   | スネークケース（複数形）     | `app/controllers`内に置かれる。                             |
| **ビューのディレクトリ名**   | `users/`               | スネークケース（複数形）     | `app/views`以下に配置。コントローラー名と一致させる。        |
| **ルーティングのパス**       | `/users`, `/users/:id` | スネークケース（複数形）     | RESTfulなURLパス。                                           |

## バリデーション

### フォームより、画像が必ず入力されるようにする
app/models/モデル.rbより、
```
validates :カラム名, presence: true
```
対象のカラムが**空（nil や 空文字 ""）はNGとなる。

validatesメソッドによって、
```
<!--エラーメッセージ-->
<% if @book.errors.any? %>
  <div class="alert alert-danger" role="alert">
    <p class="alert-heading">
      <%= @book.errors.count %>errors prohibited this obj from being saved:
    </p>
    <ul>
    <% @book.errors.full_messages.each do |message| %>
      <li><%= message %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```
のような「.errors」メソッドが使えるようになる

### 複数回押せないように制限する(いいね機能)
app/models/モデル名.rbにて、
```
validates :カラム名1, uniqueness: {scope: :カラム名2}
↓ 例
validates :user_id, uniqueness: {scope: :post_image_id}
```
user_idとpost_image_idのペアが一意である（重複しない）状態に制限する。<br>
バリデーションにおいてuniquenessを指定することで、<br>
validatesメソッドの引数であるuser_idカラムの値が<br>
すでにテーブルに保存されている値と重複していないかをチェックしてくれるようになります。<br>
またscopeの指定もすることができ、このように記述することでuser_idとpost_image_idのペアに対して、<br>
すでに同じ値のペアがテーブルに保存されていないかを判定してくれます。

### カラムに文字数制限
app/models/モデル名.rbにて、
```
validates :カラム名, length: { maximum: 200 }
```
- length: { minimum: 10 } → 10文字以上
- length: { in: 10..200 } → 10〜200文字以内
- length: { is: 50 } → ちょうど50文字

### カラムに一意性を持たせる
app/models/モデル名.rbにて、
```
validates :カラム名, uniqueness: true
```

## 共通化
### 部分テンプレートのファイル名
Railsでは、ファイル名の先頭にアンダースコア（_）付きのファイルが、部分テンプレートファイルとして認識されます。
```
app/views/post_images/_list.html.erb
```
みたいな。

### 部分テンプレート用にコードを変更する
部分テンプレートファイル内でインスタンス変数を利用すると、<br>
controller側でインスタンス変数の名前や挙動を変更したとき、<br>
部分テンプレート側も変更しなければいけません。<br>
これでは、再利用しにくいテンプレートになってしまいますね。

部分テンプレートが呼び出されたときに、Viewから渡される変数が使えるように変更します。<br>
この変数には、ローカル変数を使います。

例えば、「@post_imagesをpost_images」のように変更する。

### 部分テンプレートファイルを呼び出せるようにする
大元のerbファイルから呼び出す場合、
```
<%= render [部分テンプレートファイルの指定], [ローカル変数]:[渡す値] %>
↓ 例
<%= render 'list', post_images: @post_images %>
```
とする。

`post_images/list.html.erb`のようなパスが本来の呼び出し方だが、<br>
拡張子は省略でき、同じフォルダ内から呼び出す場合はフォルダ名も省略できる。

`'../common/error.html.erb'`　のような相対パスはダメ！<br>
=> `'common/error'`と書くべき。

## params[:id]
params[:id]は、パスに含まれているidを取得するメソッドです。<br>
例えば、/user/3/editであれば、、params[:id]の中身は数値の"3"となります

## ストロングパラメータ
フォームのデータを制限する。

```
private
def フォームからのデータキー_params
  params.require(:フォームからのデータキー).permit(:カラム, :カラム1, ...)
end
```
- params: フォームから送られてきたデータが入ってる。
- require(:キー): キーの中にデータがあることを期待。
- permit(...): その中で、受け取ってOKなカラム（＝セキュリティ的に許可するやつ）を指定。

:キーは、`<%= form_with(model: @モデル) ...`の「@モデル」と対応する

ちなみに、ストロングパラメータがフォームの値によってエラーかどうかをバリデーションするわけではない。<br>
(errorメソッド発火はこれによるものではない。)

errorメソッドは、
- モデルインスタンス.save
- モデルインスタンス.valid?
- モデルインスタンス.update()
- モデルインスタンス.create()
などによりバリデーションされてから使える。

ちなみに、モデルインスタンスは、`モデル.new()`のことで、
- モデル クラスの 新しいインスタンス（≒オブジェクト） を作る
- データベースにはまだ保存されていない状態
- フォームに値を渡すためなどに使う

ちなみに、 .save!, .update!, .create!<br>
「!（バン）」がつくバージョンは、バリデーションに失敗したら例外（エラー）を発生させる。

## アクセス制限
### ログインしていない場合
ログイン認証が済んでいない状態で特定のページ以外の画面にアクセスしても、ログイン画面へリダイレクトする。また、ログイン認証が済んでいる場合には全てのページにアクセスすることができる。
```
class ApplicationController < ActionController::Base
  # except以外のページは、ログインしていない時にアクセスするとログイン画面へ遷移
  before_action :authenticate_user!, except: [:アクション名]
  ↓ 例
  before_action :authenticate_user!, except: [:top]
  ...
```
【重要】このコードは頭行に書く！

アクション名は、routes.rbやコントローラ内の、
```
[routes.rb]
root to: 'homes#top'          # :top に対応
get "homes/about", to: "homes#about", as: "about"  # :about に対応

[コントローラ]
class HomesController < ApplicationController
  def top    # ← :top が指す
  end

  def about  # ← :about が指す
  end
end
```
と対応する。

### 投稿の編集、削除ボタンは投稿したユーザにのみ表示
ビューファイルにて、
```
<% if モデルインスタンス.user == current_user %>
  投稿の編集、削除ボタン
<% end %>
```

### 他ユーザーを編集するのを防ぐ
他のユーザーでも直接URLを入力することで、ユーザー個人のページにアクセスできてしまうのを防ぐ。<br>
例: ユーザー(id: 1)が、ユーザー(id: 3)の「`https://~/user/3/edit`」にアクセスするのを防ぐ。

*** ミソ: ログインしているユーザーIDが、編集画面のパスに含まれるIDと一致しているかチェックする！ ***

params[:id]とdeviseを使う。

app/controllers/users_controller.rbにて、
```
private

...

# 他人のアクセス防止
def is_matching_login_user
  user = User.find(params[:id])
  unless user.id == current_user.id
    redirect_to リダイレクト先_path
  end
end
```
is_matching_login_userメソッドを追記し、

/app/controllers/application_controller.rbにて、
```
class UsersController < ApplicationController
  before_action :is_matching_login_user, only: [:edit, :update]
  ...
```
を追記する。

## Railsアプリの初期設定
/config/initializersにて、
- application.rb:	全ての環境で共通して使用する設定
- environment/:	開発環境や本番環境など、環境ごとの設定ファイルを保存するフォルダ。development.rb, test.rbなど
- initializers/:	RailsやGemに関する細々とした初期設定ファイルを保存するフォルダ
- locales/:	複数言語の対応など、国際化対応に必要なファイルを保存するフォルダ

/config/initializersフォルダは、Railsアプリを起動したときに、一緒に読み込まれる設定ファイルを集めたものです。<br>
Gemをインストールしている場合、初期設定ファイルが/config/initializersに格納されることもあります。<br>

## 削除機能
```
<%= link_to "Destroy", モデル_path(モデルインスタンス.id), method: :delete, data: { confirm: "Are you sure?"} %>
```

### 注意
confirmはバベルをインストールしないと使えない。
`yarn add @babel/plugin-proposal-private-methods @babel/plugin-proposal-private-property-in-object`