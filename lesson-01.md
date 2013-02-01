# Prerequisites

Welcome to _Getting Started with Django_. Before we start, I want to go over the basic idea of the series and what I hope to give you through it.

First, this series is meant to take you from novice to competent. What do I mean by novice? You’ve done the Django tutorial, you understand the basics of Python, but you have little-to-no idea about how to go forward from here without making a ton of mistakes.

Secondly, this series is meant to show you current best practices. Libraries that are go-to solutions for myself and many other Django developers out there.

And, lastly, this series is meant to be a solid platform for you to build your own sites and applications in a testable, re-usable, and distributable manner. I don’t want you just assembling building blocks, but actively creating.

So, let’s get started. The first step is to go to [gettingstartedwithdjango.com](http://gettingstartedwithdjango.com), which you’re probably already at.

You’ll notice that there are a few requirements before you’re really ready for this series; let’s talk about them quickly:

* First, you’ve done most, if not all, of Zed Shaw’s excellent [_Learn Python the Hard Way_](http://learnpythonthehardway.org). This implies that you have a terminal and editor that you already like and trust. Learning an editor, the terminal for you computer, and a new language is **not** something to do while learning a framework. These videos will still be here when you get back, so go familiarize yourself with those tools and Python first.

* Second, you’ve worked through the [official Django tutorial](https://docs.djangoproject.com/en/1.4/intro/tutorial01/), the one where you build the Polls app. While it’s not necessarily a great “real world” application, it explores a large amount of Django and is a great basis to start this series on.

* Finally, you have both [Vagrant](http://vagrantup.com) and [VirtualBox](http://virtualbox.org) installed for your OS. They’re both free and easy to download and install.

Once you have all the requirements fulfilled and everything downloaded, go back to the _Getting Started with Django_ site and scroll to the bottom of the front page. There’s a big button there to download the Virtual Machine that we’re going to use. [Download that file](http://s3.amazonaws.com/GSWD/gswd-vagrant.zip).

Move the downloaded .zip file to wherever you want to have the VM set up on your machine. I’d suggest a folder with nothing else in it. For my example, I’ll be using the Sites folder in my user’s home folder.

Unzip the file and you’ll see several new files in the current directory.

The `Cheffile` and `Vagrantfile` files control [Chef](http://www.opscode.com/chef/) and Vagrant, respectively. If you want to learn more about either of these technologies, these two files, and the official docs, are a great place to start.

To sum up:

`Cheffile`
: orchestrates building the VM using the `cookbooks` and `site-cookbooks`. Both of these directories hold onto files describing how to build the virtual machine, installing things like PostgreSQL and Git.

`Vagrantfile`
: controls the settings of the VM, for things like forwarding ports.

We’ve forwarded one _Getting Started with Django_-specific port, VM port `8000` to host port `8888`. More on that in just a bit.

## Start the VM

Our first step of all, and the first one you’ll take at the beginning of every video, is to run `vagrant up`. This fires up the VM and, if there’s not a suspended state, builds the machine.

Depending on whether or not you have a `precise64` box already installed locally, this make take awhile. `Precise64` is the version of Ubuntu that we’re using for this entire series.

Once the box is downloaded and installed and  the new system is created, you’ll be ready to go.

While the download and build is going on, though, let’s use that time to get ready for later steps.

Head over to [heroku.com](http://heroku.com) and sign up for their service. The way that we’ll be using Heroku is completely free and shouldn’t cost you anything.

Click the sign-up link, enter your email address, and then check your inbox. You’ll get a confirmation email with a link to click and then you’ll enter a password.

Hopefully, by now, your VM is finished installing and you’re back to your usual command prompt.

Everything should be green. If it’s not, or you see errors in the list, check on the _Getting Started with Django_ forums or other places online.

After that, we’ll do `vagrant ssh` to SSH into the Vagrant VM. In the home folder, you’ll see a file named `postinstall.sh`. Run this with `sudo` to get some other libraries installed, things like git and ruby.

**NOTE:** If you're on Windows, you'll need to install some sort of SSH client on your system. Most people seem to prefer [PuTTY](http://www.putty.org). User Joe Golton added the following steps to get PuTTY and shared folders to work:

### PuTTY
1. Install PuTTY
2. Using PuTTYGen convert the %HOMEPATH%\.vagrant.d\insecure_private_key to .ppk
3. Set host to 127.0.0.1, Port to 2222 
4. Go into connection/SSH/auth and browse to `c:/Users/<your username>/.vagrant.d/<your generated key.ppk>`
5. Go back to main session screen, click on "Default Settings", then click "Save" so you won't have to repeat these steps
6. Click "Open" to start SSH.
7. Enter `vagrant` as username if prompted

If you don't see shared folders after running the `postinstall.sh` script, try the following:

### Shared Folders
1. After the `./postinstall.sh` step, exit SSH
2. Run `vagrant halt`
3. Run `vagrant up`

While that’s installing, let’s talk about what we’re going to build in this series.

## Brief GSWD Overview

We’re actually going to build two different projects.

The first, which we’ll cover in just an episode or two, will be a brief overview/re-imagining of the project from the first version of _Getting Started with Django_: a microblog. It’ll be fairly self-contained and meant to be used privately, not for public sign up.

The second item will be a lending library. The basic idea will be a site for you and your friends and family to use to list items you’re willing to share and to record who has borrowed your items. The items could be anything from video games to movies or books. We’ll be using some third party services through APIs and lots of handy Django packages. I’m expecting this project to take 8 or 9 episodes to cover.

## Last System-wide Requirements

The last thing we have to install before we start doing Django-specific work is a couple of Python development tools. `sudo apt-get install python-dev python-pip`. `Python-dev` gives us the Python development headers and `python-pip` installs the `pip` Python installer library, so we can install packages using `pip` instead of `easy_install` or doing `python setup.py` in downloaded packages.

Since we’ll be doing more than one project on this VM, and it’s generally a good idea to keep your code base corralled anyway, we also need to install `virtualenv`.

`Virtualenv`, if you don’t know, is a handy tool to create different, contained environments for Python packages. This way, each project has its own packages with their own versions, so you can have legacy projects using Django 1.2 and new projects with Django 1.4 or 1.5. While it’s not **required** for working with Python and Django, it’s very much recommended and is what we’ll be doing throughout the series.

So, to get `virtualenv` available everywhere, we’ll `sudo` its installation. `sudo pip install virtualenv` and give it a minute to install.

The folder where you ran `vagrant up` on your machine, which we’ll call the “host”, is symlinked into the VM at `/vagrant`, so we’ll switch to there for all of our real work.

If we list the contents, you’ll see everything you saw before, like the `Cheffile` and the `Vagrantfile`.

This is amazingly handy since it means you can edit the files on your VM from your host computer in whatever text editor you like.

We need to make a directory in here that’ll hold our project files, the ones that we’ll create. So I’m going to name mine “projects”.

## Virtualenv and packages

Before I go any further, though, I want to make the virtual environment for this project. Because of how Vagrant works, we can’t make it in the shared directory, but that’s OK. We can pass a path to `virtualenv` and it’ll create the virtual environment there. So we’ll pass in `~/blog-venv` to build the virtual environment in our home folder. That complete command is `virtualenv ~/blog-venv`.

Now to activate the virtual environment, so it’s where all of our packages are installed instead of the global environment, we need to “source” it. To do that, we run `source ~/blog-venv/bin/activate` and you’ll see that it changes our prompt to mention the virtual environment. Running `which python` after this gives us back a path to the Python executable in our environment.

Since we want to use Django for our project, we now need to install it. We’ll do `pip install django` and, assuming you’re watching this before Django 1.5 reaches stable, it’ll download and install Django 1.4.something.

**NOTE:** If Django 1.5 has been released when you follow this guide, it _should_ be safe to use, or you can do `pip install django==1.4.3` to match the version in the series.

Let’s start the project! We have a new tool available to us since we installed Django called `django-admin.py`. We’ll use that to start the project with `django-admin.py startproject microblog`. This’ll create a directory named `microblog` that we’ll go into.

Inside, there are two items: `manage.py` and another directory named `microblog`.

Inside `microblog`, you’ll find some standard Django files, `__init__.py`, `settings.py`, `urls.py`, and `wsgi.py`.

Back in the parent directory, let’s see if it all runs. `python manage.py runserver` would run the server, but, since we’re using a VM, we need to make sure it’s available to the host machine. So we’ll specific an address and port, too. We’ll use `0.0.0.0` so the service listens on all addresses, and the port `8000` since that’s the one that has been forwarded already. The complete command is `python manage.py runserver 0.0.0.0:8000`.

On the host machine, if we load [http://127.0.0.1:8888](http://127.0.0.1:8888) in a web browser, we get the “It worked!” page from our running Django server. This a great first step, but isn’t something we can share with the world&hellip;yet.

`Ctrl-C` will kill the running Django server.

## Getting to Heroku

We need to do a couple of more things. First, we need to install the Heroku Toolbelt. If you go to [https://toolbelt.heroku.com](https://toolbelt.heroku.com), you can download a version for whatever your OS is. Since we’re using Ubuntu, we need to copy and paste this shell command and it’ll be installed for us.

This’ll add some Heroku package repos to your Ubuntu apt list, and then install the Heroku client, Foreman, and, if we didn’t already have it, Git.

Heroku provides a couple of great guides, actually, for getting [Python](https://devcenter.heroku.com/articles/python) and [Django](https://devcenter.heroku.com/articles/django) up and running on Heroku. We’ll follow them, for the most part, for getting our project up, too.

First off, we need to run `heroku login` and give our credentials to our shell.

We already have Python installed and we’re not using Flask, so we can skip a decent amount of this. We _do_, however, need to use the `requirements.txt` file convention to get Django, and other packages later, installed on the server.

Running `pip freeze` will show us all of our currently installed packages. You see the Django that we installed, and its two dependencies, argparse and wsgiref. Now we’ll run `pip freeze > requirements.txt` to echo the contents of `pip freeze` into a text file named `requirements.txt`.

Heroku wants a file with this name for a couple of reasons. First, it seems to use this name to identify a Python project. Secondly, it uses it to install your needed packages.

If we print out the contents of the file, we’ll see that it’s the same as what we had before.

We now need one other magical Heroku file, one named `Procfile`. This file controls our Heroku processes and, for now, only has one entry. The process will be named `web`, and it’ll run almost exactly the same command we ran to test this locally. `python manage.py runserver 0.0.0.0:$PORT --noreload`. The `$PORT` will be set automatically by Heroku. 

We also want to have a `.gitignore` file so that Git doesn’t include everything in the directory. Create the file, then add `*.pyc`, save the file, then let’s create the Git repo.

`git init` will create the repo, `git add .` will add the current directory and its contents, and if we do `git status`, we’ll see that it did not include any `.pyc` files, even though we have some.

That’s what we want, so we’ll go ahead and make a commit (`git commit -am "initial commit"` or the like). If this is the first time that you’ve run git on this VM, it’ll ask for your name and email address. Add these in so it can tag all of your commits.

If we do a `git status`, we’ll see that everything has been committed, and `git log` will show our single, first commit. Now we need to push everything to Heroku. Running `heroku create` will create an app for us and set up a Git remote. Your app will get a generic, random name. Mine is named `cryptic-beyond-3326`. If we, at this point, try to do a `git push heroku master` to send our master branch to Heroku, we’ll get a permission denied error about our public key.

### SSH keys for fun and profit

**NOTE:** Some people have reported that doing the `heroku create` created their SSH keys for them. If that's the case for you, skip the generation and uploading steps that are below.

We need to generate a key and give it to Heroku. To generate the key, we need to run `ssh-keygen`. You can change the location of the key if you want. Enter your passphrase twice, and then we’ll get the fingerprint and a success message. Now you need to send the key to Heroku. Since I can never remember how to do that, we’ll check `heroku —help`.

Turns out that we submit our keys with `heroku keys:add ~/.ssh/id_rsa.pub` (assuming you didn't give it a different location). Now that the key is added, if we do `git push heroku master` again, it’ll go through.

We get a bunch of feedback from Heroku showing us what’s going on. It detects that it’s a Python or Django app, creates a virtual environment just like we did, then installs the dependencies that we listed in `requirements.txt`.

Then it launches the site. Before we check out the site, let’s talk about what happened.

### What Heroku does every time you push to them is:

* Creates a new app space with its own virtual machine, virtual environment, and packages installed through `pip`.
* Swaps the subdomain over to the new location.
* Deletes the old app space.

What this means is that anything added to the app that is *not* in Git, won’t last between pushes. Even more, this exact same process happens once every 24 hours anyway, so uploaded files have to go somewhere other than Heroku.

We can’t run `heroku app open` on the VM since it can’t send a URL to our host machine. Instead we’ll go to heroku.com, go into our control panel, and we’ll see our app listed there.

After we click on our app, we get a pretty good overview of what it’s doing. Right now, it’s running the one process we specified, “web”, and it’s running the command we gave it.

Running 1 dyno gets you enough free processor hours for an entire month, each month. Since we’re just doing applications for ourselves, we won’t be spawning more processes and don’t have to worry about going over on processor time.

We also have a PostgreSQL database waiting for us that we’ll get into later.

If we click "open application" it’ll open our app and we’ll see our old friend, the “It worked” Django default page.

If you want to do some more reading about this, feel free to check out the two Heroku guides.

## Diving into Django

We’ve launched our site both locally and on Heroku, but there’s nothing really *to* our site yet. Before we start actual development, though, we should really learn more about Django and how it structures and controls a project.

In our project directory, we have the `manage.py`, `.gitignore`, and `Procfile` files. The `.gitignore` and `Procfile` files are specific to this project as a whole; they’re not specific to the Python or Django portions of the project. The `Procfile` is even more specifically just for Heroku.

The `requirements.txt` file is specific to this project, too, in that it controls which libraries and which versions of those libraries are installed.

So what did Django give us? Honestly, not a whole lot at this point. It gave us the `manage.py` file, which lets us run management commands.

There’s also the `microblog` directory that Django created for us when we ran `startproject`.

Inside `microblog` is the somewhat-generic meat of our project. While in here, or anywhere else, really, you can ignore the `.pyc` files when you’re browsing. They are just the compiled bytecode that Python generates and aren’t something you’ll want to edit.

The `microblog/__init__.py` file in here, and in most Python modules that you’ll deal with in Django, is empty.

The `microblog/settings.py` file, which we’ll be dealing with soon, controls a lot of Django. As opposed to many frameworks, it’s not just a text file but an actual Python module, so you can execute code in it. This is *very* handy and something we’ll be taking advantage of later.

The `microblog/urls.py` file is our root URL configuration file. It’ll control the top-level of routing for our entire project.

And, finally, the `microblog/wsgi.py` file is a kickstart for hosting your site with a WSGI-compliant server like gunicorn or uwsgi. We’ll use this later when we deploy a more solid `Procfile` to Heroku.

### Settings

Having just a single `settings.py` file is a bit limiting, especially for development. A lot of Django tutorials and articles will suggest having a `local_settings.py` which you import everything from at the bottom of your `settings.py`. This works, but isn’t really clean and makes it difficult to extend `settings.py` items in your local settings.

So we create a `microblog/settings` directory that’ll hold all of our settings modules with better names and more flexibility.

Inside there we want to create an `__init__.py` file that’ll let us import settings as a module. Contrary to what I said just a minute ago, we *will* be editing this init file. And, as the last step of preparation for this, we’ll move the existing `settings.py` into the settings directory as `base.py`, since it’s our starting base.

Now open up the `settings/__init__.py` and `settings/base.py` files in your text editor and let’s start working on them.

In the `__init__.py`, the first thing I want to do is import everything from base.py. So we’ll add `from settings.base import *`. Then we want to `try` and import everything from our local settings with `from settings.local import *`. If that triggers an `ImportError` exception, we’ll catch it and just `pass` on through.

Most of the time, you don’t want to use `import *`. It clobbers the namespace and just makes a mess of things. In this case, though, where we know we want everything and there’s not really anything to mess up, it’s OK to do.

So now for setting the default settings to something sane for our work.

At the top is the `DEBUG` setting. This controls whether the project runs in DEBUG mode, which triggers things like auto-reloading when it detects changes to Python files and showing that nice debug screen when something goes wrong. That screen is more finely controlled by this second setting, `TEMPLATE_DEBUG`, which actually turns on that screen, but requires DEBUG to be true as well.

Since our base settings are what’ll be run on the server, not locally, we want to change `DEBUG` to be `False`.

Below that is the `ADMINS` tuple. This controls the names and email addresses that are considered to be admins capable of dealing with site problems. They’ll get an email every time your site throws a 500. Since I want to be notified when something goes wrong, I need to add myself to the list.

Below that is the `MANAGERS` setting which is used whenever the `mail_managers()` method is called. By default this is set to be the same as the `ADMINS` tuple and I’ve never had a reason to change it.

Next, comes the `DATABASES` dict. We’re not going to edit this for now since we’ll be using Heroku and there’s a shortcut for them. You’ll notice, however, that there are several database backends supported and you can add more by installing third-party packages. **I’d suggest you not use the SQLite backend for development, though.**

The `TIME_ZONE` setting controls how Django manipulates times in the database. You’ll want to set this to whatever timezone you want your site running in. I usually change it to `America/Los_Angeles` since I’m on the west coast, but we’ll leave it alone for now.

The `LANGUAGE_CODE` setting controls the language that your site sends for headers and what you’re promising your content is in. Since I’m an English speaker, I’ll leave it alone.

`SITE_ID` is a setting I don’t think I’ve ever had to change. The setting is used if you’re running multiple Django installations off of one code base. Each one would get its own settings file with a different `SITE_ID` so you could allocate content in the database to a specific site.

`USE_I18N` is for controlling whether or not you want Django to take internationalization of content into consideration. If you’re only going to have content, both user-generated and built into your templates and Python files, in one language, change this to false. Since we’ll be using the `{% trans %}` tag and other translation utilities, we’ll leave it set to true.

`USE_L10N` is similar to `USE_I18N` but it’s for localizing content like dates, commas and decimals in numbers, and calendar formats. While we may not take direct advantage of this, we’ll leave it on.

And `USE_TZ` controls whether or not Django records UTC timestamps to the database or localized ones. Leave this set to `True` and you can manipulate datetimes so they always appear local to a user.

Now starts the area that I find confuses a lot of newcomers. The various roots, urls, and dirs that Django needs for finding files in the filesystem and serving them as URLs.

`MEDIA_ROOT` is where Django should store uploaded files. This includes files from your users and files you upload yourself through the Django admin.

`MEDIA_URL` is the URL that these uploaded files should be available at. This URL should end with a slash.

We’re going to leave both of these alone for now.

`STATIC_ROOT` is where Django should collect all of the static assets in your project, which means both the files you’ve added as part of your design, and files which come with third-party libraries like Django’s admin.

`STATIC_URL` is, as I’m sure you can guess, where these files should be served from. We’ll leave it as its default of `/static/` for now. Again, this URL, if you change it, needs to end with a slash.

`STATICFILES_DIRS` is where it starts to get more fun. This is a tuple of directories, **outside** of app directories, where Django can find asset files. Again, these are your site design files, like CSS and Javascript. Usually you’ll only have one of these but if, perhaps, you were running a Django site alongside an existing site built in another language or framework, you could include that site’s static files area and Django would have access to them.

`STATICFILES_FINDERS` controls how Django looks for static files (not _where_, that’s controlled by the setting above this one). By default, it looks on the file system where you’ve specified files to be and then in a directory named `static` inside of each app’s directory.

The `SECRET_KEY` setting is a key that’s used for a seed in secret-key hashing algorithms. You want this to be unique and, obviously, secret.

`TEMPLATE_LOADERS` works like `STATICFILES_FINDERS` but for finding templates instead of asset files.

`MIDDLEWARE_CLASSES` is the tuple of classes that act as middleware for your project. If you’ve never heard of middleware, the premise is fairly simple. These classes run both before and after your view functions so they are in the middle of the entire cycle. When they are run before the view, on a **request**, they are executed from top to bottom. When they run after the view, on a **response**, they run from bottom to top. Since it’s generally a good idea to try and prevent clickjacking, we’ll go ahead and enable this last middleware.

`ROOT_URLCONF` just tells Django which module to use _first_ when it starts building URL patterns. It’s already set to the `urls.py` file inside of our `microblog` folder so we’ll leave it alone.

`WSGI_APPLICATION` sets which WSGI module we want to run for the dev server. Luckily the one that Django provides will work for both local development and on the final server and has already been set.

`TEMPLATE_DIRS` should really be specified higher up in the settings file with the other filesystem-related settings. It works just like `STATICFILES_DIRS` in telling Django where to look for templates that’s not inside of an app. For how I develop most projects, the directory or directories that you list in here are where someone can expect to find your site-wide templates, your generic layouts and templates that don’t relate to the functionality of a specific app.

`INSTALLED_APPS` is a tuple of all the apps that Django runs as an app. These will all include a models.py file, since that’s the magic that makes a Python module into a Django app. I, however, don’t like having one big tuple of apps. So we’ll rename `INSTALLED_APPS` to `DJANGO_APPS`, since these are the apps provided by Django. Then I add two new, empty tuples. The first is named `THIRD_PARTY_APPS` and the second is `LOCAL_APPS`. Then we bring back `INSTALLED_APPS` and set it equal to `DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS`. This will help us with testing later, too, as we can tell our test runner to only test the apps listed in `LOCAL_APPS` and not to test other people’s apps or the apps provided by Django.

Since we want to have access to the Django admin, we need to uncomment this line in `DJANGO_APPS`. We won’t be using the `admindocs` app, though, so we’ll remove the lines about it.

The last set of settings in the file is the `LOGGING` dict. This is where you can specify loggers for use throughout your project. For now, though, we’ll leave it alone. You can see, though, that by default it’s set up to email the admins when it logs an error.

### File paths in Django Settings

All of the places where we mentioned that Django would be looking for files on your file system, Django is expecting absolute paths from the root of your system.

Obviously that would be a nightmare to set up on multiple machines for servers and other developers.

So I have a small snippet that I use almost everywhere. I’ve put a copy of it on [snipt.net](https://snipt.net/kennethlove/django-absolute-paths-for-settingspy/) so you can copy and paste it into your project.

At the top of the settings file, we want to import the `os` module.

The `here` lambda just gives us a chainable function for building file paths from the location of the file it’s being run from. In our case, that’s our `base` settings file.

And so we set `PROJECT_ROOT` to be the directory above the settings file.

Lastly, the `root` lambda gives us another chainable function that builds file paths from the project root.

I want `MEDIA_ROOT` to be one directory above where the `PROJECT_ROOT` is, up where things like `manage.py` are, and then a directory named `uploads`. Django will create this directory for me if it doesn’t already exist.

The `STATIC_ROOT` gets the same treatment, but we’ll call the directory `static` instead of `uploads`. Again, Django will create this one for us.

For `STATICFILES_DIRS`, we don’t want to go all the way up to where the public files live. We’ll name this directory `assets`. Django will **not** create this one, that’s up to us to do when we need it.

`TEMPLATE_DIRS` gets the same content as `STATICFILES_DIRS` but using the directory name `templates`. This is another one that we’ll have to create for ourselves.

So let’s go up a directory in the terminal and restart our app.

## Mistake #1!

And it throws an `ImportError`! The error comes from how I’m importing the settings. I don’t need to include the module name, I just need to **do the imports as relative imports**. So I’ll change them from `settings.base` and `settings.local` to just `.base` and `.local`.

Now, try to run the server again. This time it starts correctly. If we try to load the site, we actually get an error again.

We don’t get the pretty debug page any more because we’re running as `DEBUG = False`. And you can see that the error thrown is saying that we don’t have the 500.html template.

Let’s start on creating local settings. Back in `microblog/settings/`, we need to create `local.py`. And then we can pull it up in our text editor.

I’ll add in `DEBUG = True` so the project will run in debug mode again. I’ll also add in `TEMPLATE_DEBUG` so we get the pretty error page again.

If I restart the project and reload the page, I get the “It worked!” screen again.

## Database

Since I’m using the Vagrant VM, there’s already a database server set up. We need to set up a role, however, for the vagrant user so we don’t have to connect to PostgreSQL as a different user.

**NOTE:** Some users have reported that the vagrant role already exist in their VM. If that's the case for you, you have two choices.

1. You can remove the role with `dropuser vagrant` as the postgres user, and recreate the role with the following instructions.
2. Or you can just use the vagrant role that already exists. Its password _should_ be "gswd" without the quotes.

So, `sudo su postgres` will get you into the postgres user’s account. Next we’ll run `createuser` and we’ll include the `-P` flag because we want to set a password for the user. We’ll name the role the same as our standard user, which is `vagrant` and then put in whatever password we want. At the end, we do want the user to be a superuser so they can create new databases and users and whatever else we may need.

To try and test that this works, we’ll connect through `psql`. `psql -U vagrant template1`. This doesn’t work, though, since, by default, Postgres uses peer authentication. So exit back to the standard user and try to connect again.

This time, though, we can just do `psql template1` (or run the same command as before, to be explicit) to connect to the `template1` database and it’ll automatically use our user.

Since it lets us in, it must work. Now we need to create a database for our project. I don’t like to be too clever when naming things, so I’ll name the database `microblog`. Running `createdb` this sparsely automatically sets our user as the owner of the database.

So back in `microblog/settings/base.py`, we need to set the database settings in the `DATABASES` dict. Since we’re using PostgreSQL, we’ll set that as the engine, set the name to `microblog`, and leave the rest alone. I tend to set the user anyway, though, just to be a little more explicit.

We need to move this to our `local.py`, though, since that’s where these settings are actually correct. Which brings up the question: _“what do we do for Heroku?”_

**Note:** Before we go any further, be sure to add `local.py` to your `.gitignore`. The ignore file should now look like:

```
*.pyc
microblog/settings/local.py
```

You'll also want to be sure to add all the current files to git by doing `git add .` again.

Heroku suggests using the [`dj-database-url`](http://pypi.python.org/pypi/dj-database-url/0.2.0) Python package so I’m going to go ahead and install that with `pip`. Then, as illustrated by Heroku, we’ll add `import dj_database_url` into our base settings file, and set the `DATABASES` dict’s default key to be `dj_database_url.config()`. If you actually follow the Heroku docs exactly, you’ll get an error about the `DATABASES` dict not existing already.

If we run the project again, I get told that I forgot to install the database driver. To fix that, `pip install psycopg2`.

Back over on Heroku, we’ll log in like before and go look at our app. As you’ll see, it’s already set up with the Heroku Postgres Dev addon, which provides us with a small PostgreSQL database to use on Heroku.

Before we can deploy anything, we need to make sure that our `requirements.txt` is up to date. Running `pip freeze` will give us the current list of installed packages. There are two new packages listed, `dj-database-url` and `psycopg2`. We need to add both of these to the `requirements` file.

After we add and commit everything, we can push to Heroku and then we can open our application.

Our application has an error, but at least we know it’s running. We’ll solve the problems with the server as we wrap up this lesson.

## Syncing up the DB

I set up a database and added it into the settings, but I didn’t do anything with it yet. I need to get the default tables into the database. Thankfully, Django provides the `syncdb` command.

Run `python manage.py syncdb` and, using the database that we specified, it’ll connect and start creating tables. And then it’ll ask if I want to create a superuser. Since I’ll be using the Django admin, I definitely want to create one.

Fill in the username, email, and password prompts and everything should be created successfully. If I run the server again, the local deployment should work fine but I haven’t enabled the admin yet so I can’t test the superuser I just made.

The next file we need to edit is `microblog/urls.py`.

First I’ll uncomment the import line and the ``autodiscover` line. The `autodiscover` line is what makes it possible for Django to introspect your apps, find the `admin.py` file in them, and then add it to the existing admin urls.

Second, in the actual url patterns, I’ll remove the lines about the `admindocs`, and then uncomment the route for the `admin`.

Since we’re running in `DEBUG = True` mode, the server reloads every time we hit save on a Python file, so I don’t need to restart it this time. If I load [`/admin/`](http://127.0.0.1:8888/admin/) in my browser, I get a very barebones login screen. I can log in with my credentials that I set on the command line and I’ll get an, again, very barebones admin.

The static files are missing for the admin and, if you look in the terminal, you’ll see that we got an error about that. Specifically, the error is about the fact that we left out the comma after our tuple item. That’s easily fixed back in `microblog/settings/base.py`.

Add a comma after both of the `root()` entries in `TEMPLATE_DIRS` and `STATICFILES_DIRS`. Save and refresh and now our admin looks like it should.

So now let’s get that same thing running on Heroku.

Since we’ve changed a couple of files, we need to commit the changes and push them to Heroku.

If we reopen our application on Heroku, we still get the 500 error, but let’s try the admin there. That’s broken, too. Now it’s time to debug Heroku.

## Debugging

Running `heroku logs` gives us the logs from our Heroku instance and, as we see, it’s complaining about the 500.html file being missing.

Before we go to far into this template problem, which might be a bit of a red herring, let’s get everything else set up on Heroku.

The first thing we need to do is sync the database. We do that with `heroku run python manage.py syncdb`. The command is exactly the same as it was locally, but with `heroku run` at the beginning. You remember me mentioning that your app space on Heroku was reset every time you pushed, and that’s still true, but your database doesn’t get reset like that. So anything you do in a `heroku run`, or in your actual app, that affects the database will persist.

Give details for the user account again, just like you did before, and let everything finish up.

To start debugging our previous problem, let’s make the two environments match as much as possible by changing our local settings to run the site as `DEBUG = False`. Ironically, our local set up still works.

As another possibility, let’s go into the `microblog` directory and create a `templates` directory. And then, in `templates`, we’ll create a blank file named `500.html`.

Run the server again and hit the homepage. It’s blank because it’s serving the `500.html`. This points to a likely culprit, which is that we don’t have a catch-all URL route. Let’s make one.

In `microblog/urls.py`, we need to add our URL.

I do all of my views as class-based views (don’t worry, we’ll cover function-based views too) and that’s the recommendation from Django, now, for generic views. Since we want a view that just shows a template, this is a great use of the TemplateView CBV.

First things first, we need to import the view class: `from django.views.generic import TemplateView`. Then we create our top, catch-all URL. Our default URL catches an empty string, then calls `TemplateView`’s `as_view()` method which we’ll pass the `template_name` argument to it, which we’ll set as `index.html`.

We won’t name this URL for now because we’ll be replacing it before going much further.

Also, I want to point out, this is only acceptable, to me and many other Django developers, when quickly building something as a stopgap. You should **not** mix view logic with route logic and even something as simple as a `TemplateView` belongs in a views file.

The view actually needs its arguments as keyword arguments, not as as dict. If we fix this, we still get our blank 500.html template because the index.html template doesn’t exist. Let’s go create that now.

Touch `microblog/templates/index.html` and then open it in your text editor. Since this is a stopgap, we’ll just put in an `<h1>` with “Hello World”. Save it, re-run the server, refresh, and you’ll get the template.

Add the template files, commit everything, and push it to Heroku.

Then go refresh the Heroku app and we get the “Hello World”. If we try to go to the admin, we get the admin but it’s missing it’s media. That’s fine, we’ll be putting our static media somewhere more appropriate soon anyway.

## Conclusion

That wraps up our first video and our first lesson. We’ll spend one more lesson on basic Heroku setup and the microblog app and then move on to the more complex lending circle app.

Today, we covered:

* Setting up and SSHing a Vagrant VM
* Installing `pip` and `virtualenv`
* Creating and activating a virtual environment
* Setting up our system for publishing to Heroku
* Created a Django project
* Configured a Django project with flexible settings
* Synced databases both locally and on Heroku
* Debugged issues and got everything running both places

Thanks for watching!