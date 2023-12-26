---
title: "注文商品の商品合計、請求額"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [markdown]
published: true
---
# やりたいこと
注文情報確認画面で、送料、商品合計額、請求額の表示




## 自身が苦労した点
下記の記述方法で記入しており、２つ以上の商品を購入した時その商品ごとに表が作成されてしまった。
```index.rb:orders/confirm
<table class="table">
      <% @cart_items.each do |cart_item| %>
      <tr>
        <th class="table-warning">送料</th>
        <td><%= @shipping_fee %>円</td>
      </tr>
      <tr>
        <th class="table-warning">商品合計</th>
        <td><%= cart_item.subtotal%>円</td>
       <% subtotal += cart_item.subtotal %>
      </tr>
      <tr>
        <th class="table-warning">請求額</th>
        <td><%= @subtotal +  @shipping_fee %>円</td>
      </tr>
    </table>
```

### 記述
そこで以下の記述を記載
```index.rb:orders/confirm
 <table class="table">

      <tr>
        <th class="table-warning">送料</th>
        <td><%= @shipping_fee %>円</td>
      </tr>
      <tr>
        <th class="table-warning">商品合計</th>
        <td><%= @cart_items.sum(&:subtotal) %>円</td>

      </tr>
      <tr>
        <th class="table-warning">請求額</th>
        <td><%= 800 + @cart_items.sum(&:subtotal) %>円</td>
      </tr>
    </table>

```
このように記述することでeachの繰り返し文を使用せず、合計金額の表示が可能になる。

#### 解説
`sum(&:subtotal)`は、`cart_items`の`subtotal`メソッドの値を合計するためのコードです。
このコードは`&:subtotal`というシンボルをブロックとして渡しています。これにより、各要素の`subtotal`メソッドを呼び出して得られる値を合計することができます。

##### 終わりに
ここで紹介したコードはあくまで合計金額を算出する手段の１つです。
他にもいい記述方法がありましたら、コメントで教えてもらえると幸いです。



