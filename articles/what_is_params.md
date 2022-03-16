---
title: "Rails paramsって何か."
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ruby", "rails"]
published: false
---
# paramsとは...
paramsはrailsで送られてきた値を受け取るメソッド.

:::message
主な送られる情報は、GETのクエリパラメータとPOSTでformを使用して送信されるデータ
:::

```ruby
params[:カラム名]
```
上記のように受け取ることができる.

# 検証用アプリケーションの作成
まず準備として簡単なアプリケーションをscaffoldを使用して作成し、`rails db:migrate`も一緒にしていきます。

```ruby
$ rails g scaffold Post title:string body:text
$ rails db:migrate
```

以下のようなアプリケーションが作成されます。

```ruby
ActiveRecord::Schema.define(version: 2022_03_09_061023) do
  create_table "posts", force: :cascade do |t|
    t.string "title"
    t.text "body"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end
end
```

`$ rails c`を使用して、データを直接作成していきます。

```ruby
Post.create(title: 'テスト1', body: '内容1')
Post.create(title: 'テスト2', body: '内容2')
Post.create(title: 'テスト3', body: '内容3')
Post.create(title: 'テスト4', body: '内容4')
```

# link_toの受け渡し

まずは、index.html.erbの
```ruby
posts/index.html.erb
<tbody>
  <% @posts.each do |post| %>
    <tr>
      <td><%= post.title %></td>
      <td><%= post.body %></td>
      <td><%= link_to 'Show', post %></td>

      # ここのEditの edit_post_path(post) の部分に注目します
      <td><%= link_to 'Edit', edit_post_path(post) %></td>
      <td><%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
  <% end %>
</tbody>
```

上記の`@posts`には、以下のようにコントローラでPostのデータをすべて渡してます。
```ruby
"posts_controller"
def index
  @posts = Post.all
end
```

以下の`Edit`のlink_toについて説明します。
![](/images/posts-index.png)

つまりここでは各postのidが入っていることになります。
```ruby
link_to <%= link_to 'Edit', edit_post_path("eachで渡されたpostのidが入る") %>
```


ここでpostコントローラのeditアクションへのルーティングを確認します。
```
edit_post GET /posts/:id/edit(.:format) posts#edit
```
上記のように:idを含む形にURLになっています。

なので`Edit`をクリックしたときのURLは、以下のようになってます。
![](/images/posts-index.png)

```
http://localhost:3000/posts/1/edit
```

クリック後にeditページを表示するときにpostsコントローラのeditアクションを経由してedit.html.erbを表示しますが、そのときに`params[:id]`でURLから受け取ってることになります。
```ruby
def edit
  @post = Post.find(params[:id])
end
```

# formでよるパラメータの受け渡し

railsでformを使用する場合、form_withを使用するのが一般的です。
posts/new.html.erbファイルは以下のようになっています。
```ruby
<%= form_with(model: @post, local: true) do |form| %>
  <div class="field">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>
  <div class="field">
    <%= form.label :body %>
    <%= form.text_area :body %>
  </div>
  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```
ここでルーティングを確認しておきます。

```
posts POST /posts(.:format) posts#create
new_post GET /posts/new(:format) posts#new
```
form_withで送信をクリックした時、postsコントローラのcreateアクションにPOSTメソッドでデータが送信されます。
では、それがpostsコントローラのcreateアクションではどのように受け取ってるか。ここでは見ていきます。

```ruby
def create
  @post = Post.new(post_params)

  if @post.save
    redirect_to post_url(@post), notice: "Post was successfully created."
  else
    render :new
  end
end
```
実際にscaffoldで記述されたcreateアクションを見てみると上記のように記述されています。
`@post = Post.new(post_params)`の`post_params`ってなに??ってなります...
さらにここを見ていきます。
ざっくり説明するとストロングパラメータという仕組みを使用しています。

:::message
ストロングパラメータとは、パラメータを安全に受け取るための仕組み
:::

この`posts_params`は、コントローラの下の法に記述されてます。
```ruby
def post_params
  params.require(:post).permit(:title, :body)
end
```
この記述がストロングパラメータで受け取る仕組みになってます。
この中の記述を見ていきます。
まず、`require`は直訳すると"必要とする" `permit`は"許可"という意味になります。

:::message
つまりrequireの部分では、paramsに:postがあるはず。なければ例外にして。そしてpermitの部分では、params[:post]から取り出していい情報は、:titleと:bodyだけ取り出して。他の情報を無視して。という意味になります。
:::

このストロングパラメータという仕組みを使ってcreateアクションで受け取ってるということになります。

### なぜ直接、値を渡していけないか...
以下のようにデータを取得することもできます。
```ruby
@post = Post.new(:post)
# これだと以下のようにデータを取り出すことができる。
"rails c"
irb(main):001:0> post = Post.new(title: 'a', body: 'ka')
# Post.new(title: 'a', body: 'ka')とPost.new(:post)は同じ意味です。
   (3.2ms)  SELECT sqlite_version(*)
=> #<Post id: nil, title: "a", body: "ka", created_at: nil, updated_at: nil>
irb(main):002:0> post.title
=> "a"
irb(main):003:0> post.body
=> "ka"
```
直接データが入れられるし取り出せるし、とても便利に見えます。
上記のような機能のことをマスアサイン機能と言います。
ですが、これを使用すると...
悪意あるユーザーがいる場合、よきせぬ危険があります。

:::message alert
例えば、Userというadminユーザーか判断するカラムとして`admin: false`みたいなカラムがあるとします。
これが`true`のときはadmin権限を得て、`false`のときは操作ができない。
デフォルトでは、falseになってる。ですがストロングパラメータがない状態だと、データを送信するときにadminのフラグをtrueにする。みたいなパラメータを送ることも可能です。
これでは一般ユーザーが全ての権限を得る。そんなこともあり得ることになってしまいます。
:::

上記の理由からストラングラパメータを使用してデータの受け渡しをします。
