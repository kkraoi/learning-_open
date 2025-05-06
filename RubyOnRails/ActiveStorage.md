# ActiveStorageとは
Railsで画像の投稿や表示を行うためのものです。
画像は通常のカラムとして保存できないため、特別な保存方法が必要になり、それを行うのがActiveStorageです。

Railsに元から含まれている機能だから、Gemfileに追加する必要はない。

```
rails active_storage:install
```
でインストール、マイグレーションファイルができる。

この1行のコマンドで、Active Storage が次のようなことを自動でやってくれる。
- モデル(小文字).image.attach(...) で画像のアップロードができる
- モデル(小文字).image.attached? で画像があるか確認できる
- モデル(小文字).image で画像データにアクセスできる
- モデル(小文字).image.blob や .filename などでメタ情報も取得可能

その後、
```
rails db:migrate
```
でマイグレートする。

その後、models/モデルファイル:モデル名(小文字).rbに、
```
has_one_attached :image
```
を書き込んで、imageカラムが追記されたかのように扱われるようにする。

## 画像が投稿されていない場合はエラーが出る (デフォルトの画像設定)
エラーを回避するためにsample_appでは以下のようにビューでif式を使っていました。
```
<% if list.image.attached? %>
    <%= image_tag list.image, size: "200x200" %>
<% else %>
    <%= image_tag 'no_image', size: "200x200" %>
<% end %>
```
これでは画像の表示をする際に毎回同じ記述を書く必要があり、開発効率も可読性も落ちてしまいます。<br>
そこで、models/モデルファイル:モデル名(小文字).rbに、
```
  ...略
  ##
  # @モデル名に含まれるイメージを表示させるメソッドを実行する
  # @return [String] 画像ファイルの文字列
  def get_image
    if image.attached?
      image
    else
      'no_image.jpg'
    end
  end
end
```
get_imageメソッドを作ってやると良い。
```
@モデル名(小文字).get_image
```

もしくは、上記のget_imageメソッドは、Railsで画像のサイズ変更を行うことができないため、
```
  # @モデル(小文字)に含まれるイメージを表示させるメソッド
  #
  # no_image.jpgをActiveStorageに格納する。
  # @return [Object] ActiveStorage の添付ファイルオブジェクト。
  def get_image
    unless image.attached?
      file_path = Rails.root.join('app/assets/images/no_image.jpg')
      image.attach(io: File.open(file_path), filename: 'default-image.jpg', content_type: 'image/jpeg')
    end
    image
  end
```
とする。

もしくは、ImageMagickとimage_processingを使用した場合、
```
has_one_attached :画像カラム名

# 画像をリサイズして返す。画像が添付されてなければ、デフォルト画像(jpeg)を返す。
#
# ImageMagickとimage_processingのインストールが必要。
# @param width [Integer] 画像の幅
# @param height [Integer] 画像の高さ
# @return [ActiveStorage::Variant] 投稿されたリサイズ済み画像 または デフォルト画像
def get_画像カラム名(width, height)
  unless 画像カラム名.attached?
    file_path = Rails.root.join('app/assets/images/default-profile_iamage.jpg')
    画像カラム名.attach(
      io: File.open(file_path),
      filename: 'default-image.jpg',
      content_type: 'image/jpeg'
    )
  end
  # 引数から受け取った値で画像サイズの変換を行なっている。
  画像カラム名.variant(resize_to_limit: [width, height]).processed
end

↓ erbファイルにて
<%= image_tag @モデルインスタンス.get_画像カラム名(100, 100) %>
```
なぜ、 image_tag のsizeを使わないのか?
