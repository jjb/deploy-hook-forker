# Deploy hook forker

Many hosting services, like Heroku, only allow you to have one HTTP deploy hook. We needed more so we wrote this app. This app accepts a post to /:app and forwards the post and its params to all the urls listed for that app in the config file.

# Usage

### Clone this app

```sh
git clone https://github.com/deadlyicon/deploy-hook-forker.git
cd deploy-hook-forker
```

### Modify the config file

```yaml
your-heroku-app-name:
  slack: https://other.slack.com/services/hooks/heroku?token=<%= ENV["SLACK_TOKEN"] %>
  appsignal: https://push.appsignal.com/1/markers/heroku?api_key=<%= ENV["APP_SIGNAL_API_KEY"] %>
  honeybadger: https://api.honeybadger.io/v1/deploys?deploy[environment]=production&api_key=<%= ENV["HONEYBADGER_API_KEY"] %>
another-heroku-app-name:
  honeybadger: https://api.honeybadger.io/v1/deploys?deploy[environment]=staging&api_key=<%= ENV["HONEYBADGER_API_KEY"] %>
```

### Push it up to heroku

```sh
heroku apps:create ${COMPANY_NAME}-deploy-hook-forker
```

### Point your app's HTTP deploy hook to the new deploy hook forker app

```sh
heroku addons:create deployhooks:http --url=https://${COMPANY_NAME}-deploy-hook-forker.herokuapp.com
```

### Securing with a secret

As deployed, anyone on the internet can POST to your deploy-hook-forker instance
and trick you into thinking your project has been deployed.

To avoid this, you can optionally require a secret be present when your apps POST to your
deploy-hook-forker instance. To do so, add a secret to your app:

```
➔ heroku config:set secret=`hexdump /dev/random | head | md5` --app ${COMPANY_NAME}-deploy-hook-forker
Setting secret and restarting ⬢ company-name-deploy-hook-forker... done, v7
secret: e4b9c3c27ad5a3b6f0c9b0291eeccc28
```

Now, when pointing an app's deploy hook to your deploy hook forker, use the secret in the url:

```sh
heroku addons:create deployhooks:http --url=https://${COMPANY_NAME}-deploy-hook-forker.herokuapp.com?secret=e4b9c3c27ad5a3b6f0c9b0291eeccc28
```


## Development

```ruby
bundle
bundle exec rackup

# in another terminal...
curl http://localhost:9292
curl -d ... http://localhost:9292
```

## References

[Heroku deploy hook docs](https://devcenter.heroku.com/articles/deploy-hooks)
