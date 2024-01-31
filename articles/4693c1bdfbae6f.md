---
title: "カート内非同期通信"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rails javascript]
published: true
---

# はじめに
　先月作成したECサイトの中で非同期にできそうなものがあったため
挑戦したいと思います。

## 実装
viewファイル
```html:cart_items.html.erb
 <% @cart_items.each do |cart_item| %>
  <tr id="cart_item_<%= cart_item.id %>">
   <% @nedan = @nedan + cart_item.subtotal %>
    <td>
      <%= image_tag cart_item.item.item_image, size:'50x50' %>
      <%= cart_item.item.name %>
    </td>
    <td><%= cart_item.item.with_tax_price.to_s(:delimited)%>円</td>
    <td id="select-form">
      <% cart_item.amount %>
      <%= form_with model: cart_item, url: cart_item_path(cart_item), method: :patch do |f| %>
       <%= f.select :amount, *[1..10] %>
       <%#= f.submit "変更", class: "btn btn-success text-white" %>
      <% end %>
      </td>
      <td class="subtotal-area"><%= render 'subtotal', cart_item: cart_item %></td>
      <td><%= link_to "削除する", cart_item_path(cart_item), method: :delete, class: "btn btn-danger text-white" %></td>
  </tr>
<% end %>
・
・
・
<tr>
   <td class="table-warning">合計金額</td>
   <td id="total-area"><%= render 'total', nedan: @nedan %></td>
</tr>
・
・
・
```
`tr id="cart_item_<%= cart_item.id %>`
* ここでtrタグにidを持たせます。

続いて非同期通信にさせたいところにid,classを指定します。

このviewページの下部分にAjaxの部分の記述を記載します。
```
<script>
  $(document).on("change", "#select-form select", function() {
    let form = $(this).closest("form");
    let url = form.attr("action");
    let method = form.attr("method");
    let data = form.serialize();

    $.ajax({
      url: url,
      method: method,
      data: data,
      success: function(response) {
        let cartItemRow = form.closest("tr#cart_item");
        cartItemRow.find(".subtotal-area").html(response);
      },
      error: function(xhr) {
        console.log(xhr.responseText);
      }
    });
  });
</script>
```

`$(document).on("change", "#select-form select", function() { ... });`
select-formクラスを持つセレクトボックスが変化するときに作動します。
つまり、カート内の個数が変更されたときにこのイベントが発火します。

`let form = $(this).closest("form");`
変更されたセレクトボックスに最も近いform要素を取得しています。

`let url = form.attr("action");`
フォームのaction属性を取得し、リクエストの送信先URLとして使用します。

`let method = form.attr("method");`
フォームのmethod属性を取得し、リクエストのHTTPメソッドとして使用します。

`let data = form.serialize();`
フォームのデータをシリアライズし、リクエストのデータとして使用します。

`$.ajax({ ... });`
Ajaxリクエストを送信します。指定されたURL、メソッド、データでリクエストを行います。

`success: function(response) { ... }`
リクエストが成功した場合のコールバック関数です。レスポンスデータを受け取り、変更されたカートアイテムの行の小計を更新します。

`error: function(xhr) { ... }`
リクエストがエラーとなった場合のコールバック関数です。エラーメッセージをコンソールに出力します。

このJavaScriptコードにより、数量の変更が行われるとAjaxリクエストが発生し、
サーバーからのレスポンスに基づいてカートアイテムの小計が動的に更新されます!

### jsファイル
```js:update.js.erb
$("#cart_item_<%= @cart_item.id %> .subtotal-area").html("<%= j(render 'subtotal', cart_item: @cart_item) %>")
$("#total-area").html("<%= j(render 'total', nedan: @nedan) %>")
```
これはjQueryセレクタで、特定のカートアイテムの .subtotal-area というクラスを持つ要素を指定しています。
.html("<%= j(render 'subtotal', cart_item: @cart_item) %>"): この部分では、指定された要素のHTMLを新しいHTMLに置き換えています。
render 'subtotal', cart_item: @cart_item は、_subtotal.html.erbを呼び出して、
@cart_itemというローカル変数を渡してその結果をHTMLエスケープして返します。
<%= j(...)%>はJavaScriptのエスケープメソッドで、JavaScriptコード内でRailsの出力を正しく扱うために使用されています。

このコード全体の目的は、Ajaxリクエストの応答として、特定のカートアイテムの .subtotal-area 要素を新しいHTMLに更新することです。

### コントローラ
```html:cart_items.controller.erb
def update
 @cart_item = CartItem.find(params[:id])
 @cart_item.update(cart_items_params)
 @cart_items = current_customer.cart_items.all
 @nedan = @cart_items.inject(0) { |sum, item| sum + item.subtotal }
# redirect_to cart_items_path, notice: "数量を変更しました。"
end
```

### 部分テンプレート
```html:_subtotal.html.erb
<%= cart_item.subtotal.to_s(:delimited) %>円
```

```html:_total.html.erb
<%= nedan.to_s(:delimited) %>円
```

## 終わりに
　非同期を色々な場面で試していこうと思いました。



