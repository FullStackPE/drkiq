https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application

# OSX/Windows users will want to remove --­­user "$(id -­u):$(id -­g)"
docker­-compose run --­­user "$(id ­-u):$(id -­g)" drkiq rake db:reset
docker­-compose run --­­user "$(id ­-u):$(id -­g)" drkiq rake db:migrate
