# Captain Hook

[![Build Status](https://travis-ci.org/mark-rushakoff/captain_hook.png?branch=master)](https://travis-ci.org/mark-rushakoff/captain_hook)
[![Code Climate](https://codeclimate.com/github/mark-rushakoff/captain_hook.png)](https://codeclimate.com/github/mark-rushakoff/captain_hook)
[![Coverage Status](https://coveralls.io/repos/mark-rushakoff/captain_hook/badge.png)](https://coveralls.io/r/mark-rushakoff/captain_hook)

Captain Hook simplifies setting up a web app to handle GitHub repo hooks.

## Installation

Add this line to your application's Gemfile:

    gem 'captain_hook'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install captain_hook

## Usage

### Your App

Create an application by subclassing `CaptainHook::App`:

```ruby
# app.rb
require 'captain_hook/app'

class MyApp < CaptainHook::App
  # You must supply a secret for each repository and hook.
  # The secret should only be known by GitHub; that's how we know the request is coming from GitHub's servers.
  # (You'll specify that secret when you go through subscription.)
  def secret(owner, repo_name, event_type)
    if owner == 'octocat' && repo_name == 'Spoon-Fork' && event_type == 'pull_request'
      'qwertyuiop'
    else
      raise 'unrecognized!'
    end
  end

  # Code to execute for GitHub's pull request hook.
  # The secret has already been verified if your code is being called.
  # See app.rb and spec/app/*_spec.rb for signatures and examples of the valid handlers.
  def on_pull_request(owner, repo_name, action, number, pull_request_json)
    message = <<-MSG
      GitHub user #{pull_request_json['user']['login']} has opened pull request ##{number}
      on repository #{owner}/#{repo_name}!
    MSG
    send_text_message(MY_PHONE_NUMBER, message) # or whatever you want
  end
end
```

You'll need to host your app online and hold on to the base URL so you can subscribe to hooks.

### GitHub Authentication

To create hooks on GitHub repositories, you need to be authenticated as a collaborator on that repository.
GitHub's UI currently only supports configuring push hooks, so you'll want to authenticate through Captain Hook to set up custom hooks.

Captain Hook never stores your password.
He always uses OAuth tokens.
The only time he asks for your password is when he is creating a new token or listing existing tokens.

Captain Hook doesn't store your tokens, either.
It's your duty to manage storage of tokens.

Here's one way you can generate and list tokens:

```ruby
require 'captain_hook/client'

puts "Please enter your username:"
username = $stdin.gets

# Prompt the user for their password, without displaying it in the terminal
client = CaptainHook::Client.interactive_build(username)

# Note for the token (this will be displayed in the user's settings on GitHub)
note = "Created in my awesome script"
client.create_authorization(note)

client.authorizations.each do |token|
  puts "Token: #{auth.token}\nNote: #{auth.note}\n\n"
end
```

### Subscribing to Hooks

Captain Hook provides a way to subscribe to hooks.
It's easy!

```ruby
require 'captain_hook/subscriber'

default_opts = {
  base_url: "http://captain-hook.example.com",
  oauth_token: ENV['CAPTAIN_HOOK_TOKEN']
}

subscriber = CaptainHook::Subscriber.new(default_opts)

subscriber.subscribe(
  owner: 'octocat',
  repo_name: 'Hello-World',
  event_type: 'pull_request',
  secret: 'secret_for_hello_world'
)
```

(For more details, consult the RDoc documentation.)

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
