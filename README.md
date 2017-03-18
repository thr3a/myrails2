# オレオレRails

Nginxとpuma利用を想定したRailsテンプレート

- ruby 2.3.1
- Rails 5.0.2
- MySQL
- Capistrano
- Puma

# インストール

```
git clone https://github.com/thr3a/myrails.git
cd myrails
bundle install
```

# 初期設定

**config/application.rb** にてサービス名の設定

```ruby
module Myrails
  class Application < Rails::Application
    config.title = "My rails"
  end
end
```

**config/initializers/session_store.rb** にてセッションのプレフィックスを変更しておくと吉

```ruby
Rails.application.config.session_store :cookie_store, key: '_myrails_session'
```

# デプロイ

**config/database.yml** と **config/secrets.yml** と **config/deploy/production.rb** を作成する

**confing/deploy.rb** にて各自設定して、いざデプロイ

```
bundle exec cap production deploy:mkdir
bundle exec cap production deploy:upload
bundle exec cap production deploy
```

# その他

### pumaの再起動

```
bundle exec cap production puma:stop
bundle exec cap production puma:start
```

### モデルのバリデーションメッセージ出したい

```ruby
flash[:danger] = @post.errors.full_messages
```

### nginxの設定例

myrailsのところは適宜変更すること

```
upstream puma {
  server unix:/var/www/myrails/shared/tmp/sockets/puma.sock;
}

server {
  listen 80 default_server;
  access_log off;
  error_log /var/log/nginx/error.log;
  root /var/www/myrails/current/public;
  client_max_body_size 0;
  error_page 404 /404.html;
  error_page 500 502 503 504 /500.html;

  location / {
    try_files $uri @proxy;
  }
  location @proxy {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_pass http://puma;
  }
}
```
