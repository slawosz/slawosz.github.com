---
title: "How do I test? Spork and Emacs experience."
layout: post
old-url: /2011/08/11/how-do-i-test-spork-and-emacs
categories: ['en']
---

 Few weeks ago "Ryan Bates":https://github.com/ryanb created a "screencast":http://railscasts.com/episodes/275-how-i-test about testing. He uses "Guard":https://github.com/guard/guard to run specs. Guard watches changes in files and may run appropriate specs files, which is very convenient. However, I have a little different way to run specs.

I do not always like to run full spec file, very often I need only to run one spec. To accomplish it very easily, I use few functions in my emacs, thanks to <span class="strike">"rinari":https://github.com/eschulte/rinari mode</span> "emacs-rails":https://github.com/remvee/emacs-rails mode:

<code>rails-spec:run-this-spec</code> - runs specs with current line as rspec line parameters. I use this command very very often (<code>C-c C-c ,</code> shortcut)

<code>rails-spec:run-current</code> - runs all spec in current spec file or specs for file I actually edit. It works exactly like Guard, but I use it only when I need to (<code>C-c C-c z .</code>shortcut)

<code>rails-spec:run-last</code> - this function runs lately ran spec. It is great for TDD/BDD workflow, because with this I can run only one spec while I work on implementation in corresponding file (<code>C-c C-c z l</code> shortcut)

One of the most important things in TDD/BDD process is running specs after each change in code. Thus we run spec even hundreds times in (good) developer day. That is why specs should run very fast. We may say, that specs that run more than 10 seconds are distracting  and may cause checking lastest rss/facebook/google+ news ;) Of course, I talk about single spec or single spec file, because it is normal that running entire test suite takes few minutes in large projects. But we run entire test suite before commit, or even we use CI.

<h2>Spork</h2>

"Spork":https://github.com/timcharper/spork is a great tool for TDD/BDD. It is test server, it loads our app to memory and forks for spec you actually run. With this, spec runs immediately, without waiting for rails app bootstraping. It makes a big difference in specs speed. For example I did some benchmarks in Teambox project:

{% highlight bash %}
All spec file, without spork:
$ time (bundle exec rspec spec/models/user_spec.rb)

...............................................................................

Finished in 22.38 seconds
79 examples, 0 failures

real	0m35.714s
{% endhighlight %}

Single spec:

{% highlight bash %}
$ time (bundle exec rspec spec/models/user_spec.rb -l 90)

Run filtered using {:line_number=>90}
.

Finished in 0.54218 seconds
1 example, 0 failures

real	0m14.283s
{% endhighlight %}

And with Spork:
Whole spec file:

{% highlight bash %}
$ time (bundle exec rspec spec/models/user_spec.rb --drb)
...............................................................................

Finished in 24.55 seconds
79 examples, 0 failures

real	0m27.399s
{% endhighlight %}

And single spec with spork:

{% highlight bash %}
$ time (bundle exec rspec spec/models/user_spec.rb -l 90 --drb)
Run filtered using {:line_number=>90}
.

Finished in 0.51661 seconds
1 example, 0 failures

real	0m3.286s
{% endhighlight %}

Nice speed up!  With spork, single spec lasts about 3 seconds. In other projects with longer bootstrap time, we save much more time!

To run rspec with spork, first run spork <code>`spork`</code> and then run any rspec with <code>`--drb`</code> option. Do not forget to add this option to spec runner in your editor. Before first run we have to bootstrap spork with command <code>`spork --bootstrap`</code>

<h2>My test flow:</h2>

1 I write spec
2 Run only this spec <code>(rails-spec:run-this-spec</code> in emacs)
3 It does not pass.
4 I write the code to make test pass
5 I run last spec (<code>rails-spec:run-last</code> in emacs)
6 If spec does not pass, back to point 4
7 Spec pass
8 Run  spec for entire file (<code>rails-spec:run-current</code>) or even for the whole project.

I am hope that this note will usefull for You. I encourage you to run
your spec with spork and run them with your editor. It gives you much
more fun in TDD/BDD!



