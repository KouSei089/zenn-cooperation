---
title: "Railsのschema_migrationsってなんぞや"
emoji: "<img src="https://user-images.githubusercontent.com/77420123/157225601-d7fdc37c-a29f-4f4e-94e6-271b30d46111.svg" width="10px">"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby", "rails"]
published: false
---

migrationの仕組みってなんとなく、日付順に実施してくれるくらいの認識だったので、今回は少し深堀ってみようと思います！

まずは、scaffoldで簡単なアプリ雛形を作成し、合わせてdb:createも実行します。
```ruby
$ rails g scaffold Post title:string body:text
$ rails db:create
```
`$ rails db` コマンドを使用して、テーブル一覧を確認します。
まだ、`$ rails db:migrate` していないのでなにも表示されないはずです。
```ruby
$ rails db
SQLite version 3.32.3 2020-06-18 14:16:19
Enter ".help" for usage hints.
sqlite> .tables
sqlite>
```

`$ rails db:migrate`実行前にマイグレーションファイルを確認してみます。
```ruby
class CreatePosts < ActiveRecord::Migration[6.0]
  def change
    create_table :posts do |t|
      t.string :title
      t.text :body

      t.timestamps
    end
  end
end
```
`$ rails db:migrate`を実行します。
```ruby
$ rails db:migrate
== 20220309061023 CreatePosts: migrating ======================================
-- create_table(:posts)
   -> 0.0036s
== 20220309061023 CreatePosts: migrated (0.0036s) =============================
```
ここでもう一度DBを確認すると...

`schema_migrations`を発見しました！！
```ruby
$ rails db
SQLite version 3.32.3 2020-06-18 14:16:19
Enter ".help" for usage hints.
sqlite> .tables
ar_internal_metadata  posts                 schema_migrations
```
`schema_migrations`をもう少し見てみると、先程のマイグレーションファイルの日付情報が入ってることがわかります。
```ruby
sqlite> .tables
ar_internal_metadata  posts                 schema_migrations
sqlite> select * from schema_migrations;
20220309061023
```

> The table schema_migrations contain one column with one entry that is the unique id of the file that was created. Coincidence ? I don’t think so, rails actually does book keeping of all the migrations that had already run by keeping an entry of the unique id in this table.
参考記事より引用させていただきました.

簡単な翻訳をかけると...

どうやら、railsは、このテーブルに一意のIDのエントリを保持することによって、すでに実行されたすべての移行を記録しているらしいです。

`$ rails db:rollback`で最後に実行された移行ファイル（最後の移行ファイル）をロールバックして、`schema_migrations`の中身を確認してみます。
```ruby
$ rails db:rollback
== 20220309061023 CreatePosts: reverting ======================================
-- drop_table(:posts)
   -> 0.0144s
== 20220309061023 CreatePosts: reverted (0.0225s) =============================
```
`schema_migrations`が殻になっていることがわかります。
```ruby
sqlite> .tables
ar_internal_metadata  schema_migrations
sqlite> select * from schema_migrations;
sqlite>
```
`$ rails db:migrate`をもう一度実行することで、`schema_migrations`に同じ値が追加されます。

## まとめ

:::message
`migrationファイル`は日付情報で判断して、 `schema_migrations`に値がないものを実行するようになっているっぽいです！
:::

## 参考記事

https://tushartuteja.medium.com/demystifying-rails-migrations-53abcf3a7ddd
