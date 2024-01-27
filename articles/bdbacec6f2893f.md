---
title: "DM通知機能実装"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rails]
published: true
---

# はじめに
　DMにも通知機能をつけてみようと思って付けてみました。

## コントローラ
```html:app/controllers/public/messages_controller.rb
・
・
・

# 新規作成された@messageに紐づくroomを@roomに格納する
   @room = @message.room
# 本引数を２つ持たせてcreate_notification_dmメソッドを実行
 @room.create_notification_dm(current_user, @message.id)
・
・
・

```
meaasageとroomは１対多の関係なのでアソシエーションの関係を使うと
上記に使い方ができるよ！！

create_notification_dmは
これからRoomモデルに定義する
create_notification_dmメソッドを呼び出しており
実行後、roomモデルに動作が移るよ！

(current_user, @message.id)は
Notificationモデルに
・現ユーザーのidを保存
・メッセージを保存
するために、メソッドに渡す本引数として設定。

## 追加したカラム

```html:model.rb
t.integer :room_id
t.integer :message_id
```

## 追加したモデル
モデルは一覧を載せていきます。
```html:app/models/message.rb

class Message < ApplicationRecord
  has_many :notifications, dependent: :destroy
  belongs_to :user
  belongs_to :room
end


```

```html:app/models/message.rb

class Message < ApplicationRecord
  has_many :notifications, dependent: :destroy
  belongs_to :user
  belongs_to :room
end


```

```html:app/models/room.rb

class Room < ApplicationRecord
  has_many :notifications, dependent: :destroy
  has_many :messages, dependent: :destroy
  has_many :entries, dependent: :destroy


```

```html:app/models/notification.rb

class Notification < ApplicationRecord
  default_scope -> { order(created_at: :desc) }
  belongs_to :room, optional: true
  belongs_to :message, optional: true
  belongs_to :visitor, class_name: 'User', optional: true
  belongs_to :visited, class_name: 'User', optional: true
end


```

## create_notification_dmメソッドの定義

roomモデルの通知データを保存するためのメソッドを実装していきます。

```html:app/models/room.rb

class Room < ApplicationRecord
  has_many :notifications, dependent: :destroy
  has_many :messages, dependent: :destroy
  has_many :entries, dependent: :destroy


  def create_notification_dm(current_user, message_id)
    # ルームに関連する全てのエントリーレコードを取得
    @multiple_entry_records = Entry.where(room_id: id).where.not(user_id: current_user.id)
    # ルームに関連する単一のエントリーレコードを取得
    @single_entry_record = @multiple_entry_records.find_by(room_id: id)
    
    # メッセージの送信者以外のユーザーに通知を作成
    notification = current_user.active_notifications.build(
      room_id: id,
      message_id: message_id,
      visited_id: @single_entry_record.user_id,
      action: 'dm'
    )

    notification.save if notification.valid?
  end

end


```

## viewページ
```html:app/views/notifications/_notification.html.erb
<% notifications.each do |notification| %>
  <% visitor = notification.visitor %>
  <% visited = notification.visited %>
  <div class="col-md-12 mx-auto notification-container">
    <div class="notification-box align-items-center">
      <div class="box16">
       <p><!-- 通知の種類に応じて表示内容を切り替え -->
      <% case notification.action when 'follow' then %>

      <!-- フォロー通知の場合 -->
        <strong><%= visitor.name %></strong>さんがあなたをフォローしました
      
      <!-- DMが送られた通知の場合 -->
      <% when 'dm' then %>
      <p>あなたに</p>
        <h5><i class="fas fa-envelope"></i>DMを送りました</h5>
        <span>メッセージ内容:</span>
          <%= truncate(notification.message.message, length: 20, omission: '... (一部表示)') %>
        <%= link_to room_path(notification.room) do %>
          <h4>DMの送信画面に移動</h4>
        <% end %>
        <hr>

      <!-- 投稿がいいねされた通知の場合 -->
      <% when 'favorite_post' then %>
        <%= link_to post_path(notification.post) do %>
          <strong><%= visitor.name %></strong>さんが<strong><%= truncate(notification.post.text, length: 10) %></strong>の投稿にいいねしました
        <% end %>

      <!-- コメントが投稿された通知の場合 -->
      <% when 'post_comment' then %>
        <!-- 自分自身の投稿へのコメントの場合 -->
        <% if notification.post.user_id == visited.id %>
          <%= link_to post_path(notification.post) do %>
            <strong><%= visitor.name %></strong>さんが<strong><%= truncate(notification.post.text, length: 10) %></strong>の投稿にコメントしました
            <% post_comment = PostComment.find_by(id: notification.post_comment_id) %>
            <div class="notification-comment">
              <%= truncate(post_comment&.comment, length: 30) %>
            </div>
          <% end %>

        <!-- 投稿者が異なり自身がコメントした投稿に他のユーザーがコメントした場合 -->
        <% else %>
          <span>
            <%= link_to post_path(notification.post) do %>
              <strong><%= visitor.name %></strong>さんが<strong><%= notification.post.user.name + 'さんの投稿' %></strong>にコメントしました
              <% post_comment = PostComment.find_by(id: notification.post_comment_id) %>
              <div class="notification-comment">
                <%= truncate(post_comment&.comment, length: 30) %>
              </div>
            <% end %>
          </span>
        <% end %>
      <% end %></p>
      </div>
    </div>
    <div class="small text-muted text-right">
      <%= notification.created_at.in_time_zone('Tokyo').strftime('%Y-%m-%d') %>
    </div>
  </div>
<% end %>
```
他の時と場合分けを行っていたためここにDM通知を追加しました。






