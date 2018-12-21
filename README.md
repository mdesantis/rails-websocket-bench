# Rails Websocket Bench

```bash
bundle install
yarn install
RAILS_ENV=production bundle exec rails db:create
RAILS_ENV=production bundle exec rails db:migrate
RAILS_ENV=production bundle exec rails assets:precompile
curl -Lo bin/anycable-go https://github.com/anycable/anycable-go/releases/download/v0.5.4/anycable-go-0.5.4-linux-amd64 && \
  chmod +x bin/anycable-go
```

## Start the bench application

[`Foreman`](https://github.com/ddollar/foreman) is used in order to ease processes management with Procfiles.

### Start with ActionCable as Websocket server

```bash
foreman start -e .env.production,.env.actioncable -f Procfile.actioncable
```

### Start with AnyCable as Websocket server

```bash
foreman start -e .env.production,.env.anycable -f Procfile.anycable
```

## Start the test suite

```bash
tsung -f tsung.xml -k start
```
