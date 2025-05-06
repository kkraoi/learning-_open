# Bootstrapについて

https://getbootstrap.com/docs/4.6/getting-started/introduction/

## インストール手順

### 1: yarnでインストール
```
yarn add jquery bootstrap@4.6.2 popper.js
```

### 2: 環境ファイル編集
config/webpack/environment.jsにて
```
...

const webpack = require('webpack')
environment.plugins.prepend(
  'Provide',
  new webpack.ProvidePlugin({
    $: 'jquery/src/jquery',
    jQuery: 'jquery/src/jquery',
    Popper: 'popper.js'
  })
)
```
を追記。

### 3: scssファイル編集
app/javascript/stylesheets/application.scss を作成し、<br>
（`mkdir app/javascript/stylesheets/ & touch app/javascript/stylesheets/application.scss`）
```
@use '~bootstrap/scss/bootstrap';
```
を記述。

### 4: jsファイル編集
app/javascript/packs/application.js にて
```
...
import "channels"

# ここから
// Bootstrap
import "jquery";
import "popper.js";
import "bootstrap";
import "../stylesheets/application";
# ここまで

Rails.start()
...
```
を追記。

### 5: ビューファイル編集
app/views/layouts/application.html.erbにて
```
<%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
↓　これを
<%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
```
に変更。

stylesheet_link_tagの場合cssファイルは、app/assets 配下のファイルを参照します。<br>
stylesheet_pack_tagの場合は、app/javascript 配下のファイルを参照するようになります。

ちなみに、stylesheet_pack_tagのみの記述で独自のcssを読み込ませる方法を紹介しておきます。<br>
1. app/javascript/stylesheets/application.scssにcssを記述する。
2. app/javascript/packs/application.jsに追加したcssファイルをimport で追加する。

例: app/javascript/stylesheets/にmystyle.cssを作成した場合<br>
packs/application.jsにて
```
import '../stylesheets/mystyle.css'
```

### クラス番号
スペースユーティリティ（p-*, m-*, gap-* など）

- 0 => 0rem, 0px
- 1 => 0.25rem, 4px
- 2 => 0.5rem, 8px
- 3 => 1rem, 16px
- 4 => 1.5rem, 24px
- 5 => 3rem, 48px

この6段階のみ。

## グリッドシステム
Bootstrapのグリッドシステムでは、兄弟要素（同じ row 内の col-* クラスを持つ要素）の合計値が12を超えないことが基本ルール。
```
<div class="row">
  <div class="col-6">50%幅（6/12）</div>
  <div class="col-4">33.3%幅（4/12）</div>
  <div class="col-2">16.6%幅（2/12）</div>
</div>
```


## 覚えておくと良い
### ナビゲーション: バー
```
<nav class="navbar navbar-expand-lg navbar-light">
```
- navbar: ナビゲーションバーの表示
- navbar-expand-[ブレークポイント]: 指定したブレークポイント（画面幅）に応じてナビゲーションバー、ナビゲーションアイコン（ハンバーガーメニュー）を切り替えて表示
- navbar-[カラースキーム]: ナビゲーションのカラースキームの指定

### ナビゲーション: ロゴ
```
<a class="navbar-brand p-3" href="/"><%= image_tag('logo.png') %></a>
```
- avbar-brand: ロゴに使用されるnavbarのサポートclass

### ナビゲーション: ハンバーガーメニュー
```
<button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation">
  <span class="navbar-toggler-icon"></span>
</button>
```
- navbar-toggler: ナビゲーションアイコン（ハンバーガーメニュー）の枠部分
- navbar-toggler-icon: ナビゲーションアイコン（ハンバーガーメニュー）の三本線部分。<br>※この三本線はnavbar-toggler-iconのclassに対してbackground-imageで画像が割り当てられています。

### ナビゲーション: 閉じたり開いたりする部分
ナビゲーションアイコンの開閉する動作についてはBootstrapとjQueryが内部的に行っている部分です。

```
<button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation">
  <span class="navbar-toggler-icon"></span>
</button>
<div class="collapse navbar-collapse" id="navbarNavDropdown">
  <ul class="navbar-nav ml-auto">
  ...
  </ul>
</div>
```
- ナビゲーションアイコンを開く前に隠しておきたいメニューは「collapse」classが付与された要素の中に入れておく
- 「collapse」classを付与している要素に任意のidを付与する
- 付与したidは「navbar-toggler」classが付与される要素のdata-targetに指定する（頭にセレクタ#をつける）
- 「navbar-toggler」classが付与される要素にdata-toggle="collapse"、data-target="#[id名]"、aria-controls="[id名]"、aria-expanded="false"、aria-label="Toggle navigation"

※この内data-toggle="collapse"、data-target="#[id名]"は必ず指定しなければ、ナビゲーションアイコンの開閉を行うことができませんので、注意しましょう。

### カード
「card」classには以降の子要素で使用するclassが依存する形になります。<br>
この「card」classを指定さえすれば、その中の要素で使用するものは自由に組むことができます。

```
<div class="card" style="width: 18rem;">
  <!-- 画像（オプション） -->
  <img src="image.jpg" class="card-img-top" alt="...">

  <!-- カード本体 -->
  <div class="card-body">
    <h5 class="card-title">カードタイトル</h5>
    <p class="card-text">説明文が入ります。</p>
    <a href="#" class="btn btn-primary">ボタン</a>
  </div>
</div>
```
- .card: カードの外枠（背景色・ボーダー・角丸を付与）。
- .card-img-top / .card-img-bottom: カードの上下に画像を固定表示。
- .card-body: テキストやボタンを配置するコンテンツ領域。
- .card-title / .card-text: タイトルと本文用のテキストスタイル。

【cardの主な用途】
- 商品アイテムの表示: 画像・価格・説明をまとめた商品カード。
- プロフィール/ブログ記事: ユーザー情報や記事のプレビュー。
- ダッシュボードのウィジェット: 統計データや通知をカードで配置。
- 画像ギャラリー: サムネイルとキャプションのセット。

https://getbootstrap.jp/docs/4.5/components/card/