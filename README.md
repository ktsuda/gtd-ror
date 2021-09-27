# gtd-ror

## Requirements

- docker v20.10.8
- docker-compose v1.29.2

## How to build the docker image

Make the `Dockerfile`.

```Dockerfile
FROM ruby:3.0.2
RUN wget --quiet -O - /tmp/pubkey.gpg https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo 'deb https://dl.yarnpkg.com/debian/ stable main' > /etc/apt/sources.list.d/yarn.list
RUN set -x && apt-get update -y -qq && apt-get install -yq nodejs yarn

RUN mkdir /app
WORKDIR /app
COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock
RUN bundle install
COPY . /app
```

Make the `Gemfile`.

```Gemfile
source 'https://rubygems.org'
gem 'rails', '6.0.3.4'
```

Make the empty `Gemfile.lock`.

```bash
touch Gemfile.lock
```

Make the `docker-compose.yml`.
Set the environment variable, `MYSQL_ROOT_PASSWORD` in it.

```yaml
version: '3'
services:
  app:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/app
    ports:
      - 3000:3000
    depends_on:
      - db
    tty: true
    stdin_open: true
  db:
    image: mysql@sha256:dce31fcdd15aaedb5591aa89f19ec37cb79981af46511781fa41287d88ed0abd
    volumes:
      - db-volume:/var/lib/mysql
      - ./mysql-confd:/etc/mysql/conf.d
    environment:
      MYSQL_ROOT_PASSWORD: password
volumes:
  db-volume:
```

Make a file, `mysql-confd/default_authentication.cnf`,
to specify the default authentication method for MySQL.

```conf
[db]
default_authentication_plugin = mysql_native_password
```
Build image and run the `rails new` command.

```bash
docker-compose run app rails new . --force --database=mysql
```

Edit `database.yml` and specify the password and host for the MySQL container.

```yaml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: password
  host: db
```

Build the container again to reflect the modification above.

```bash
docker-compose build
```

Launch containers in the background.

```bash
docker-compose up -d
```

Create the database.

```bash
docker-compose run app rails db:create
```

Finally, access `localhost:3000` with your web browser to check if you can see the default page.

## How to run the containers

Launch containers in the background.

```bash
docker-compose up -d
```

Access `localhost:3000` with your web browser to check if you can see the default page.
