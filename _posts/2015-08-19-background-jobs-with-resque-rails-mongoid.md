---
title: Background jobs with resque, rails, and mongoid
date: 2015-08-19 12:45 PM
category: coding
from: stephanieliu.me
tags:
- rails
- resque
- redis
- mongoid
- mongodb
- ruby
---
For those (like me) who have/had no idea what background jobs are, let's start
with that (though I'm not even sure I know exactly what they are, to be fair). I guess
the idea is that, sometimes things fail, and when things fail, you don't want everything
to fail. Like lightbulbs in series vs lightbulbs in parallel. When they're in series, if
one bulb goes out, they all go out. But if they're in parallel, when one dies, the others
don't go down with it. So, in your app, if you're sending out all these emails, and one
of them fails, you don't want all subsequent emails to fail! You want the rest to keep
truckin, right?


Hence libraries like [delayed_job](https://github.com/tobi/delayed_job) and
[resque](https://github.com/resque/resque), which let you add jobs to queues and use
separate workers to take jobs from the queues and perform them. If one worker fails, it's
fine, cause another worker will come along and finish off the queue!


So now that we have a preliminary idea of what background jobs are, let's
start. Resque is a redis-backed library, so you'll need to install redis for this to work.
You can install it with brew `$ brew install redis` or you can download the latest
[tarball](http://redis.io/topics/quickstart). Then you'll need to run redis in the background
with `$ redis-server`.


Next, add the following two gems to your gemfile and bundle:

{% highlight ruby %}
gem 'resque'
gem 'resque-scheduler'
{% endhighlight %}

Rails uses [ActiveJob](http://edgeguides.rubyonrails.org/active_job_basics.html)
as its main job infrastructure, so we can continue using ActiveJob and not have to change
much code, but tell it to use Resque as its queueing backend.

{% highlight ruby %}
# config/initializers/active_job.rb
ActiveJob::Base.queue_adapter = :resque
{% endhighlight %}

Next, we'll need to tell our app where our redis server is located, which
will by default run on port 6379

{% highlight ruby %}
# config/initializers/resque.rb
Resque.redis = Redis.new(url: 'redis://localhost:6379')
{% endhighlight %}

The next part is optional, but helpful. If you add the following lines to your
routes file, you'll be able to go to '/resque' and see all your queues, jobs, and failing
jobs on a nice little interface. You can even retry jobs from there.


{% highlight ruby %}
require 'resque/server'
Rails.application.routes.draw do
  mount Resque::Server.new, at: '/resque'
  # The rest of your routes...
{% endhighlight %}


Alright, last part of the setup: you'll need to load the rake tasks for
resque.


{% highlight ruby %}
require 'resque/tasks'
require 'resque/scheduler/tasks'
namespace :resque do
  task setup: :environment do
     require 'resque'
     require 'resque-scheduler'
  end
end
{% endhighlight %}


If you run `$ rake -T resque` you'll be able to see all tasks that
were loaded by resque


Now, let's start adding our jobs to queues. If you're already sending
some emails in your rails app, then all we have to do is tell the Mailer objects
not to deliver now, but to "deliver later" which will add those jobs to the "mailer"
queue instead of performing them immediately.


{% highlight ruby %}
MyMailer.send_cool_email(user: current_user).deliver_later
# instead of
MyMailer.send_cool_email(user: current_user).deliver_now
{% endhighlight %}


Now, if you go into your rails console, you should be able to check that
there are jobs in the queue.


{% highlight ruby %}
MyMailer.send_cool_email(user: User.first).deliver_later
Resque.size('mailers') # 1
{% endhighlight %}


To start some workers to perform these jobs, you can run the rake task with
a specified queue, or set QUEUE=* to perform all jobs from any queue.


{% highlight bash %}
$ rake resque:work QUEUE=mailers
{% endhighlight %}


And that's pretty much it! One more thing though, if you're using Mongoid
with resque, you'll need to add `include GlobalID::Identification` to any model you're
passing to the Mailer methods (in these examples, the User model). If you wanna know why, you can read about it
[here](http://stackoverflow.com/questions/27820934/is-mongoiddocument-a-globalididentification-for-activejobs/28263871).
