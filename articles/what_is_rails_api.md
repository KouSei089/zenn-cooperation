---
title: "RailsでWebAPIのデータをコンソールに出力するまで"
emoji: "📣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby", "rails"]
published: false
---
# そもそもAPIってなにか
関数みたいなもの。
> APIは「Application Programming Interface」の略です。アプリケーションプログラミングインターフェイス。

> 例えば「AというファイルをBという名前でコピーをして、作業完了したら、ポップアップウィンドウを出して知らせる！」というプログラムを作るとします。実際にどんな動きをするのかパートに分けてみると……、
> 1. Aというファイルを選択
> 2. 実行ボタンを押すと3のステップへ
> 3. データをコピーする
> 4. コピーされたデータをBという名前を付け保存
> 5. ポップアップウィンドウを出して作業完了を告げる

> この1.〜5.の作業をすべて一から作成すると、かなり手間が掛かります（マウスの動きを計算して、ウィンドウのデザインを考えて……）。そこで登場するのがAPIです。
> 1. ファイルを選択するAPI
> 2. ボタンを押すとプログラムを動かすAPI
> 3. データをコピーするAPI
> 4. ファイルに名前を付けるAPI
> 5. ウィンドウを出してメッセージを出すAPI

> と、いろいろな機能があるAPIから、必要なAPIを探し出し組み合わせるだけで、プログラムができてしまうのです。つまりAPIは「特定の機能を持つプログラム部品」なのです。よく使われる命令をAPIにしてみんなで共有してしまえば、非常に効率的に作業ができますね。
![http___image itmedia co jp_ait_articles_0703_13_r85minapi_01](https://user-images.githubusercontent.com/77420123/159111601-56fa37af-fa9a-4c99-bbcf-b9c2910afad0.gif)
図1 APIを使えば、細かい作業や無駄を省いてプログラムができる

> 引用 : @IT

# じゃあWebAPIはなに？
> web apiとは、HTTP・HTTPS通信によってやり取りするAPIのことです。
web apiを使用すれば、自社のWebサイトやアプリケーションに別のサービスの機能を組み込み、より便利な仕組みを構築できます。

> 例えば、グルメ情報サイトの店舗ページなどにはお店の位置情報を示したGoogle Mapが載っていますが、あれはweb api（Google Map API）を利用したもの。web apiによって使い慣れているGoogle Mapをページ内に表示させることで、サイト利用者は迷うことなく店舗の位置を把握できるのです。
> 引用 : JBAT

なるほど、つまり、インターネット通信を使用したAPIのことをWebAPIというのか。

# 実際に動かしてみる。
```
http://zipcloud.ibsnet.co.jp/api/search?zipcode=3510011
```
ブラウザにアクセスするとAPIににアクセスシた結果が出力されます。
<img width="289" alt="スクリーンショット 2022-03-19 16 27 49" src="https://user-images.githubusercontent.com/77420123/159111975-8f52ef57-6611-4001-98fd-7ceeadeea8aa.png">

こんな感じです。

WebAPIは、JSON形式でデータの受け取りをすることが多いです。上記の画像もJSON形式です。

:::message
JSONとは...
JavaScript Object Notationの略でJSONです。
JSONの特徴は以下です。
- データとして文字列、数値、空など扱える。
- データを括弧で囲んで構造を表すので、データのファイルが小さい
- 人が読みにくい（タグなどがないので）
:::

それでは、rubyで実際に動かしてコンソールにデータを取得したいと思います。

### まずは、結論から
以下のようなコードでデータを取得してコンソールに住所、市、町を結合した住所を出力することができます。

```ruby
require 'net/http'
require 'uri'
require 'json'

uri = URI.parse('http://zipcloud.ibsnet.co.jp/api/search?zipcode=0510011')
json = Net::HTTP.get(uri)
result = JSON.parse(json, {symbolize_names: true})

prefecture = result[:results][0][:address1]
city = result[:results][0][:address2]
town = result[:results][0][:address3]
address = prefecture << city << town

puts address
```

上記のコードで以下のような実行結果が返ってきます。

```
$ ruby api.rb
=> 北海道室蘭市中央町
```
ここから1つずつ解説していこうと思います。

```ruby
require 'net/http'
require 'uri'
require 'json'
```
まずはじめの3行では、必要なrubyのライブラリを読み込んでいます。各ライブラリの詳細はのちほど説明します。

```ruby
uri = URI.parse('http://zipcloud.ibsnet.co.jp/api/search?zipcode=0510011')
```
次にこのコードについて説明します。
ここでは、rubyのuriライブラリを使用してURIをパースしています。

どゆこと..??
:::message
そもそもURIとは.
URIとは、情報やデータといったリソースを識別するための記述方法をURIという。
URLは、URIのうちリソースが存在する場所を示すもの。
つまり、URIはURLの抽象概念といったところです。
:::

なぜパースする必要なあるのか
uriは文字列のように簡単に扱えるように見えるが、実は規約がかなら複雑で正確に処理するのがかなり大変らしい。

> uriは国際標準化団体のW3Cにより標準化された規格です。ただし次の複数の標準規格を満たす仕様のため、正確に処理しようとするとかなり複雑です。
[RFC1738] Uniform Resource Locators (URL) (Updated by [RFC2396])
[RFC2255] The LDAP URL Format (Obsoleted by [RFC4510], [RFC4516])
[RFC2368] The mailto URL scheme
[RFC2373] IP Version 6 Addressing Architecture (Obsoleted by [RFC3513])
[RFC2396] Uniform Resource Identifiers (URI): Generic Syntax (Obsoleted by [RFC3986])
[RFC2732] Format for Literal IPv6 Addresses in URL’s (Obsoleted by [RFC3986])
[RFC3986] Uniform Resource Identifier (URI): Generic Syntax
[RFC3987] Internationalized Resource Identifiers (IRIs)

うーん。パッと見わかんないですね..笑
なので、rubyでuriを処理するときは、uriライブラリを使えばそんなこと考えなくても使えるよ。とそういうことらしいです。

```ruby
uri = URI.parse('http://zipcloud.ibsnet.co.jp/api/search?zipcode=0510011')
puts uri

# $ ruby api.rb
# => http://zipcloud.ibsnet.co.jp/api/search?zipcode=0510011
```
実際、さっきのコードの一部を使用してパースしたあとのデータを見てみると何も変わってないように見えますが、中身は使用しやすい形に変わってます。

次の行を見ていきます。

```ruby
json = Net::HTTP.get(uri)
```
この行ですね、ここでは、net/httpライブラリというrubyのライブラリを使用して、サーバからJSON形式の結果をもらってます。
また、ここではHTTP通信でAPIにアクセスしています。

net/httpライブラリとは、クライアントとサーバー間で情報をやり取りするプロトコルであるHTTPを扱うライブラリです。
ここでは、言葉で説明するよりも実際の実行結果を見たほうがわかりやすいと思います。

```ruby
json = Net::HTTP.get(uri)
puts json

# $ ruby api.rb
=begin
{
    "message": null,
    "results": [
        {
            "address1": "北海道",
            "address2": "室蘭市",
            "address3": "中央町",
            "kana1": "ﾎｯｶｲﾄﾞｳ",
            "kana2": "ﾑﾛﾗﾝｼ",
            "kana3": "ﾁｭｳｵｳﾁｮｳ",
            "prefcode": "1",
            "zipcode": "0510011"
        }
    ],
    "status": 200
}
=end
```
上記のようにはじめにブラウザでみたJSONデータをrubyのファイルで出力することができました。

:::message alert
※=begin =end は複数行のコメントアウトで使用しているだけなので実際の出力結果には関係ありません。
:::

次に進みます。

```ruby
result = JSON.parse(json, {symbolize_names: true})
```
ここでは、rubyのjsonライブラリを使用してJSON文字列をハッシュに変換しています。

```ruby
result = JSON.parse(json, {symbolize_names: true})
puts result

# $ ruby api.rb
# => {:message=>nil, :results=>[{:address1=>"北海道室蘭市中央町", :address2=>"室蘭市", :address3=>"中央町", :kana1=>"ﾎｯｶｲﾄﾞｳ", :kana2=>"ﾑﾛﾗﾝｼ", :kana3=>"ﾁｭｳｵｳﾁｮｳ", :prefcode=>"1", :zipcode=>"0510011"}], :status=>200}
```
このようにハッシュの形にしています。今回の記述では、第2引数に`symbolize_names: true`を指定することでシンボルのキーをシンボルのままにしています。

"シンボルのキーをシンボルのままにしている。"どういうこと？

実は、先程の記述で引数を付けないとハッシュのキーがString型になります。
String型でもデータは出力できるのですが、String型よりもシンボルの方が実行スピードが早い。また、String型の場合、データを細かくとる際、直感的に分かりづらいと思い、シンボルにしてます。
少しわかりにくいと思うので、実際のコードを見てみます。

```ruby
result = JSON.parse(json, {symbolize_names: true})
puts result[:results][0][:address2]

# $ ruby api.rb
# 室蘭市
```
こんな感じで市だけとることができます。
これが、String型だと以下のようになります。

```ruby
result = JSON.parse(json)
puts result["results"][0]["address2"]

# $ ruby api.rb
# 室蘭市
```
ダブルクオーテーションで囲ってあげて文字列にしてあげる必要があります。

さて、最後の部分です。
```ruby
prefecture = result[:results][0][:address1]
city = result[:results][0][:address2]
town = result[:results][0][:address3]
address = prefecture << city << town

puts address

# $ ruby api.rb
# 北海道室蘭市中央町
```
ここでは、`JSON.parse`を使用してJSONデータをハッシュにした値から必要なデータを取り出して、各県、市、町を結合して住所としています。

最後に改めて作成したコードと実行結果を見てみましょう。

```ruby
require 'net/http'
require 'uri'
require 'json'

uri = URI.parse('http://zipcloud.ibsnet.co.jp/api/search?zipcode=0510011')
json = Net::HTTP.get(uri)
result = JSON.parse(json, {symbolize_names: true})

prefecture = result[:results][0][:address1]
city = result[:results][0][:address2]
town = result[:results][0][:address3]
address = prefecture << city << town

puts address

# $ ruby api.rb
# 北海道室蘭市中央町
```

自分自身、API通信に関する知識が浅かったので、少し深堀りしてみました。

# 参考記事

https://qiita.com/NagaokaKenichi/items/df4c8455ab527aeacf02

https://qiita.com/ArakawaYukihiro/items/9247dad036ea39b1e5f6

https://style.potepan.com/articles/31408.html

