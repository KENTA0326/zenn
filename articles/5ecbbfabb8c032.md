---
title: "検索機能"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rails]
published: true
---

# はじめに
　後回しにしていた検索機能を爆速で作ったためこれをまとめていきたいと思います。

## イメージ
![](https://storage.googleapis.com/zenn-user-upload/8a870b325cc4-20240124.png)

## 方法
ransackを用いて実装します。
```html:Gemfile
gem 'ransack'
```
これを記述し、bundle installします！

次に実装したいコントローラのindexアクションに以下のように記述します。
```html:***/controller.rb
def index
    @q = User.ransack(params[:q])
    @users = @q.result(distinct: true)
end

```

そして検索機能を表示したい場所に以下の記述をします。
```html:index.html.erb
<%= search_form_for @q do |f| %>
    <%= f.label :name_cont %>
    <%= f.search_field :name_cont %>

    <%= f.submit %>
<% end %>
```

## 実装
![](https://storage.googleapis.com/zenn-user-upload/0befeddfcd50-20240124.png)
あとから色々付け加える予定ですが、今はこんな感じにレイアウトを整えました。



