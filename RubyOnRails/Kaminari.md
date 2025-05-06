# Kaminariとは
Kaminariはページネーションに必要な機能を効率的に提供するGem（ライブラリ）です。

https://github.com/kaminari/kaminari

## 主な機能
- ビューヘルパー: ページネーションのリンクを表示するためのHTMLコードを生成
- メソッド: モデルのクエリをページネーションに対応する形式に変換する
- クエリの最適化: 取得数などのカスタマイズ

## インストール
1. Gemfileの最終行に、次のコードを追加して保存します。`gem 'kaminari','~> 1.2.1'`
2. `bundle install`
3. `rails g kaminari:config` にてkaminariの設定ファイルを作成する。
4. `rails g kaminari:views default` にてkaminariがページャで利用するテンプレートを作成を作成。

## 実装する
### コントローラにて
app/controllers/実装するモデル_controller.rb にて、
```
def index
  @post_images = モデル.all
end
↓　こうする
def index
  @post_images = モデル.page(params[:page])
end
```
1ページ分の決められた数のデータだけを、新しい順に取得するように変更しています。<br>
pageメソッドは、kaminariをインストールしたことで使用可能になったメソッドです。

### ビューにて
app/views/モデル/〇〇.html.erbにて、
```
<%= paginate モデルインスタンス %>
```
とする。

## カスタマイズ
config/initializers/kaminari_config.rb にて、

### 1ページの表示件数を設定
```
  ...
  config.default_per_page = 5

end
```
こうしたら、5件分表示される。