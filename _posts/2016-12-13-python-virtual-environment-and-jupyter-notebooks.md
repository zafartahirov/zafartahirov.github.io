---
layout: post
title: "Setup: Python Virtual Environment and Jupyter Notebooks"
modified: 2016-12-13 14:22:21 -0800
tags: [setup,python,virtualenv,jupyter]
categories: [setup,python]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---

If you work in Python only occasionally, and don't need different versions of packages, isolated setups, never have problems with dependencies, or never work on 'dev' versions of python -- chances are you don't need this post, and you can stop reading here :)

This is more of a "I know what Python is, and I am ready to start with more advance stuff, like running it... or contributing to other people's codes, but I don't wanna screw up whatever I have on my machine, and how do I do that again?" type of thing

<!-- more -->
* Table of Contents
{:toc}

I am assuming you are using `Python 2.7`, because `~>3.3` has the `venv` module already, and doesn't require this setup. In `Python >3.3`, you can just do `pyvenv ~/.virtualenv/`, and it will create it for you. The following is for those of us who use Python 2.x, and refuse to switch :)

Also, all the setups are tested on MAC OS X Sierra

## Prerequisites
If you haven't already install `pip`, `easy_install`, `XCode`, etc! If you haven't done it already, stop reading!!! and get back to whatever you were doing before... Like building colorful towers from plastic blocks or something... Seriously, go away!

# How to install `virtualenv`

```bash
# Installing venv
pip install virtualenv
```

### Example -- Create a "my_project_env"
Usage the `virtualenv` is not very "people-friendly" (IMHO). For example, if you need to create a virtual environment, you do:

```bash
# Usage: Creating a virtual environment and starting to use it
cd some_project/
virtualenv my_project_env
source my_project_env/bin/activate
```

It creates all the necessary documents right in your project directory. I personally don't like it, because I want to keep all my environments in a fixed location. That brings us to the installation of the `Virtualenvwrapper`:

```bash
# Installing a wrapper for virtualenv
pip install virtualenvwrapper
# Assuming that you want to keep your env's in here:
cat << EOF >> ~/.bash_profile
## If the virtualenvwrapper is installed, make sure it is sourced
if [ -f `which virtualenvwrapper.sh` ]; then
    export WORKON_HOME=$HOME/.virtualenvs
    export PROJECT_HOME=$HOME/DevProjects
    export VIRTUALENVWRAPPER_SCRIPT=`which virtualenvwrapper.sh`
    source `which virtualenvwrapper_lazy.sh`
    # source `which virtualenvwrapper.sh`
fi
EOF
```

# Using Python Virtual Environments (with wrapper)

All the flags for the commands could be found with a `--help` flag. Commands you need to know:

| Command             | What it does
|:--------------------|:-------------------------
| `mkvirtualenv venv1`| Creates a new **clean** environment `venv1`
| `workon venv1`      | Switches to environment `venv`
| `deactivate`        | Switches out of the virtual environment
|===
{: rules="groups"}

Suppose you want to create a completely clean python environment with nothing installed in it. Let us call it "cleanenv". To create it, you just do in the terminal (I use `$>` to indicate command prompt):

{% highlight bash %}
$> which pip
/usr/local/bin/pip
$> mkvirtualenv cleanenv
(cleanenv) $> which pip
~/.virtualenvs/dev-test/bin/pip
(cleanenv) $> deactivate
$>
{% endhighlight %}

Notice that your command prompt changes to indicate that you are in a virtual environment. At this point, any actions you do with python related stuff will be relative to that environment. You can install new packages without them affecting the ones you have already installed.

## What if I want to use packages installed globally?

If you have installed some big packages, you might want to keep them global, while installing smaller packages, and packages that you are developing in the isolated environment. You can use `--system-site-packages` flag to show that you want to use the global packages.

{% highlight bash %}
$> pip install numpy
........
$> workon cleanenv
(cleanenv) $> python
>>> import numpy as np
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named numpy
>>>
KeyboardInterrupt
(cleanenv) $> deactivate
$> # Now create another virtual environment with global package access
$> mkvirtualenv --system-site-packages globalenv
$> workon globalenv
(globalenv) $> python
>>> import numpy as np
>>> print (np.__version__)
'1.11.2'
{% endhighlight %}

# Using Virtual Environments with Jupyter Notebooks

Assuming you have [Jupyter installed](http://jupyter.readthedocs.io/en/latest/install.html),
create a new file '~/.ipython/profile_default/startup/00-virtualenv.py', and paste the following Gist

{% gist zafartahirov/cc763e59f35b760061efce3adb60927e %}

If you switch to the environment you want and after that start your jupyter notebooks as you would normally do but, they will run in the virtual environment :)

{% highlight raw %}
$> workon myEnv
(myEnv) $> jupyter notebook
[I 16:44:36.979 NotebookApp] Serving notebooks from local directory: ~/GitHub/codes
[I 16:44:36.979 NotebookApp] 0 active kernels
[I 16:44:36.979 NotebookApp] The Jupyter Notebook is running at: http://localhost:8888/
[I 16:44:36.980 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
{% endhighlight %}
