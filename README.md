Griddler
========

Griddler is a Rails engine (full plugin) that provides an endpoint for the
[Sendgrid parse
api](http://sendgrid.com/docs/API%20Reference/Webhooks/parse.html) that hands
off a built email object to a class implemented by you.

Installation
------------

Add griddler to your application's Gemfile and run `bundle install`:

```ruby
gem 'griddler'
```

Griddler comes with a default endpoint that will be displayed at the bottom of
the output of `rake routes`. If there is a previously defined route that matches
`/email_processor`–or you would like to rename the matched path–you may
add the route to the desired position in routes.rb with the following:

```ruby
match '/email_processor' => 'griddler/emails#create', via: :post
```

Defaults
--------

By default Griddler will look for a class to be created in your application
called EmailProcessor with a class method implemented named process, taking in
one argument (presumably `email`). For example, in `./lib/email_processor.rb`:

```ruby
class EmailProcessor
  def self.process(email)
    # all of your application-specific code here - creating models, processing
    # reports, etc
  end
end
```

The contents of the `email` object passed into your process method is a hash
containing:

* `:to`
* `:from`
* `:subject`
* `:body`

Each of those has some sensible defaults.

`:from` and `:subject` will contain the obvious values found in the email, the
raw from and subject values.

`:body` will contain the full contents of the email body **unless** there is a
line in the email containing the string `-- Reply ABOVE THIS LINE --`. In that
case `:body` will contain everything before that line.

`:to` will contain all of the text before the email's "@" character. We've found
that this is the most often use portion of the email address and consider it to
be the token we'll key off of for interaction with our application.

Configuration Options
---------------------

An initializer can be created to control some of the options in Griddler.
Defaults are shown below with sample overrides following. In
`config/initializer/griddler.rb`:

```ruby
Griddler.configure do |config|
  config.handler_class = EmailProcessor # MyEmailProcessor
  config.handler_method = :process # :go
  config.raw_body = false # true
  config.to = :token # :raw, :email, :hash
  # :raw    => 'AppName <s13.6b2d13dc6a1d33db7644@mail.myapp.com>'
  # :email  => 's13.6b2d13dc6a1d33db7644@mail.myapp.com'
  # :token  => 's13.6b2d13dc6a1d33db7644'
  # :hash   => { raw: '', email: '', token: '', host: '' }
  config.reply_delimeter = '-- REPLY ABOVE THIS LINE --'
end
```

* `config.handler_class` change the class Griddler will use to handle your
  incoming emails.
* `config.handler_method` change the class method called on
  `config.handler_class`.
* `config.reply_delimeter` change the string searched for that will split your
  body.
* `config.raw_body` use the full email body whether or not
  `config.reply_delimeter` is set.
* `config.to` change the format of the returned value for the `:to` key in the
  email object. `:hash` will return all options within a (surprise!) - hash.

More Information
----------------

* [Sendgrid](http://www.sendgrid.com)
* [Sendgrid Parse API](www.sendgrid.com/docs/API Reference/Webhooks/parse.html)

Credits
-------

Griddler was written by Caleb Thompson and Joel Oliveira.

Large portions of the codebase were extracted from thoughtbot's
[Trajectory](http://www.apptrajectory.com).

![thoughtbot](http://thoughtbot.com/images/tm/logo.png)

The names and logos for thoughtbot are trademarks of thoughtbot, inc.

License
-------

Griddler is Copyright © 2012 Caleb Thompson, Joel Oliveira and thoughtbot. It is
free software, and may be redistributed under the terms specified in the LICENSE
file.