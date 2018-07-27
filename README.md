# Rails Websocket Bench

```bash
bundle install
yarn install
RAILS_ENV=production bundle exec rails db:create
RAILS_ENV=production bundle exec rails db:migrate
RAILS_ENV=production bundle exec rails assets:precompile
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
