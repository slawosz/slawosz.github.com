--- 
title: "Run single test case to speed up development"
layout: post
categories: ['en']
---
When you use common TestUnit (maybe with great gem "Shoulda":http://github.com/thoughtbot/shoulda) or "RSpec":http://rspec.info/, you may speed up development by running only single test case.
In TestUnit, run test file with `-n` switch:

{% highlight bash %}
ruby -n "name_of_test_method" path/to/test_file.rb
{% endhighlight %}

or use regexp:

{% highlight bash %}
ruby -n "/name of test method/" path/to/test_file.rb
{% endhighlight %}

Regexp is very convinient when you work with shoulda.

In Rspec, you can use `-e`:

{% highlight bash %}
rspec -e 'name of pattern' path/to/spec_file.rb
{% endhighlight %}

Or maybe you would like to install "autotest":http://www.zenspider.com/ZSS/Products/ZenTest/ or "autospec":http://github.com/dchelimsky/rspec/wiki/Autotest-Integration for Rspec?
