# FontAwesomeについて
https://fontawesome.com/kits

## 導入する
app/views/layouts/application.html.erbにて
```
<!-- xxxxxxxxxxには自身のKitCodeのIDが入ります  -->
<script src="https://kit.fontawesome.com/xxxxxxxxxx.js" crossorigin="anonymous"></script>
```
を追記。

※css読み込みより前に記述する。

## 使い方
```
<%= link_to リンク先path do %>
  <i class="fontawesomeのクラス"></i>
<% end %>

<%= link_to リンク先path, class="fontawesomeのクラス" %>
```