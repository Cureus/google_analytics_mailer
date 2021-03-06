# GoogleAnalyticsMailer 
[![Gem Version](https://badge.fury.io/rb/google_analytics_mailer.png)](http://badge.fury.io/rb/google_analytics_mailer) [![Build Status](https://secure.travis-ci.org/fabn/google_analytics_mailer.png)](http://travis-ci.org/fabn/google_analytics_mailer) [![Coverage Status](https://coveralls.io/repos/fabn/google_analytics_mailer/badge.png)](https://coveralls.io/r/fabn/google_analytics_mailer) [![Code Climate](https://codeclimate.com/github/fabn/google_analytics_mailer.png)](https://codeclimate.com/github/fabn/google_analytics_mailer) [![Dependency Status](https://gemnasium.com/fabn/google_analytics_mailer.png)](https://gemnasium.com/fabn/google_analytics_mailer)

This gem automatically rewrites **absolute** links generated by ActionMailer. It intercepts all'`url_for` calls (so `link_to` calls are intercepted as well)
and change the final url to add [Custom Campaign parameters](http://support.google.com/analytics/bin/answer.py?hl=en&answer=1033867) to your URLs.

## Installation

Add this line to your application's Gemfile:

    gem 'google_analytics_mailer'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install google_analytics_mailer

## Usage

There are three level of customization for rewritten links:

 * **Class level:** Params specified at class level will appear on every link of every message generated by that class
 * **Method level:** Params specified at method level will appear only on that message
 * **View level:** Views level parameters will take the highest precedence and they will always override all the others

In order to enable Google Analytics params for a given mailer you should simply add a line to a given mailer as in:

```ruby
class UserMailer < ActionMailer::Base
  default :from => 'no-reply@example.com'

  # declare url parameters for this mailer
  google_analytics_mailer utm_source: 'newsletter', utm_medium: 'email' # etc

  # Links in this email will have all links with GA params automatically inserted
  def welcome
    mail(to: 'user@example.com')
  end
end
```

Then in your view generate links as usual:

```erb
<!-- this will produce your-url?utm_medium=email&utm_source=newsletter because of class default params -->
<%= link_to('Read online', newsletter_url) -%>
<!-- local parameters are not overridden, so this produces ?utm_medium=email&utm_source=my_newsletter -->
<%= link_to('Read online', newsletter_url(utm_source: 'my_newsletter')) -%>
```

In order to override params for a specific message you can override params in the method which defines
the message as in:

```ruby
class UserMailer < ActionMailer::Base
  default :from => 'no-reply@example.com'

  # declare url parameters for this mailer
  google_analytics_mailer utm_source: 'newsletter', utm_medium: 'email' # etc

  # Links in this email will have utm_source equal to second_newsletter
  def welcome
    google_analytics_params(utm_source: 'second_newsletter', utm_term: 'welcome2')
    mail(to: 'user@example.com')
  end
end
```

At view level you can override generated parameters using the `with_google_analytics_params` method

```erb
<div class="footer">
  <%= with_google_analytics_params(utm_term: 'footer') do -%>
    <!-- this will override other params and produces ?utm_medium=email&utm_source=newsletter&utm_term=footer -->
    <%= link_to('Read online', newsletter_url) -%>
  <%- end -%>
</div>
```

or you can disable them for a specific block

```erb
<div class="footer">
  <%= without_google_analytics_params do -%>
    <!-- this will output link with no analytics params at all -->
    <%= link_to('Read online', newsletter_url) -%>
  <%- end -%>
</div>
```

## Action Controller integration

Since ActionMailer and ActionController use the same code base to provide view rendering this gem can be used in the
 same way in any action controller which inherits from `ActionController::Base` (i.e. almost every controller in a
 Rails application). The alias `google_analytics_controller` is provided for better naming thus in a controller you can do

```ruby
class UserController < ApplicationController

  # declare url parameters for this controller. Absolute links in this controller will be tagged with GA parameters
  google_analytics_controller utm_source: 'some site', utm_medium: 'web' # etc

  # Override for single action
  def index
    google_analytics_params(utm_source: 'index page', utm_term: 'foo bar')
  end
end
```

View syntax is obviously the same as in mailer.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
