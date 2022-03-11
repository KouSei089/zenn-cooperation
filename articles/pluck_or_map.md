---
title: "mapメソッドとpluckメソッドって結局どっちを使うか?"
emoji: "🎓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby", "rails"]
published: false
---

# pluckとmapの違い
### pluckメソッド
指定したカラムのみをSQLで取得する。

```ruby
Post.pluck(:title)
SELECT `post`.`title` FROM `post`
=> ["test_1", "test_2", "test_3", "test_4"]
```

### mapメソッド
すべてのデータ取得後、そのデータから特定のカラムのデータを取得します。
mapはレシーバをメモリに読み込むのでメモリを浪費してしまう。
```ruby
Post.all.map(&:title)
SELECT `post`. FROM `post`
=> ["test_1", "test_2", "test_3", "test_4"]
```

### どのように使い分けるか

pluckメソッドは、特定のカラムのデータしか使用しないとき

:::message alert
インスタンス化されたオブジェクトにも毎回SQLを実行してしまう
:::

mapメソッドは、インスタンス化されたオブジェクトからデータを取得するとき

:::message alert
特定のカラムだけしか必要ないのに、余分にすべてのデータを読み込んでしまい、メモリの無駄使いになる。
:::
