---
title: Importing Mephisto comments into Disqus
layout: post
---

I've switched to [Jekyll][] for managing my blog. It's funny how
things come full-circle. Once you get the power of a really good
editing environment (this time for me, it's Emacs, but it could just
as easily have been vim) you stop wanting to write words anywhere
else.

Jekyll doesn't do comments, obviously, since it's really a static site
generator, so I hooked up [Disqus][]. There's 
[a little Sinatra app][disqus-import] to import comments into Disqus, but
it only works via RSS, and Mephisto doesn't export all the comments into
the RSS feed, just the 10 most recent.

So I hacked it to use a local MySQL database of comments instead. It's
presented here [and as a gist][mephsito-to-disqus] for your enjoyment.

{% highlight ruby %}
# Copyright 2009 Michael Ivey, released to public domain
# Disqus guts lifted from http://github.com/squeejee/disqus-sinatra-importer
# I wanted it to run from MySQL and command line, instead of a Sinatra app

require 'rubygems'
require 'feed_tools'
require 'rest_client'
require 'json'
require 'sequel'


# Fill these in. See disqus-sinatra-importer for details on what they do
# if they're not obvious
user_api_key = ''
forum_shortname = ''
current_blog_rss = ''
db = ''
db_user = ''

disqus_url = 'http://disqus.com/api'

resource = RestClient::Resource.new disqus_url
forums = JSON.parse(resource['/get_forum_list?user_api_key='+user_api_key].get)
forum_id = forums["message"].select {|forum| forum["shortname"]==forum_shortname}[0]["id"]
forum_api_key = JSON.parse(resource['/get_forum_api_key?user_api_key='+user_api_key+'&forum_id='+forum_id].get)["message"]

db = Sequel.mysql(db, :user => db_user, :host => 'localhost')
query = "SELECT title, body, author, author_email, created_at FROM contents WHERE type = 'Comment'"

# Get all of the articles from the current blog site
articles = FeedTools::Feed.open(current_blog_rss)

db[query].each do |comment|
  comment_article_title = comment[:title]
  
  # Get the blog article for the current comment thread
  article = articles.items.select {|a| a.title.downcase == comment_article_title.downcase}[0]

  if article
    article_url = article.link  
    
    thread = JSON.parse(resource['/get_thread_by_url?forum_api_key='+forum_api_key+'&url='+article_url].get)["message"]
    
    # If a Disqus thread is not found with the current url, create a new thread and add the url.
    if thread.nil?  
      thread = JSON.parse(resource['/thread_by_identifier'].post(:forum_api_key => forum_api_key, :identifier => comment[:title], :title => comment[:title]))["message"]["thread"]
      
      # Update the Disqus thread with the current article url
      resource['/update_thread'].post(:forum_api_key => forum_api_key, :thread_id => thread["id"], :url => article_url) 
    end
    
    # Import posts here
    if resource['/create_post'].post(:forum_api_key => forum_api_key, :thread_id => thread["id"], :message => comment[:body], :author_name => comment[:author], :author_email => comment[:author_email], :created_at => comment[:created_at].strftime("%Y-%m-%dT%H:%M"))
      puts "Success: #{comment.author} on #{comment.title}"
    end
  end
end


{% endhighlight %}

[Jekyll]: http://github.com/mojombo/jekyll
[Disqus]: http://disqus.com
[disqus-import]: http://github.com/squeejee/disqus-sinatra-importer
[mephsito-to-disqus]: http://gist.github.com/43440
