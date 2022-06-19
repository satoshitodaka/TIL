# Docker
## Dockerについて

## 実装
### 必要なファイルを作成
```
touch {dockerfile,docker-compose.yml,endpoint.sh}
```

#### Dockerfile
- コンテナの作成に使われる命令を記したもの。
```
FROM ruby:3.1.1
ARG ROOT="/i_am_here"
ENV LANG=C.UTF-8
ENV TZ=Asia/Tokyo

WORKDIR ${ROOT}

RUN apt-get update; \
  apt-get install -y --no-install-recommends \
                mariadb-client tzdata

COPY Gemfile ${ROOT}
COPY Gemfile.lock ${ROOT}
RUN gem install bundler
RUN bundle install --jobs 4

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT [ "entrypoint.sh" ]
EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```

#### docker-compose.yml
```yml
version: "3.9"
services:
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/i_am_here:cached
      - bundle:/usr/local/bundle
    ports:
      - 3000:3000
    depends_on:
      - db
    stdin_open: true
    tty: true
  db:
    image: mysql:5.7.33
    environment:
      MYSQL_ROOT_PASSWORD: "mysql"
      ports:
        - 3306:3306
      volumes:
        - mysql_data:/var/lib/mysql

volumes:
  bundle:
  mysql_data:
```
#### entrypoint.sh
```sh
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /i_am_here/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```
## 参考
