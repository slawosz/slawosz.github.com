---
title: "Add new resource with ajax"
layout: post
old_url: /2011/04/26/add-new-resource-with-ajax
categories: ['published']
---
In "Railscast 258":http://railscasts.com/episodes/258-token-fields "Ryan Bates":https://github.com/ryanb shows how to use "Token Fields":http://loopj.com/jquery-tokeninput/, javascript plugin which helps adding entries for <code>many</code> and <code>many to many</code> association. In this article I will exetend railscast and show you how to create not existing entries with ajax.

You can "download":https://github.com/slawosz/Add-resource-with-ajax-and-facebox-tutorial application for this tutorial from github or use it on "heroku":http://add-new-resource-with-ajax.heroku.com/ to see how it works. I started with application created by Ryan in 258 episode.

To add authors we can go to <code>/author/new</code> url, but when we already on <code>/books/new</code> we rather want to render form with ajax. To render form we will use "facebox":https://github.com/defunkt/facebox, writen in jquery by "Chris Wanstrath":https://github.com/defunkt. So, simply download facebox from github, decompress and place <code>facebox.css</code> in <code>public/stylesheets</code>,  <code>facebox.js</code> in <code>public/javascripts</code> and <code>loading.gif</code> and <code>closelabel.png</code> in <code>public/images</code>

Now, point write paths for images in <code>public/javascripts/facebox.js</code>:

{% highlight js %}
loadingImage: '/images/loading.gif',
closeImage: '/images/closelabel.png',
{% endhighlight %}

and place stylesheet and javascript in our layout <code>app/views/layouts/application.html.erb</code>:

{% highlight rhtml %}
<%= stylesheet_link_tag "application", "token-input-facebook","facebox" %>
<%= javascript_include_tag :defaults, "jquery.tokeninput","facebox" %>
{% endhighlight %}


Now it is time to show form for new author resource on facebox.

Lets add link in <code>app/views/books/_form.html.erb</code> partial:

{% highlight rhtml %}
<%= f.label :author_tokens, "Authors" %><%= link_to 'Add new author', new_author_path, :remote => true %><br />
{% endhighlight %}

It looks like a normal link, but with one exception: it has a remote option, indicating, that we want perform an ajax request with this link. This is new way in Rails 3 to generate ajax request and I think it is great. It simply adds a html <code>data-remote='true'</code> attribute to our link. Javascript code located in <code>public/javascripts/rails.js</code> binds <code>onclick</code> event on such link. Whe link is clicked, an ajax request is performed to links url, in our case to <code>new_author_path</code>. So now, we have rewrite new method in <code>AuthorsController</code>, which is our link destination. You can read more about Rails 3 unobtrusive javascript in "Simone Carletti":http://www.simonecarletti.com/blog/2010/06/unobtrusive-javascript-in-rails-3/ blog.

With Rails it is so simple :) We had to add only a new template in js format, <code>app/views/authors/new.js.erb</code>. When we click link, method new will recognize that request is in js format and render js template. So now, in template we will write code, which will render new action on facebox:

{% highlight rhtml %}
$.facebox('<%= escape_javascript(render :template => 'books/new.html') %>');
{% endhighlight %}


Code above generate a form on facebox with <code>app/views/books/new.html.erb</code> template. We have to add write new.html to generate html. Much more often in rich ajax Rails application we use partials, but I would like to show how to achieve our goal with minimal effort.

If you would like to know more about using facebox, see <code>public/javascripts/facebox.js</code>. Every use case is described in this file.

Now, we want to send our form and create a new author.
Normaly, we can use

{% highlight ruby %}
:remote => true
{% endhighlight %}

option in our form, but we only use remote form in facebox. So, let's add data-remote attribute to ajax form in facebox. To do it, we will add following to <code>app/views/books/new.js.erb</code>:

{% highlight rhtml %}
$('#facebox form').data('remote','true');
{% endhighlight %}

 This will add attribute <code>data-remote='true'</code> to our form, and with this attribute, <code>public/javascripts/rails.js</code> will process our form with ajax. After submiting, the request goes to method create in <code>AuthorsController</code> and  is looking for js template. In this template, we need only close facebok. So, we create file <code>app/views/authors/create.js.erb</code> and write:

{% highlight rhtml %}
$(document).trigger('close.facebox');
{% endhighlight %}

Thus we would like to perform other actions in html and js request in this method, we use <code>respond_to</code> block and redirect on html format and render template on js format:

{% highlight ruby %}
def create
  @author = Author.new(params[:author])
  if @author.save
    respond_to do |format|
      format.html { redirect_to @author, :notice => "Successfully created author." }
      format.js
    end
  else
    render :action => 'new'
  end
end
{% endhighlight %}

Now it works, but it need some more improvements. When <code>@author</code> instance cannot be save, we should render form again in facebox.  In <code>app/models/author.rb</code> we add validadion:

{% highlight ruby %}
validates :name, :presence => true
{% endhighlight %}

Now we cannot add new no-name author. When we try, we got validation error, which we want to show in facebox.

At first we need to change create action to use js format in else section:

{% highlight ruby %}
def create
  @author = Author.new(params[:author])
  if @author.save
    respond_to do |format|
      format.html { redirect_to @author, :notice => "Successfully created author." }
      format.js
    end
  else
    respond_to do |format|
      format.html { render :action => 'new' }
      format.js
    end
  end
end
{% endhighlight %}

And finally change <code>app/views/books/create.js.erb</code>:

{% highlight rhtml %}
<% if @author.save %>
  $(document).trigger('close.facebox')
<% else %>
  $('#facebox .content').html('<%= escape_javascript(render :template => 'authors/new') %>')
  $('#facebox form').data('remote','true');
<% end %>
{% endhighlight %}

Now we have fully working form on facebox, with minimal changes in our code. We can do last small improvement, remove link 'Back to List'. In facebox link is completely unnecessary. After checking source of <code>app/views/authors/new.html.erb</code> we simply need remove last <code><p></code> tag. In jQuery, it's simple:

{% highlight js %}
$('#facebox .content p:last').remove();
{% endhighlight %}

We add this line at the end <code>app/views/authors/new.js.erb</code> and at the end in else section in <code>app/views/authors/create.js.erb</code>. Now we are done.

See our application working on "heroku":http://add-new-resource-with-ajax.heroku.com/, and check the
repo on "github":https://github.com/slawosz/Add-resource-with-ajax-and-facebox-tutorial.
All changes you can see in "cf13adacdff595059dc6eba0fcb33c0204ad6714":https://github.com/slawosz/Add-resource-with-ajax-and-facebox-tutorial/commit/cf13adacdff595059dc6eba0fcb33c0204ad6714

I hope this tutorial will be helpfull for You. Any feedback is welcome.



