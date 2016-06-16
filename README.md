# Quovo

Quovo RESTful API ruby client.

## Installation

```ruby
# Gemfile
gem 'quovo'

# run `bundle` in shell
```

## Configuration

```ruby
  Quovo.configurate do |config|
    config.username               = 'username'
    config.password               = 'password'
    config.request_timeout        = 30.seconds # by default 60 seconds
    config.token_ttl              = 2.hour     # by default 1 hour
    config.token_prefix           = 'APP-NAME' # add custom prefix for token (helps to manage token list)
    config.debug                  = true # if you want to see detailed logs
    config.strip_sensitive_params = true # show [FILTERED] in logs for sensitive data 
  end
```
If you use Rails put this configuration into `config/initializers/quovo.rb`.

**Note:** By default client uses in-memory storage for access tokens. If you want to use persistent storage (redis, file)
you need to set storage object in configuration:

```ruby
  Quovo.configurate do |config|
    # ...
    config.token_storage = Object.new.tap do |o|
      def o.read(key)
        # read value by key from storage
      end

      def o.write(key, value)
        # write value by key to storage
      end
    end
  end
```

You can also use `Rails.cache` as storage:

```ruby
  Quovo.configurate do |config|
    # ...
    config.token_storage = Rails.cache
  end
```
## Scopes and Hooks

You can scope quovo actions with user-defined hash that
could be used in hooks. Useful for action logging.

```ruby
  Quovo.scope(user: user) do
    Quovo.accounts.all
  end
```

Hook is a registered callback that invokes when web request happens. 

```ruby
  Quovo.hook do |path, method, params, status_code, response, elapsed_time, scope|
    # path, method, params, status_code, response - attributes of web request
    # elapsed_time - time in seconds of web request 
    # scope - user-defined hash, see docs about scopes

    # log quovo action in database or file
  end
```

## Quovo Api bindings

### Brokerages
```ruby
  # Provides information on Quovo's supported brokerages
  client.brokerages.all
  # Information about single brokerage
  client.brokerages.find(brokerage_id)
```

### Users
```ruby
  client.users.all
  client.users.find(user_id)
  # Creates new user
  # additional parameters
  # name:   - client's name
  # email:  - client's email. Cannot match another user's email
  # phone:  - client's phone number  
  client.users.create(username)
  # Updates user information
  # additional parameters
  # name:   - client's name
  # email:  - client's email. Cannot match another user's email
  # phone:  - client's phone number  
  client.users.update(user_id)
  client.users.delete(user_id) 
```

### Accounts
```ruby
  client.accounts.all
  client.accounts.find(id)
  client.accounts.create(user_id, brokerage_id, username, password)
  client.accounts.update(account_id, brokerage_id, username, password)
  client.accounts.delete(account_id)
  client.accounts.for_user(user_id)
  # Init new sync
  client.accounts.sync!(account_id)
  # Get sync status
  client.accounts.sync(account_id)
```

### Challenges
```ruby
  client.challenges.for_account(account_id)
  # for text, image questions
  client.challenges.answers!(account_id, [{question: 'question text', answer: 'answer text'}])
  # for choice questions
  client.challenges.answers!(account_id, [{question: 'question text', answer: 0}])
```

### History
```ruby
  client.history.all
  # additional parameters
  # start: - pointer to next set of items
  # count: - max number of results to return
  # start_date: - filters out history before this date
  # end_date:   - filters out history after this date
  # start_id:   - filters out history before this id
  # end_id:     - filters out history after this id  
  client.history.for_user(user_id)
  client.history.for_account(account_id)
  client.history.for_portfolio(portfolio_id)
```

### Portfolios
```ruby
  client.portfolios.all
  client.portfolios.find(portfolio_id)
  client.portfolios.update(portfolio_id, nickname, portfolio_type, is_inactive)
  client.portfolios.for_user(user_id)
  client.portfolios.for_account(account_id)
```

### Positions
```ruby
  # Load all positions
  # additional parameters
  # start:   - pointer to next set of items
  # count:   - max number of results to return  
  client.positions.all
  client.positions.for_user(user_id)
  client.positions.for_account(account_id)
  client.positions.for_portfolio(portfolio_id)
```

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
