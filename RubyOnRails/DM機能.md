# DM機能の実装
※userモデルが存在している、フォロー機能実装済みの前提

- [ ] ユーザ同士で 1 対 1 の DM ができるようにする(※ 140 字まで送信可能にする)
- [ ] 相互フォローしている人限定で DM 機能を使えるようにする

## モデル設計
必要なモデルは、
1. users: ユーザー個人の情報
2. rooms: 個別のチャットルーム
3. chats: チャットメッセージの保存
4. user_rooms: 中間テーブル。どのユーザーがどのチャットルームにいるか管理

users:rooms = 多:多 となるので、
- users:1 => 多:chats:多 <= 1:rooms
- users:1 => 多:user_rooms:多 <= 1:rooms
というリレーション。

### rooms
| カラム名      | データ型 | カラムの説明                 |
|--------------|----------|------------------------------|
| id           | integer  | ルームごとのID              |

### chats
| カラム名      | データ型 | カラムの説明                 |
|--------------|----------|------------------------------|
| id           | integer  | チャットごとのID              |
| user_id      | references  | usersのFK              |
| room_id      | references  | roomsのFK              |
| message      | string(140字制限のためstring)  | メッセージ|

※stringは255文字まで・軽い、textは無制限・重め

### user_rooms
| カラム名      | データ型 | カラムの説明                 |
|--------------|----------|------------------------------|
| id           | integer  | ルームごとのID              |
| user_id      | references  | usersのFK              |
| room_id      | references  | roomsのFK              |

## モデル作成
```
rails g model Room

rails g model Chat user:references room:references message:string

rails g model UserRoom user:references room:references
```
そして、`rails db:migrate`でマイグレーション。

`:references`: 外部キーとして生成するためのもの。外部キー制約がつく。自動でアソシエーションしてくれる。

## アソシエーション設定
/app/models/user.rb
```
  # DM機能に必要な関連
  has_many :chats, dependent: :destroy
  has_many :user_rooms, dependent: :destroy
  has_many :rooms, through: :user_rooms
```
を追記。

`has_many :rooms, through: :chats`にしない理由<br>
=> chatsはメッセージを投稿しなければレコードが何もない。レコードが何もない場合、roomsとの関連がなくなってしまう。「ユーザーが参加しているルーム」にならず「ユーザーがメッセージを送信したルーム」になってしまい、参加しているがメッセージを送っていないルームは取得できなくなる。

/app/models/room.rb
```
class Room < ApplicationRecord
  has_many :user_rooms, dependent: :destroy
  has_many :chats, dependent: :destroy
end
```

/app/models/chat.rb
```
class Chat < ApplicationRecord
  belongs_to :user
  belongs_to :room

  # 要件の140字制限
  validates :message, presence: true, length: { maximum: 140 }
end
```

/app/models/user_room.rb
```
class UserRoom < ApplicationRecord
  belongs_to :user
  belongs_to :room
end
```

とする。(自動的にそうなっている分には要チェック)

## コントローラ作成
```
rails g controller chats
```

## ルーティング設定
/config/routes.rb
```
resources :chats, only: [:show, :create, :destroy]
```
を追記。
```
chats POST   /chats(.:format)      chats#create
chat  GET    /chats/:id(.:format)  chats#show
      DELETE /chats/:id(.:format)  chats#destroy
```
みたいになってたらOK。

## コントローラ設定
/app/controllers/chats_controller.rb
```
class ChatsController < ApplicationController
  before_action :block_non_related_users, only: [:show]

  # チャットルーム表示のアクション
  def show
    # チャット相手のユーザを取得
    @user = User.find(params[:id])

    # 現在のユーザーが参加しているチャットルーム一覧の取得
    rooms = current_user.user_rooms.pluck(:room_id)

    # 相手ユーザーとの共有チャットルームが存在するか確認。
    user_rooms = UserRoom.find_by(user_id: @user.id, room_id: rooms)

    #ここまでの流れ。例えば下記のようなテーブルになっているとして。
    # user_rooms
    # | user_id | room_id |
    # | :------- | :------- |
    # | 1        | 1        |
    # | 1        | 2        |
    # | 2        | 1        |
    # | 3        | 5        |
    # | 4        | 8        |
    # current_userは1、@userは2だとすると、
    # rooms = [1, 2]となる
    # ①UserRoom.find_by(user_id: 2 => {user_id: 2, room_id: 1})となる。
    # ②UserRoom.find_by(room_id: [1, 2]) だと↓のようなレコードが候補となる。
    # | 1        | 1        |
    # | 1        | 2        |
    # | 2        | 1        |
    # user_id: 2 かつ room_id: 1 か 2 の両方満たすレコードは
    # | 2        | 1        |
    # とうことで、UserRoom.find_by(user_id: @user.id, room_id: rooms)は
    # {user_id: 2, room_id: 1} のような感じとなる。

    unless user_rooms.nil?
      # 共有チャットルームが存在する場合、そのチャットルームをインスタンス変数で渡す
      @room = user_rooms.room
    else
      # 共有チャットルームが存在しない場合、新しいチャットルームを作成
      @room = Room.new
      @room.save

      # チャットルームに現在のユーザーと相手ユーザーを追加
      UserRoom.create(user_id: current_user.id, room_id: @room.id)
      UserRoom.create(user_id: @user.id, room_id: @room.id)
    end

    # チャットルームに関連付けられたメッセージを取得
    @chats = @room.chats

    # 新しいメッセージを作成するための空のChatオブジェクトを生成
    @chat = Chat.new(room_id: @room.id)
  end

  # チャットメッセージの送信するアクション
  def create
    # フォームから送信されたメッセージを取得し、現在のユーザーに関連付けて保存
    @chat = current_user.chats.new(chat_params)

    # バリデーションに合格しない場合はエラーを表示(= validate.html.erb を描画する)
    render :validate unless @chat.save
  end

  # チャットメッセージの削除するアクション
  def destroy
    # ログイン中のユーザーに関連するチャットメッセージを削除
    @chat = current_user.chats.find(params[:id])
    @chat.destroy
  end

  private

  # フォームからのパラメータを制限
  def chat_params
    params.require(:chat).permit(:message, :room_id)
  end

  def block_non_related_users
    # チャット相手のユーザを取得
    user = User.find(params[:id])

    # ユーザーが互いにフォローしているか
    unless current_user.following?(user) && user.following?(current_user)
      # していなければ、前の画面にリダイレクト
      redirect_to request.referer
    end
  end
end
```

### pluckメソッド
```
モデル名.pluck(:カラム名)
```
ActiveRecordのメソッド。SQLで無駄なデータを取らず、欲しいカラムだけ取得する。<br>
戻り値はカラムの値を格納した配列。

SQLでは
```
SELECT カラム名 FROM テーブル名;
```
となる

### find_byメソッド
```
モデル名.find_by(カラム名: 値)
モデル名.find_by(カラム名1: 値, カラム名2: 値)
```
引数に条件を指定して、最初の1件だけ レコードを取ってくるメソッド。<br>
戻り値はオブジェクト。

SQLでは
```
SELECT * FROM テーブル名 WHERE 条件 LIMIT 1;
```
となる

## ビュー設定
### チャット開始ボタン
/app/views/chats/_start_btn.html.erb
```
<%# ユーザーshowページに居て、お互いにフォローし合ってないとリンクは表示されない %>
<% if current_user != user && current_user.following?(user) && user.following?(current_user)  %>
  <%= link_to 'chatを始める', chat_path(user.id), class: "ml-3" %>
<% end %>
```

### チャットコメント
/app/views/chats/_chat.html.erb
```
<%# 自分が送るメッセージ 右よせ %>
<div class="right-container d-flex justify-content-end">
  <p id="chat_<%= chat.id %>"><%= chat.message %></p>
  <%= button_to "削除", chat_path(chat), method: :delete, data: { confirm: "本当に削除しますか？" }, remote: true, id: "delete_chat_#{chat.id}" %>
</div>
```
id="chat_<%= chat.id %>とid: "delete_chat_#{chat.id}"でメッセージごとに異なるIDをつける。これによって、消したいメッセージを非同期で削除できる。

### チャットルーム(show)
/app/views/chats/show.html.erb
```
<h2 id="room" data-room="<%= @room.id %>" data-user="<%= current_user.id %>"><%= @user.name %> さんとのチャット</h2>

<div class="message" style="width: 400px;">
  <% @chats.each do |chat| %>
    <% if chat.user_id == current_user.id %>
      <%= render "chats/chat", chat: chat %>
    <% else %>
      <div class="left-container d-flex">
        <%= image_tag @user.get_profile_image, size: '30x30', style: 'border-radius: 50%' %>
        <p class=ml-1 style="text-align: left;"><span style="background-color: #F5F5DC; padding: 5px; border-radius: 10px;"><%= chat.message %></span></p>
      </div>
    <% end %>
  <% end %>
</div>

<div class="errors">
  <%= render "layouts/errors", obj: @chat %>
</div>

<%= form_with model: @chat, data: {remote: true} do |f| %>
  <%= f.text_field :message %>
  <%= f.submit "送信", class: "btn btn-dark btn-sm" %>

  <%# 隠しフィールド: これを使うことで、ユーザーには表示したくない情報も一緒に送る。チャットルームIDを一緒に送信する。メッセージを送信する際、どのチャットルームにメッセージを送るのかという情報が必要。 %>
  <%= f.hidden_field :room_id %>
<% end %>
```

### create.js.erb
/app/views/chats/create.js.erb
```
<%# メッセージを追加 %>
$(".message").append("<%= j(render "chats/chat", chat: @chat) %>")

<%# フォームを空にする %>
$("input[type=text]").val("")
```

### destroy.js.erb
/app/views/chats/destroy.js.erb
```
<%# メッセージを追加 %>
$(".message").append("<%= j(render "chats/chat", chat: @chat) %>")

<%# フォームを空にする %>
$("input[type=text]").val("")
```

### validate.js.erb
/app/views/chats/validate.js.erb
```
<%# 空のメッセージ等を送るとエラー表示を出す %>
$(".errors").html("<%= j(render "layouts/errors", obj: @chat) %>");
```

## TIPS
### `rails db:migrate`した後に、マイグレーションファイルを修正する場合
必要なカラムがなく、後で追加するケース。

例えば、`rails g model Chat user:references room:references message:string`を実行したつもりが、/db/migrate/20250509084850_create_chats.rbマイグレファイルに
```
t.references :room, null: false, foreign_key: true
```
が欠落していて、そのまま`rails db:migrate`した、とする。

対応手順<br>
① マイグレーションファイルを作成する。
```
rails g migration Addカラム名Toテーブル名 追加したいカラム:型
↓ 例
rails g migration AddRoomIdToChats room:references
```

Add〇〇To〇〇: マイグレーション・ジェネレータの命名規約

② 生成されたマイグレーションファイルを確認する<br>
欠落したカラムがちゃんと記述されているかチェック。

③ そしてマイグレーションを実行<br>
```
rails db:migrate
```
そして、schema.rbを確認する。

===

アナザー対応手順（アプリリリース後はやっちゃダメ。開発段階のみ）
1. マイグレファイルの欠落した部分を正しく書く。
2. `rails db:migrate:status`を実行し、当該マイグレファイルが何番目のマイグレか確認する。
3. n番目だと分かったら、`rails db:rollback STEP=n番目`でロールバックする。
4. `rails db:migrate`でマイグレし直す。
5. /db/schema.rbで正しいテーブルになっているかを確認。