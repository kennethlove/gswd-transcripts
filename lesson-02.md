Welcome to episode 2 of Getting Started with Django. In the first episode, we created the Microblog project and got it running on Heroku, but it didn't really do anything. Now let's take it further and get something we can actually use.

The first thing you'll want to do is go back to the directory where you previously did `vagrant up` and run that command again. Whenever you're finished working, you'll want to run `vagrant halt` or `vagrant suspend` so that vagrant will power down the VM and the VM will be waiting for you to come back. If you run `vagrant destroy`, it'll delete the VM and wipe out all of the work you've done installing packages and the like. Your code will still be safe, of course, because it's on your system, not the VM.

Source the environment again (`source ~/blog-venv/bin/activate`), and go back to the `/vagrant/projects/microblog/` directory.

### South ###

The first thing we need to do is install [South](http://south.readthedocs.org/en/0.7.6/). South is a great Django tool that handles data and schema migrations. Schema migrations are what we call changes to your database structure, and data migrations would be scripted changes of data. `pip install south` will get it installed and then you'll want to add `south` to the `THIRD_PARTY_APPS` tuple in `settings/base.py`. The last thing we need to do is run `python manage.py syncdb` to create South's needed table.

### Our First App ###

Since our project is a blog, we should create an app named "blog". It's typically best practice to name apps with plural names, but "blogs" doesn't fit our intended usage; we're creating one blog, not multiple blogs.

Run `python manage.py startapp blog` and Django will create all of our needed files for us in a `blog/` directory. This shows off the structure of a Django project in that `microblog` is sort of a meta-app that really just controls the entire project instead of being a standard app with models and such.

Open up `settings/base.py` again and add `blog` to the `LOCAL_APPS` tuple. Now we'll open up `blog/models.py` to start creating models for our app.

#### The Post Model ####

The first model we're going to create is going to be named `Post`. Model names are always singular because they, logically, represent a single instance of that object. We'll extend `models.Model` to make our class inherit from Django's generic `Model` class.

##### Fields

To start on the model, we'll add a field named `created_at` that is a `DateTimeField` with `auto_now_add=True` and `editable=False`. This will cause the model to save a timestamp of when an instance was first created and not show this field to us for editing in the admin.

Next we'll add `updated_at` with the same `editable=False` but with `auto_now=True`, which'll save a timestamp every time the model is saved.

`title` is our third field, and it's a `CharField` with a `max_length` of 255. Since we want the title to always exist and be required, we won't add `blank=True` or `null=True` to the declaration. We'll also add a `slug` field of type `SlugField` with another `max_length` of 255, but we'll let this field have `blank=True` which makes the field not required for validation purposes. We'll also give `slug` an attribute of `default=''` so that, if no slug is provided, it just gets a blank string. We'll set the slug ourselves in the `save` method later.

Next we'll add a `content` field which is a `TextField` which makes it fairly freeform in the amount of text it holds.

Lastly, we'll create a `BooleanField` named `published` which has a `default` of `True`.

This finishes up our fields.

```python
class Post(models.Model):
    created_at = models.DateTimeField(auto_now_add=True, editable=False)
    updated_at = models.DateTimeField(auto_now=True, editable=False)
    title = models.CharField(max_length=255)
    slug = models.SlugField(max_length=255, blank=True, default='')
    content = models.TextField()
    published = models.BooleanField(default=True)
```

##### Methods

We're going to add an `__unicode__` method to our class. This method is called any time Django wants to know how to represent a model instance. We want our Posts to be represented by their title, so we'll `return self.title`.

I mentioned that we would set the `slug` value ourselves in the `save` method, so we'll do that now. In our `save` method, we'll check to see if there is a `slug` value already, and if there isn't, we'll set it to the `slugify`-ed value of our Post's `title`. Since we call `slugify`, we need to `import` it from `django.template.defaultfilters`. Then we need to call `super` on the `save` method so it does the work defined in `models.Model.save`.

```python
def __unicode__(self):
	return self.title

def save(self, *args, **kwargs):
    if not self.slug:
        self.slug = slugify(self.title)
    super(Post, self).save(*args, **kwargs)
```

#### Migration

Now that we have a model defined and the app installed, we need to get the model into the database. We don't want to run `syncdb` so we'll create a schema migration using South.

`python manage.py schemamigration --initial blog` will create the initial schema migration for our blog. If you explore a bit, you'll see that it created a `migrations/` directory for us with a `0001_initial.py` file in it. That file is our migration, which you can open and read if you want to.

Now to apply it, `python manage.py migrate blog`. If you run this without the app name at the end, South looks for all migrations in all apps that haven't been run. Using the app name just saves some time.

Looking at the output, you'll see that it created our blog model and installed no initial data. Let's make this more visible.

### Admin

In the `blog/` folder, `touch admin.py` so we can create an admin for the Posts. The first thing we need in the `admin.py` file is to import `admin` from `django.contrib`. The next thing we need is our model. Since we're in the same app, and it's generally a good idea to use relative imports, we'll import `Post` from `.models`.

Since we just want to get something up, the last line is `admin.site.register(Post)`, which'll create a default `ModelAdmin` for the `Post` model. Save that, switch back to the `microblog/` project directory and run the server again (`python manage.py runserver 0.0.0.0:8000`).

Load the admin (`http://127.0.0.1:8888/admin/`) and you should get the login form. Log in and there will now be a "Blog" area in the list. Going to "Posts" will show that none exist. We can add one quickly (in the video, I put in `==` in the `save` method instead of just `=`.)

If it worked correctly, you should see all or most of the content of your Post title but with "-"" instead of spaces. So, what can we do to make this more useful?

#### Customization

Let's start my changing how the Post list is displayed.

First we need to create a real `ModelAdmin` for our model. We'll give it the logical name of `PostAdmin` and it will extend `admin.ModelAdmin`. Unlike most other things that deal with models, you don't set a model attribute on the class or in its `Meta`.

We'll set `date_hierarchy` to be equal to our field `created_at`. This allows us to browse our posts based on their creation date. To test this, we need to associate the model and the admin. So, where before we had `admin.site.register(Post)`, we need to add `PostAdmin` in after the model. Refresh the admin and you should see date navigation at the top of the list.

Some other handy admin options:

* `fields` lets us control what fields show up in the admin form. We'll set ours to `("published", "title", "content")`. It also controls order of the fields, so this will put `published` at the top of the form.
* `list_display` controls the fields shown on the list page's table. Start off by setting that to `["title", "updated_at"]`. Change that to include `"published"` in the first position. Now it has a checkmark for published or not, but you'll notice that the checkmark is now the link to edit the post, not the title. We should fix that.
* `list_display_links` let's us control which columns should be linked to the instance. We'll set ours to `["title"]` so that the title is the only thing linked to the edit page.
* `list_editable` makes some columns editable from the list page. We'll set ours to `["published"]` so we can publish or unpublish posts more easily. You'll see that this changes the checkmark to a checkbox and gives us a "Save" button.
* `list_filter` provides us with a set of filters on the side of the list. We'll set ours to `["published", "updated_at"]` so that we can filter posts to show us only the published or unpublished ones, or to show us all the posts in a given timeframe. Create a new, unpublished post to test this functionality if you want.

### New Field

We should extend our model to include an `author` field, so we know who wrote a blog post. Go back to `blog/models.py` and, at the bottom of the list of model fields, add a new one named `author` that's a `ForeignKey` to the `User` model. Set it's `related_name` to be `"posts"`.

```python
author = models.ForeignKey(User, related_name="posts")
```

We now need to import the `User` model. That comes from `django.contrib.auth.models`. **NOTE**: Once Django 1.5 is released and widely used, you can't rely on this model being used any more. In later videos, we'll talk about how to take advantage of Django 1.5's customizable auth features.

#### Related Names

Related names seems to be an area that a lot of people get confused with. The `related_name` attribute is what the foreign model, the one you're linking to, would call the local model, the one you're editing. Obviously the author of a post would call his or her collection of posts "posts", and not, as I often see, "authors" or the like. If you don't provide this, the foreign model gets an automatic attribute of `<model name>_set` that works the same way. You can set `related_name` on `ForeignKey`, `ManyToMany` and `OneToOne` relationships. The name should always be pluralized except in the case of a `OneToOne` when it should be singular. Naming things this way makes your code more logical and spoken-language-like.

#### Migration

Now our model has a field that our database table doesn't. If you try to read or edit a post, or view a list of posts, in the admin, you'll get a `DatabaseError`. We'll correct this with another `schemamigration` (**NOTE**: You can just [update the last migration](http://south.readthedocs.org/en/0.7.6/commands.html#id1), but I think it's more beneficial to see the migration process). Back at the command prompt, run `python manage.py schemamigration --auto blog add_author_to_blog_posts`. You can leave the name off and South will generate a name for you.

When you create the migration, South will ask you for default information for the `author` field. It asks this because we have existing rows in the database and we didn't specific a default for the field and we didn't allow it to be `null`. Since we know our existing records are written by our user, we can just tell South to default them to a value of "1", which is our User's ID. If you were running this on real data, you'd want to either provide a default value, or allow it to be `null` and then set the values yourself later.

Run the migration (`python manage.py migrate blog`) and then start the server back up.

### Back To The Admin

Refresh the admin and there should be no problems. We don't, however, see our `author` field, so let's add it to `fields` in the `PostAdmin` class. Refresh and you should be able to add an author to your posts.

We should now allow filtering by authors on the list. Again, add `author` to the `list_filter` attribute. Then go and create a new user in the admin and then create a new post by the user, or change an old post to have that user as the author. Now you can see how the filtering works for authors.

### Ordering On The Model

It's a good idea to be set the ordering of items on your models. You can set it expressly for the admin, but that won't do you much good everywhere else that you display your models. By default, Django orders items based on their IDs, which is usually auto-incremented for each instance you create. That's not always the desired behavior, though, so let's explicitly control it.

Add a `class Meta` to the `Post` model and set `ordering` to `["-created_at", "title"]`. The `-` at the beginning of `-created_at` tells Django to order them in descending order, or newest-first. Providing two fields like this says if there are more than one instance with the exact same `created_at`, order them alphabetically by `title`.

### More Admin

Now we'll set up `prepopulated_fields` with our `slug` field. Add `slug` into the `fields` tuple and then add the `prepopulated_fields` attribute to the class.

It needs to be set to a dict where the key is the field that'll be populated with data, and the value is a tuple of fields that'll be used for the population. We want our key to be `slug` and our value to be `("title",)`. We have to provide the trailing comma to make it a tuple.

If you refresh the admin and go to create a post, you'll notice that that slug field auto-updates as you type into the `title` field. You can edit the slug, too, to make it more unique if you need to. If you didn't want it to be editable, you could add `slug` to the `readonly_fields` list or tuple.

The last thing we'll do in the admin is add in the `search_fields` attribute. This lets us search for records based on the data in specific fields. We'll set ours to be `["title", "content"]` so that we can search for words in our post's titles or content area.

Refresh and you'll see the search box at the top. Enter a word you've used in a post title or content and you'll see those items show up.

Add all the new files to git and then commit them and the other changes. We won't push to Heroku just yet, though.

### Public Views

We have models, and we have an admin, which is great, but there's not public face to our site. Let's fix that.

If you load the front page of our site, we just get our old "Hello World" template. In `blog/views.py`, you'll see that there are no views because we haven't created any.

First off, let's import `Post` from `.models`. This way we'll have it available for any views that need it.

#### Function-based views

Most people, when learing Django, and some, even when using it professionally, write all of their views as functions. This was the default and obvious method to use for all versions of Django up to 1.3. Since it's widely used, we'll go over making views as functions.

##### `blog_list`

So, to create our first view, we'll `def`ine a function named `blog_list` and it'll take `self, request, *args, **kwargs` as its arguments. At the top of the function, we'll create a list of blog posts named `blog_list` by selecting `Post.objects.filter(published=True)`. We'll set another variable named `template_name` to "post_list.html", since we'll be displaying a list of posts.

Finally, we'll `return render(request, template_name, context)`. We need to create the context dict before we can return it, though. We'll set the key `post_list` equal to the variable `post_list`. And then we'll import `render` from `django.shortcuts`.

```python
def blog_post(self, request, *args, **kwargs):
	post_list = Post.objects.filter(published=True)
	template_name = "post_list.html"
	
	context = {
		"post_list": post_list
	}
	
	return render(request, template_name, context)
```

###### Templates

We need to create some templates before we can render them. Inside the `blog/` directory, create a new directory named `templates/`.

**NOTE:** Why create templates in the `blog/` directory instead of the `templates/` or `microblog/` directories? Because these templates are specific to the blog app and if you ever wanted to make this app distributable, or simply use it again in another project, you now only have to deal with one specific directory, instead of hunting all around the project for related files.

Inside the `blog/templates/` directory, create a new directory named `blog/` (**NOTE:** I did this thinking I was making templates for class-based views. It's a mistake, but it gets corrected). This gives you the structure of `blog/templates/blog/`. Inside there, touch `post_list.html`.

Then, in `microblog/templates/`, make a new directory here named `_layouts/` (the `_` keeps it at the top of most lists, so it's easy to find). Inside of `_layouts/`, touch `base.html`. This will be our base template that all or most other templates will inherit from.

Run the server again so it'll be waiting for us.

In `base.html`, add a basic HTML stub with a couple of Django blocks. So, what are blocks? Blocks are areas in a template that can be replaced in templates that extend them. View templates can replace, for instance, the content of a page or the Javascript that's run on a page, or any other item without having to repeat the *entire* template.

```html
<!doctype html>
<html>
    <head>
        <title>{% block page_title %}{% endblock %}Microblog</title>
    </head>
    <body>
		{% block page_content %}{% endblock %}
    </body>
</html>
``` 

Now we want to edit our `post_list.html` template. We'll start by extending the `_layouts/base.html` template. And then we'll override the blocks. We'll set our `page_title` block to have "Blog posts" in it, and our `page_content` block will have an `<h1>` and then a `<ul>`. In the `<ul>` we'll loop over all of the posts in our `post_list` variable and, for each one that's there, we'll create an `<li>` with the Post's `title`. If we don't have any posts, we'll give a friendly message asking them to come back soon.

```html
{% extends "_layouts/base.html" %}

{% block page_title %}Blog posts | {% endblock %}

{% block page_content %}
<h1>Blog Posts</h1>
<ul>
	{% for post in post_list %}
	<li>{{ post.title }}</li>
	{% empty %}
	<li>Sorry, no posts yet. Check back soon!</li>
	{% endfor %}
</ul>
{% endblock %}
```
###### URL

We now, obviously, need a URL for this. Open up `microblog/urls.py` and take out the `TemplateView` we created before. We need a blank home url, though. For now, we'll just use our post list view as our homepage. So import `blog_list` from `blog.views` and then, in the `patterns` create the default url:

`url(r"^$", blog_list),`

###### Problems

Refresh and we get an error about the function needing two arguments when only one was provided. This is entirely my fault; I'm so used to writing methods on classes that I put `self` into the function signature. Remove `self` from the `post_list` function. Refresh and a new problem appears, Django can't find the template.

Move the templates up one directory out of the `blog/templates/blog/` directory. They should now live at `blog/templates/`.

Back in our `base.html` template, let's add an `<h1>` that says "Welcome to my blog" and then change the `<h1>` in `post_list.html` to an `<h2>`. Refresh and you'll see the new welcome message.

##### `blog_detail`

Our new function, `blog_detail` will take the same arguments as `blog_list`, but with the addition of a `pk` argument after `request`. This'll let us have quicker access to a variable passed in through the URL so we can grab a specific Post instance.

This time, instead of building a list of posts, we want to get one specific Post instance. So we'll create a new variable that `get`s a Post that is published and has the `pk` we've passed in through the URL. We'll set the `template_name` and `context` like before and `return render` like before, too.

```python
def blog_detail(request, pk, *args, **kwargs):
	post = Post.objects.get(pk=pk, published=True)
	template_name = "post_detail.html"
	
	context = {
		"post": post
	}
	
	return render(request, template_name, context)
```

###### New template

Now we need to create our new template. Touch `blog/templates/post_detail.html` and then open it up for editing. It works very similarly to our `post_list.html` template, `extend`ing "_layouts/base.html" but for the `page_title` block, we'll display `post.title`. In `page_content`, we'll create an `<h2>` with the post's title and then print out the `post.content` with `linebreaks` filter. Finally, a `<p>` tag with the post's `updated_at`.

The `linebreaks` filter turns a blank line into a `<p>` tag. It's not very smart or advanced, but it's plenty for a small, quick blog.

In the admin, if you go to edit a post and look in the URL, you'll see a number, which is the ID of the post. Our first one should have an ID of "1".

###### URL

Back in `urls.py`, we'll add another new URL, this time one that accepts a PK in a named regex block. 

`url(r"^blog/(?<pk>\d+)/$", blog_detail),`

Then we'll also be sure to import `blog_detail` from our `views` module. Save and then load `http://127.0.0.1:8888/blog/1` and you should see the blog post. Obviously we don't want to have to type in explicit URLs like that.

First, let's name our URLs. Let's give the newest one the `name` of `"blog_detail"`, and the previous one the `name` of `"blog_list"`.

Now, back in the `post_list.html` template, we need to make the post's name a link to the detail view. Inside the `<li>` that's created for each post, add an `<a>` tag who's `href` attribute is `{% url blog_detail post.pk %}`. This will resolve the URL with that name and argument and replace the tag with that value.

**NOTE:** The `{% url %}` tag is changing in Django 1.5. The URL name must now be quoted. Since you use the `{% url %}` tag *a lot*, I don't think it's worth the trouble of having to edit all of your templates when you upgrade to Django 1.5 or beyond. So, at the top of your templates, load the new `{% url %}` tag from future with `{% load url from future %}`. Now make sure all of your URL names are quoted and you've made your code future-ready.

Back in the `post_detail.html` template, add a new link at the bottom to go back to the list of posts. `<a href="{% url 'blog_list' %}">&larr; Blog list</a>`. Again, make sure you import the future `{% url %}` tag.

Refresh the list page and your blog post titles should be links. Click them, you should see the blog post, and have a link at the bottom that will take you back to the list.

**Congratulations**, you have a working blog.

#### Cleanup

We probably don't want to have our blog list as our home page. So instead of `r"^$"` for the URL pattern, change it to `r"^blog/$"`. Now anything that's just `blog/` will load this view, and if it has a PK after it, it'll load the detail one.

##### Homepage

Now let's create a homepage view. Create `microblog/views.py` and open it for editing and restart the server. We're going to do this view as a class-based view. Actually, from this point forward, we're going to do class-based views (where applicable) because they're officially endorsed as the correct way to do generic views and they're what I prefer creating.

So, import `TemplateView` from `django.views.generic`. Then create a new class named `HomepageView` which extends `TemplateView`. It's `template_name` attribute should be set to `"index.html"` and that's all we have to do for the view.

###### URL

Now back in `urls.py`, we'll recreate the blank URL, `r"^$"`, and point it to `views.HomepageView.as_view()` with a name of `"home"`. This means we need to import the view, so do that at the top with `from . import views`, which imports all of our `microblog` views.

###### Template

If we load that URL, we get our old "Hello World" template, which isn't really useful anymore. So let's go edit the `index.html` template.

At the top, it should extend our base template like the others. We don't, however, want to override the `page_title` block since we want the default page title. In `page_content`, we'll just provide a link to our blog list. `<a href="{% url 'blog_list' %}">Read my Blog</a>`. And, of course, we need to load the `{% url %}` tag.

One last thing, the "Welcome to my blog" in `_layouts/base.html` now makes less sense, so let's change "blog" to "site".

##### URLs

Our project is fairly logical and straightforward, but, at least to me, it doesn't make much sense to have all of our URLs in one location; it would be cleaner to have them split up by app.

Let's put them into their own module *and* namespace them, to improve the organization of our project.

First, create `blog/urls.py`. To start this file, feel free to copy and paste from `microblog/urls.py`. Assuming you do this, remove the import for `admin` and the one that imports from `blog.views` because the line above that one, the one that imports `views` from `.`, will cover that for us.

We don't need the `admin.autodiscover()` and we don't need the example URLs, so take them out along with the URL for `admin` at the bottom and the `include` import.

Next, put `views.` in front of each of the functions in the URLs. Having these URL names have the prefix "blog" really only makes sense if they're in the midst of a bunch of other URLs. Since they won't be, let's take that prefix off. Your URLs should now be named `list` and `detail`. We also need to take `blog/` off of the regex for both URLs, since that, again, only makes sense in a larger list of URLs. Save and go back to `microblog/urls.py`.

In `microblog/urls.py`, remove the URL for the `blog_detail` view. Where we have the one that's for anything starting with `blog/`, we want to take off the `$`. Instead of the view function and name, we want to add in an `include()`. We want to `include("blog.urls", namespace="blog")` which will include those URLs and stick them in the `blog` namespace. We can also remove the import `from blog.views`.

What does this all mean? It tells Django that, when it's building its list of URLs, we want all of the URLs in the `blog/urls.py` file to be included, prefixed with the regex pattern of `^blog/`. It also says that, when I'm referencing these URLs by name, they should be grouped into this `blog` namespace. Let's see how that affects our work.

###### Template URLs

If we refresh, we'll get an error because Django can't turn our `{% url 'blog_list' %}` tag into a real URL. In `index.html`, we need to change our URL. As `blog_list`, it's almost right. We changed the URL name to `list` *but* we also gave it a namespace, so we need to specify that, too. Namespaces, in URLs, are defined with a `:`, so change `{% url 'blog_list' %}` to `{% url 'blog:list' %}`, replacing the `_` with a `:`. 

This needs to be repeated in `blog/post_list.html` and `blog/post_detail.html`. Now all of our views and templates should be working.

To me, it's a lot easier to read a single application's URLs at once than it is to read all of the URLs for my entire project, especially when you get in the 10s and 100s of total view functions or classes. The biggest downside, though, is that, from app-to-app, your `urls.py` files start to look very similar. I don't find this to be a deal-breaker.

### Class-Based Views

First off, why would we want to use class-based views? Our existing views, these function-based ones in `blog/views.py`, are simple and easy to read. But, if you've been paying attention, you'll notice that they're very similar. We've repeated a lot of the same language and methods in both views. We could abstract that out to another function, but then we're better off using a class-based view to begin with.

We'll replace each of these views with a class-based alternative.

#### List View

The first one we'll do is `blog_list`. We'll make a new class named `BlogListView` and it'll extend `ListView`, which we'll need to import. The only attribute we need to specify is the `model`, which we'll set to `Post`. Add `from django.views.generic import ListView` to your imports at the top. Since we'll need it later, let's import `DetailView` from there as well.

To use the view, we need to add it our `blog/urls.py`. Where we had `views.blog_list`, let's now use `views.BlogListView.as_view()`. Actually, we should name this more intelligently. Change it to `PostListView` in both `blog/urls.py` and `blog/views.py`.

Refresh the site, go to the post list url and… template isn't found. We need to move the templates back to where we created them initially. So move all of the HTML files in `blog/templates/` into `blog/templates/blog/`. Refresh and everything should show up.

#### Filtering

Yes, *everything* showed up, including our non-published blog post. To filter the queryset for the view, we need to override the `get_queryset` method on the class. This method exists on any view class that deals with fetching objects from the database, so `ListView`, `DetailView`, `UpdateView`, etc. It only takes the `self` argument. We'll do a `super()` call on it, so it gets the normal queryset, and then we'll `return` a copy of the queryset that's been filtered to only show published posts.

```python
def get_queryset(self):
	queryset = super(PostListView, self).get_queryset()
	return queryset.filter(published=True)
```

Refresh and now you should only see the published posts.

#### Detail View

If you click on the post, you'll get an error about a missing template, so add `blog/` in front of the template name in the `blog_detail` view so it works.

We mentioned before that the URL contains the PK of our blog post. What happens if a user changes the PK portion to a PK that doesn't exist? It throws a 500 error because we have nothing catching the 404. In our existing function-based view, we'd either wrap the `Post.objects.get(…)` call with a `try/except` that catches `Post.DoesNotExist` and calls `raise Http404`, or we'd change the `.get(…)` to use Django's `get_object_or_404` method. Both of these obviously work but you have to do them each and every time.

Luckily, class-based views already do this for us.

So let's create our class for the detail view. We'll name it `PostDetailView` and it'll extend `DetailView`. Again the `model` is set to `Post` and we'll do the exact same queryset override as before.

In `blog/urls.py` we change `views.blog_detail` to `views.PostDetailView.as_view()`. If we refresh we'll see that we forgot to change the class name in the `super()`. So change `PostListView` to `PostDetailView` and refresh again.

If you now try to go to an invalid PK, it throws a 404 instead of a 500, which is the behavior  we want to see.

##### Cleanup

Remove the two function-based views and the imports that they required (like `render`) and save again. Our `urls.py` and `views.py` files are now very small and concise and that, as developers, should make us very happy. Concise except for one thing, the fact that we override the queryset, explicitly, both times. How could we fix that?

Well, we could create a `ModelManager` for our model. This is actually a really good idea and one that we'll do later. We'd still have to change the queryset, though, so that doesn't solve 100% of our problem.

##### Mixin

In both of our class-based views, we override the queryset, so let's create a mixin that does that for us and allows us to only have to do it *once* in the code.

At the top of the file, let's create a new mixin named `PublishedPostsMixin` and it'll extend `object`. Copy and paste the `get_queryset` method from one of the views and change the class name in the `super()` to `PublishedPostsMixin`.

```python
class PublishedPostsMixin(object):
	def get_queryset(self):
		queryset = super(PublishedPostsMixin, self).get_queryset()
		return queryset.filter(published=True)
```

Now remove `get_queryset` from both views and add the mixin to both class signatures.

`class PostListMixin(PublishedPostsMixin, ListView)`.

We know that we have an unpublished blog post with an ID of 2. What happens if we put that in our URL? We get the 404 page again, as we should.

Add and commit everything new.

### Style

Everything is very bland at this point. Let's add some style to the site.

We need to create our `assets/` directory. In `settings/base.html`, we said this directory would be level with `blog/` and `microblog/` so be sure to create it there. Now we'll go to [http://getbootstrap.com](http://getbootstrap.com) and download Twitter Bootstrap. Unzip that file and move the contents to our `assets/` directory.

Re-open `_layouts/base.html` and in the `<head>` add a link to the stylesheet with `<link rel="stylesheet" href="{{ STATIC_URL }}css/bootstrap.min.css">`. Refresh the site and we'll see that we get a 404 on the styling. We need to add a `".."` to our `STATICFILES_DIRS` setting. Sometimes even the best of us get file paths wrong. We'll also add another set of `".."` to `MEDIA_ROOT` and `STATIC_ROOT`.

Refresh and you should have really basic and simple styling. In `_layouts/base.html`, let's update the template just a bit. Add a `<div class="container">` inside the `<body>` and wrapped around the `<h1>` and `page_content` block.

You can add `class="btn"` to the `<a>` tag in `index.html` to give it a bit more personality.

Add `bootstrap-responsive.css` as another `<link>` tag inside the `<head>` of `_layouts/base.html` to get a bit of responsiveness if you want. Nothing we've done so far is really responsive, though.

### Slugs & URLs

Since we're holding onto slugs in our models, it doesn't make much sense that we're not using them in our URLs. Open up `blog/urls.py` and change the `<pk>` to be `<slug>` and change the regex from `\d+` to `[\w-]+`, which will catch all letters, numbers, underscores, and hypens.

Now, inside of our `blog/post_list.html` template, instead of passing in the `pk`, we need to pass in the `slug`. So change `{% url 'blog:detail' post.pk %}` to `{% url 'blog:detail' post.slug %}`. Refresh, click on a post link, and you should see the slug in the URL.

That's great but I have to put in `post.slug` every time. How can I stop repeating myself? I can define a `get_absolute_url()` method for the Post model.

Back in `blog/models.py`, add a new method to the model class named `get_absolute_url`. It only takes `self` as an argument.

A lot of times you'll see this with just `return` and then a big string with a lot of argument-replacement. That works, but isn't very friendly to changing URL patterns. You'll also see people using the `reverse` method, which, again, works but is already handled for you by Django's `permalink` decorator, which we'll be using.

Our `get_absolute_url` is going to return a tuple consisting of three parts: a name, a tuple of positional arguments, and a dict of keyword arguments. Our name is `blog:detail`, we don't have any positional arguments so we'll include an empty tuple, and our only keyword argument is `slug`, which should be filled with `self.slug`. We also need to decorate this function with the `permalink` decorator which is inside the `models` namespace.

```python
@models.permalink
def get_absolute_url(self):
	return ("blog:detail", (), {"slug": self.slug})
```

This returns the named url `reverse`-d with the provided args and kwargs.

Now, in our templates, instead of `{% url 'blog:detail' post.slug %}`, we can just call the `get_absolute_url` method on our model instance with `{{ post.get_absolute_url }}`. This is less typing and more keeps with the mentality that all of this information is contained on our model. This also means we don't need to `{% load url from future %}` on pages that only use the `get_absolute_url` model method.

Everything is working now, pretty much as we want it to but that doesn't mean there aren't things we can improve.

For instance, in our views, we override `get_queryset()` and run this `filter()` on the queryset every time. We'd have to do this anywhere that we want to get just the live articles, too, not just here in the views. That reeks of us repeating ourselves. We should, and will, clean that up with a model manager.

At the top of our `models.py` file, we'll create a `PostManager` class which will extend `models.Manager`. Inside of it, we'll define a `live()` method that returns a filtered queryset for whatever model our manager is tied to. To do the tying, we need to put the manager into our model, and, since we want it to be the default manager, we attach it at the name `objects`. We need to be sure and include the parens for it, as well.

Back in our views, instead of `super()`-ing the queryset, we'll just return the `live` method off of our model manager. You could keep in the `super()` call from before if you want, in case you do other queryset manipulation through mixins or in a view. Since we won't for the purposes of this tutorial, we'll be more explicit.

If we go back to the blog section of our site, we see that everything is still working just as before. Exactly what we want it to do.

Let's pause and commit everything now before we forget. We need to be sure and add our `assets/` folder, too. If you're on a Mac, you'll want to add `.DS_Store` to your `.gitignore` file to keep it from being tracked. I think Windows has a `Thumbs.db` file that's similar and should also be ignored.

Before we push to Heroku, we need to add South to our `requirements.txt`. Running `pip freeze` will give you the list of all currently installed packages and you should copy and paste the line about South from there to your `requirements.txt` file. Then you need to commit that change, too.

Now we'll deploy to Heroku with `git push heroku master`.

If you check the feedback from Heroku, you'll probably see these traceback lines about running `collectstatic`. This is actually caused by our `STATIC_ROOT` and `MEDIA_ROOT` being outside of the VM that Heroku provides for us. There's also a URL included that, while helpful, isn't relevant to what we'll do in this guide. Feel free to read it later, though, for some very good advice about serving static media for Heroku apps.

Since the paths are wrong, we need to change them in our `settings/base.py`. In both `MEDIA_ROOT` and `STATIC_ROOT`, take out one set of `".."` and the accompanying comma. This'll put both directories safely within our VM where `collectstatic` and upload handlers can put files.

Now to test that this works. Commit the changes and push them up to Heroku again. The `collectstatic` step worked fine so everything is correct for now. So, load the admin and see… it's still unstyled.

Open up the `microblog/urls.py` file and we'll fix that part, too. At the bottom, we'll add this snippet:

```python
urlpatterns += patterns('',
    (r'^static/(.*)$', 'django.views.static.serve', {
        'document_root': settings.STATIC_ROOT
    }),
)
```

It appends a new set of `patterns` onto the `urlpatterns` object. This set of `patterns` only has one entry. It catches anything that starts with "static/", which is our `STATIC_URL`, and, using Django's `views.static.serve` function, serves the file. It starts looking for the file at the `document_root`, which is set to the `STATIC_ROOT` setting. We'll also need to import the `settings` object from `django.conf`.

Commit and push to Heroku.

This isn't a practice you'll want to use on **real**, customer-facing sites. Serving static files through Python is much slower than serving them through a web server like Nginx or Apache. For something small like this, however, something where you're learning and experimenting, it's fine.

When we refresh the site, we get a blank page. Our old friend, the 500 error. Let's run the server locally to try and find the source of the bug. When we refresh the page, we see that it's a problem with the URL we just added; there's an unknown specifier. Let's remove the `?P` from the URL and try it again. It works!

Now commit it and push it to Heroku again. Refresh the site and we get the admin with styling.

We'll create a blog post real quick, save it, go to the site and the blog and…hey, another 500.

Locally, our blog works fine (as we've been seeing all along). So why wouldn't it work on Heroku? Our logs don't show anything. I did some digging and found what it was.

Heroku will run your South migrations for it. It won't, however, run `syncdb` for you, and South needs to have a table created before it can work. So run `heroku run python manage.py syncdb` and then, since you don't need to do another deploy, `heroku run python manage.py migrate` to get the table created for our `blog` app. Now creating blog posts, and loading the pages for blog posts on the client-facing site, will work.

One other thing that Django provides that is pretty handy is this "View on site" button for models that have a `get_absolute_url` method defined. By default, though, the only existing site has "example.com" as its domain and, as you can see, that isn't going to work for us.

So go to "Sites" in the admin and update the record to hold onto your Heroku domain. If you just copy and paste the domain from your browser, you may get the "http://" portion of it. Take that out or your URL will be wrong. Fix the URL, go back to the post, and hit "View on site". It should load your post right away.

And, with that, we have everything deployed, live, and served completely from Heroku.

## SUMMARY ##

For this episode, we've covered:

* Setting up South
* Creating a new app
* Controlling that app's models with South
* Building a simple, logical model
* Adding that model to the Django admin and customizing the admin's display of it
* Creating views, both function-based and class-based, for displaying the model's records
* Tying those views to URLs in the site-wide urls
* Moving the URLs to an app-specific file and namespacing them for better organization
* Creating app- and view-specific templates and base templates for our site
* Using a model manager to save us from repeating ourselves for common actions against the database
* Serving static media through Django

## Homework ##

If you want to take this example further, there are a few things that would be easy and useful to do:

* Make the content of a Post be able to handle Markdown, reS  tructuredText, or HTML
* Build views and templates for viewing posts by author
* Build views and templates for viewing posts by date
* Move the static media to S3 or similar