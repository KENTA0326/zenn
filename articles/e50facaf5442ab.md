---
title: "form_with と form_tag, form_for の違い"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rails]
published: true
---

# はじめに
`form_tag`と`form_for`は現在のrails では非推奨になり、将来的に`form_with`に完全に書き換えられるそうです。

## form_tagとform_for

投稿フォームなどを作成したい時関連するモデルがない時はform_tag、ある時はform_forを使用してきました。

```
<%= form_tag users_path do %>
  <%= text_field_tag :email %>
  <%= submit_tag %>
<% end %>
```

```
<%= form_for @user do |form| %>
  <%= form.text_field :email %>
  <%= form.submit %>
<% end %>
```

## form_with

上記のものとは違い関連すもモデルがあってもなくても使用できるのがform_withです。

```
<%= form_with model: @user do |form| %>
  <%= form.text_field :email %>
  <%= form.submit %>
<% end %>
```

モデルを渡したときは、URLとスコープが自動推測されます。(これまでのform_for と同じ。)
URL: @userがDBに存在するときはupdateアクションに、ないときはcreateアクションに飛びます。
