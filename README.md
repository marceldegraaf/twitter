The Twitter Ruby Gem
====================
A Ruby wrapper for the Twitter REST and Search APIs

What's New in 1.0
-----------------
This gem has been completely rewritten for version 1.0 thanks to [contributions from numerous
people](http://github.com/jnunemaker/twitter/blob/master/HISTORY.md). This rewrite breaks
compatibility with version 0.9.12 and earlier versions of the gem. Most notably, the
<tt>Twitter::Base</tt>, <tt>Twitter:Geo</tt>, <tt>Twitter::LocalTrends</tt>, and
<tt>Twitter::Trends</tt> classes [have all been merged](http://github.com/jnunemaker/twitter/commit/eb53872249634ee1f0179982b091a1a0fd9c0973) into
the <tt>Twitter::Client</tt> class. Whenever possible, we [display deprecation warnings and forward method
calls to the <tt>Twitter::Client</tt> class](http://github.com/jnunemaker/twitter/commit/192e5884f367750dbdca8471aa12385ed5b057ca).
In a handful of cases, method names were changed to resolve namespace conflicts.

* Pre-1.0
        Twitter::Base.new.user('sferik').name
* Post-1.0
        Twitter::Client.new.user('sferik').name

The <tt>Twitter::Search</tt> class has remained largely the same, however it no longer accepts a
query in its constructor. You can specify a query using the <tt>#containing</tt> method, which is
aliased to <tt>#q</tt>.

* Pre-1.0
        Twitter::Search.new('query').fetch.results.first.text
* Post-1.0
        Twitter::Search.new.q('query').fetch.results.first.text

The <tt>Twitter::OAuth</tt> class [has been removed](http://github.com/jnunemaker/twitter/commit/d33b119cdfdaefb10db99e56d28dd69625816edf).
This class was just a wrapper to get access tokens via the [oauth
gem](http://github.com/oauth/oauth-ruby). Given that there are a variety of gems that do the same
thing ([twitter-auth](http://github.com/mbleigh/twitter-auth),
[omniauth](http://github.com/intridea/omniauth), and
[devise](http://github.com/plataformatec/devise), to name a few) we decided to decouple
this functionality so you can use the authentication library of your choosing, or none at all. If
you would like to continue using the [oauth gem](http://github.com/oauth/oauth-ruby),
simply require it and make the following changes:

* Pre-1.0
        options = {:api_endpoint => 'https://api.twitter.com', :signing_endpoint => 'https://api.twitter.com', :sign_in => true}
        oauth_wrapper = Twitter::OAuth.new(YOUR_CONSUMER_TOKEN, YOUR_CONSUMER_SECRET, options)
        signing_consumer = oauth_wrapper.signing_consumer
        request_token = signing_consumer.get_request_token(:oauth_callback => CALLBACK_URL)
        redirect_to request_token.authorize_url
        # Use 'pin' instead of 'verifier' for PIN authentication
        oauth_token, oauth_token_secret = oauth_wrapper.authorize_from_request(request_token.token, request_token.secret, 'verifier')

* Post-1.0
        options = {:site => 'https://api.twitter.com', :request_endpoint => 'https://api.twitter.com', :authorize_path => '/oauth/authenticate'}
        signing_consumer = OAuth::Consumer.new(YOUR_CONSUMER_TOKEN, YOUR_CONSUMER_SECRET, options)
        request_token = signing_consumer.get_request_token(:oauth_callback => CALLBACK_URL)
        redirect_to request_token.authorize_url
        # Use 'pin' instead of 'verifier' for PIN authentication
        access_token = OAuth::RequestToken.new(signing_consumer, request_token.token, request_token.secret).get_access_token(:oauth_verifier => 'verifier')
        oauth_token, oauth_token_secret = access_token.token, access_token.secret

The public APIs defined in version 1.0 of this gem will maintain backwards compatibility until
the next major version, following the best practice of [Semantic Versioning](http://semver.org/).
You are free to continue using the 0.9 series of the gem, however it will not be maintained, so
upgrading to 1.0 is strongly recommended.

Here are a few more reasons to upgrade to 1.0:

* Ruby 1.9 compatibility: All code and specs now work in the latest version of Ruby
* Support for HTTP proxies: Access Twitter from from China, Iran, or inside your office firewall
* Support for multiple HTTP adapters: NetHttp (default), Typhoeus, Patron, or ActionDispatch
* Support for multiple request formats: JSON (default) or XML
* More flexible: Parse JSON or XML with the engine or your choosing via [MultiJSON](http://github.com/intridea/multi_json) and [MultiXML](http://github.com/sferik/multi_xml)
* More RESTful: Use HTTP DELETE (instead of POST) when calling destructive resources
* More methods: Request any documented resource in the Twitter API
* Send all requests over SSL: [Faster](http://gist.github.com/652330) and more secure
* Improved error handling: More easily retry after rate-limit errors or fail whales

For more information, please see the full documentation and examples of the gem's usage below.

Help! I'm getting: "Did not recognize your engine specification. Please specify either a symbol or a class. (RuntimeError)"
---------------------------------------------------------------------------------------------------------------------------

If you're using the JSON request format (i.e., the default), you'll need to
explicitly require a JSON library. We recommend [yajl-ruby](http://github.com/brianmario/yajl-ruby).
If you're using the XML request format, we recommend requiring [libxml-ruby](http://github.com/dvdplm/libxml-ruby) for dramatically improved performance over REXML.

Documentation
-------------
<http://rdoc.info/gems/twitter>

Usage Examples
--------------
    require 'rubygems'
    require 'twitter'

    # Get a user's location
    puts Twitter.user("sferik").location

    # Get a user's most recent status update
    puts Twitter.user_timeline("sferik").first.text

    # Get a status update by id
    puts Twitter.status(27558893223).text

    # Initialize a Twitter search
    search = Twitter::Search.new

    # Find the 3 most recent marriage proposals to @justinbieber
    search.containing('marry me').to('justinbieber').result_type('recent').per_page(3).each do |r|
      puts "#{r.from_user}: #{r.text}"
    end

    # Enough about Justin Bieber
    search.clear

    # Let's find a Japanese-language status update tagged #ruby
    puts search.hashtag('ruby').language('ja').no_retweets.per_page(1).fetch.results.first.text

    # And another
    puts search.fetch_next_page.results.first.text

    # Certain methods require authentication. To get your Twitter OAuth credentials,
    # register an app at http://dev.twitter.com/apps
    Twitter.configure do |config|
      config.consumer_key = YOUR_CONSUMER_KEY
      config.consumer_secret = YOUR_CONSUMER_SECRET
      config.oauth_token = YOUR_OAUTH_TOKEN
      config.oauth_token_secret = YOUR_OAUTH_TOKEN_SECRET
    end

    # Initialize your Twitter client
    client = Twitter::Client.new

    # Post a status update
    client.update("I just posted a status update via the Twitter Ruby Gem!")

    # Read the most recent status update in your home timeline
    puts client.home_timeline.first.text

    # Who's your most popular friend?
    puts client.friends.sort{|a, b| a.followers_count <=> b.followers_count}.reverse.first.name

    # Who's your most popular follower?
    puts client.followers.sort{|a, b| a.followers_count <=> b.followers_count}.reverse.first.name

    # Get your rate limit status
    puts client.rate_limit_status.remaining_hits.to_s + " Twitter API request(s) remaining this hour"

Submitting an Issue
-------------------
We use the [GitHub issue tracker](http://github.com/jnunemaker/twitter/issues) to track bugs and
features. Before submitting a bug report or feature request, check to make sure it hasn't already
been submitted. You can indicate support for an existing issuse by voting it up. When submitting a
bug report, please include a [Gist](http://gist.github.com/) that includes a stack trace and any
details that may be necessary to reproduce the bug, including your gem version, Ruby version, and
operating system. Ideally, a bug report should include a pull request with failing specs.

Submitting a Pull Request
-------------------------
1. Fork the project.
2. Create a topic branch.
3. Implement your feature or bug fix.
4. Add documentation for your feature or bug fix.
5. Add specs for your feature or bug fix.
6. Run <tt>bundle exec rake spec:rcov</tt>. If your changes are not 100% covered, go back to step 5.
7. Submit a pull request. Please do not include changes to the gemspec, version, or history file. (If you want to create your own version for some reason, please do so in a separate commit.)

Copyright
---------
Copyright (c) 2010 John Nunemaker, Wynn Netherland, Erik Michaels-Ober, Steve Richert. See LICENSE for details.
