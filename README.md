# 目的
今回はフレームワークを駆使したMVCとRESTの仕組み理解がメイン
- どのようにページが表示されるのか?
- ボタンクリック後、どうサーバーと通信されプロセスで結果が表示されるのか？

---

# 項目
- [事前準備](https://github.com/mikuga/studyWebMVC/blob/master/README.md#%E4%BA%8B%E5%89%8D%E6%BA%96%E5%82%99)
- [予備知識](https://github.com/mikuga/studyWebMVC/blob/master/README.md#%E4%BA%88%E5%82%99%E7%9F%A5%E8%AD%98)
  - [リクエストとは](https://github.com/mikuga/studyWebMVC#%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%81%A8%E3%81%AF)
  - [CRUDとRESTとは](https://github.com/mikuga/studyWebMVC/blob/master/README.md#crud%E3%81%A8rest%E3%81%A8%E3%81%AF)
  - [MVCとは](https://github.com/mikuga/studyWebMVC/blob/master/README.md#mvc%E3%81%A8%E3%81%AF)
- [ハンズ オン](https://github.com/mikuga/studyWebMVC/blob/master/README.md#%E3%83%8F%E3%83%B3%E3%82%BA-%E3%82%AA%E3%83%B3)
  - [プロジェクト作成](https://github.com/mikuga/studyWebMVC/blob/master/README.md#%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E4%BD%9C%E6%88%90)
  - [Welcome ページ作成](https://github.com/mikuga/studyWebMVC/blob/master/README.md#welcome-%E3%83%9A%E3%83%BC%E3%82%B8%E4%BD%9C%E6%88%90)
  - [Scaffold 作成](https://github.com/mikuga/studyWebMVC/blob/master/README.md#scaffold-%E4%BD%9C%E6%88%90)
  - [API](https://github.com/mikuga/studyWebMVC/blob/master/README.md#api)
    - [Curl](https://github.com/mikuga/studyWebMVC/blob/master/README.md#curl)
    - [Post Man](https://github.com/mikuga/studyWebMVC/blob/master/README.md#post-man)
    - [Swagger](https://github.com/mikuga/studyWebMVC/blob/master/README.md#swagger)
   
---

# 事前準備

```
git clone https://github.com/kurosawa-kuro/VagrantDocker
```

[VagrantDocker](https://github.com/kurosawa-kuro/VagrantDocker)

---

# 予備知識

## リクエストとは
クライアントとWebサーバーは「HTTPリクエスト」と「HTTPレスポンス」でやり取りします。

![](https://user-images.githubusercontent.com/20763910/46721228-70486000-ccad-11e8-8abd-6d2ff17c651d.png)

1. クライアント側のPCでブラウザにURLを入力します。
1. クライアント側のPCが、Webサーバーに「HTTPリクエスト」を送信します。
1. Webサーバーが、「HTTPリクエスト」に対応する「HTTPレスポンス」をクライアント側のPCに送信します。
1. クライアント側のPCでWebページを表示します。

## 代表的なHTTPリクエスト メソッド
### Getメソッド
1. クエリ文字列はブラウザのURL欄に表示されます。
1. クエリ文字列は大量のデータを送るのには向いていません。

### Postメソッド
1. POSTは大きめのデータを送信できます。
1. 入力内容はボディにありGETのようにブラウザのURLには表示されません。
1. ただしボディは解析すれば確認できるのでセキュリティ的に安全というわけではありません。

## CRUDとRESTとは

| 処理 | CRUD操作 | HTTPメソッド |
|---|---|---|
| 登録 | CREATE | POST | 
| 取得 | READ | GET | 
| 更新 | UPDATE | PUT |
| 削除 | DELETE | DELETE | 

RESTではURIで表されたリソースに対して各HTTPメソッドで操作を行います。

```
登録 POST    /users/:id
取得 GET     /users/
更新 PUT     /users/:id
削除 DELETE  /users/:id
```

- aap/config/routes.rb
```
Rails.application.routes.draw do
  get '/users/', to: 'users#index'
  post '/users/', to: 'users#create'
end
```

- users_controller.rb
```
class UsersController < ApplicationController
  # GET /users
  def index
    @users = User.all
  end

  # POST /users
  def create
    @user = User.new(user_params)

    if @user.save
        format.html { redirect_to @user, notice: 'User was successfully created.' }
    else
        format.html { render :new }
    end
  end
end
```

## MVCとは
HTML画面やデータベース処理等の債務によってレイヤーを分離した構造の事

- Model: ビジネスロジックを記述する。
- View: Modelを表示する。
- Controller: Viewの入力を受け取って判断し、Modelを起動する。

1. コントローラが入力機器（マウスやキーボードなど）を監視する。
1. ユーザが入力機器に入力を与える。
1. コントローラがユーザのアクションに応じてモデルのメソッドを呼ぶ。その結果モデルのデータが書き換えられる場合もある。
1. モデルが変更された場合、自身が変更された旨をビューなどのオブザーバに対して通知する。
1. ビューはモデルから関連するデータを取得し、出力を更新する。典型的には画面に図形を描画する。

---

# ハンズ オン

## プロジェクト作成
DBはMySQLの設定になるようにオプション設定しています。

Dockerの仮想環境コンテナが作成され、その中にRailsの新規プロジェクトが作成されます。
root権限で作成される可能性があるので、所有者を通常ユーザーに変更します。
```
docker-compose run web rails new . --force --database=mysql
sudo chown -R $USER:$USER workspace/
```

- config/database.yml
```
  username: root
  password: password
  host: mysql
```
Dockerから参照できるようにhostの設定も行います。


### タイムゾーンの設定
- config/application.rb
```
    # タイムゾーンの設定
    config.time_zone = 'Tokyo'
    config.active_record.default_timezone = :local
```

### データベース作成 

```
docker-compose up -d mysql
docker-compose run web bundle exec rake db:create
```

### コンテナ起動

```
docker-compose up web mysql mysql-gui portainer cloud9
```

portainer cloud9は必須ではありません。

### Webページアクセス

http://localhost:3000


## Welcome ページ作成

```
docker-compose run web rails g controller welcome index
sudo chown -R $USER:$USER workspace/
```

### コンテナ起動

```
docker-compose up web mysql mysql-gui portainer cloud9
```
portainer cloud9は必須ではありません。

### Webページアクセス
- http://localhost:3000/welcome/index
- http://192.168.33.10:3000/welcome/index

### Welcome ページ 解説
![](https://user-images.githubusercontent.com/20763910/46450264-61544000-c7cb-11e8-8c05-3ddfc6ebcb38.png)

1. ブラウザから「/」というURLのリクエストをRailsサーバーに送信する。
1. 「/」リクエストは、Railsのルーティング機構 (ルーター) によってWelcomeコントローラ内のindexアクションに割り当てられる。
1. indexアクションが実行され、indexビューに渡す。
1. indexビューが起動し、ERB (Embedded RuBy: ビューのHTMLに埋め込まれているRubyコード) を実行して HTMLを生成 (レンダリング) する。
1. コントローラは、ビューで生成されたHTMLを受け取り、ブラウザに返す。



## Scaffold 作成

| 用途 | アクション | URL |
|:--|:--|:--|
| すべてのユーザーを一覧するページ | index | /users |
| id=1のユーザーを表示するページ | show | /users/1 |
| 新規ユーザーを作成するページ | new | /users/new |
| id=1のユーザーを編集するページ | edit | /users/1/edit |

## user テーブル

| カラム名 | 型 |
|:-----------:|:------------:|
|user|string| 
|content|text| 
|datetime|datetime| 

```
docker-compose run web rails g scaffold user user:string content:text datetime:datetime
sudo chown -R $USER:$USER workspace/
docker-compose run web rails db:migrate
docker-compose up web mysql mysql-gui portainer cloud9
```

### Webページアクセス
- http://localhost:3000/users
- http://192.168.33.10:3000/users

<img width="949" alt="MVC_-_Google_スプレッドシート.png" src="https://qiita-image-store.s3.amazonaws.com/0/80065/ef1c7505-99d6-8930-f8b8-2269181dc70b.png">

1. ブラウザから「/users」というURLのリクエストをRailsサーバーに送信する。
1. 「/users」リクエストは、Railsのルーティング機構 (ルーター) によってUsersコントローラ内のindexアクションに割り当てられる。
1. indexアクションが実行され、そこからUserモデルに、「すべてのユーザーを取り出せ」(User.all)と問い合わせる。
1. Userモデルは問い合わせを受け、すべてのユーザーをデータベースから取り出す。
1. データベースから取り出したユーザーの一覧をUserモデルからコントローラに返す。
1. Usersコントローラは、ユーザーの一覧を@users変数 (@はRubyのインスタンス変数を表す) に保存し、indexビューに渡す。
1. indexビューが起動し、ERB (Embedded RuBy: ビューのHTMLに埋め込まれているRubyコード) を実行して HTMLを生成 (レンダリング) する。
1. コントローラは、ビューで生成されたHTMLを受け取り、ブラウザに返す。


| No. | Prefix | Verb | URI Pattern | Controller#Action |
|--:|:--|:--|:--|:--|
| 1 | users | GET | /users(.:format) | users#index |
| 2 | users | POST | /users(.:format) | users#create |
| 3 | new_user | GET | /users/new(.:format) | users#new |
| 4 | edit_user | GET | /users/:id/edit(.:format) | users#edit |
| 5 | user | GET | /users/:id(.:format) | users#show |
| 6 | user | PATCH | /users/:id(.:format) | users#update |
| 7 | user | PUT | /users/:id(.:format) | users#update |
| 8 | user | DELETE | /users/:id(.:format) | users#destroy |

- aap/config/routes.rb
```
Rails.application.routes.draw do
  root 'users#index'

  get '/users/', to: 'users#index'
  post '/users/', to: 'users#create'
  get '/users/new/', to: 'users#new', as: 'new_user'
  get '/users/:id/edit', to: 'users#edit', as: 'edit_user'
  get '/users/:id', to: 'users#show', as: 'user'
  patch '/users/:id', to: 'users#update'
  delete '/users/:id', to: 'users#destroy'

  resources :users

  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

docker-compose run web bundle exec rake routes

```
                   Prefix Verb   URI Pattern                                                                              Controller#Action
                     root GET    /                                                                                        users#index
                    users GET    /users(.:format)                                                                         users#index
                          POST   /users(.:format)                                                                         users#create
                 new_user GET    /users/new(.:format)                                                                     users#new
                edit_user GET    /users/:id/edit(.:format)                                                                users#edit
                     user GET    /users/:id(.:format)                                                                     users#show
                          PATCH  /users/:id(.:format)                                                                     users#update
                          DELETE /users/:id(.:format)                                                                     users#destroy
```



# 重要なフォルダ・ファイル

```
app/controllers/
app/models/
app/views/
config/application.rb
config/routes.rb
```

## Rails自体

bin

```
kurosawa:~/study/studyWebMVC/bin $ pwd
/Users/kurosawa/study/studyWebMVC/bin
kurosawa:~/study/studyWebMVC/bin $ tree
./
├── bundle
├── rails
├── rake
├── setup
├── spring
├── update
└── yarn
```

# API

```
docker-compose run web rails new . --force --database=mysql --api
sudo chown -R $USER:$USER workspace/
```

```
docker-compose run rails g scaffold Blog title:string body:text
sudo chown -R $USER:$USER workspace/
docker-compose run web rails g scaffold Blog title:string body:text
sudo chown -R $USER:$USER workspace/
docker-compose run web rails db:migrate
docker-compose up web mysql mysql-gui portainer cloud9
```

http://0.0.0.0:3000/users.json

```
curl -X POST -H "Content-Type: application/json" -d '{"blog":{"title": "RailsとSwiftでAPI通信を行うブログアプリを作成","body": "何書こうかな"}}' http://localhost:3000/blogs
```

## Curl

## Post Man

## Swagger

# Vagrant Docker
OSに左右されずDcokerを使う方法
権限がRootになってしまうので、都度Chownする必要がある
