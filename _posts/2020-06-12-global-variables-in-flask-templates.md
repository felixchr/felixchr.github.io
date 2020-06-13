---
title: Add global variables to Flask templates
categories: [tips, operations, automation]
tags: [operations, automation, tips, python, flask]
description: Add global variables to Flask templates
---

## My Requirement

I was trying to add some information regarding my code into my Flask app. Like **version** and **build** time. This is needed for all templates as a footer it should be processed once.

My version information is stored in **\_\_version\_\_.py** in this format:

{% highlight python %}
__title__ = 'fcapp'
__description__ = 'My app'
__url__ = 'https://felixchr.github.com/'
__version__='1.0.4'
__build__='0x5ee41d6d'
__author__ = 'Felix Cao'
__author_email__ = 'felix.cao@XXX.com'
__license__ = 'MIT'
{% endhighlight %}

## The Solution

I studied online to find out the resolution. And eventually I found it on [stackoverflow](https://stackoverflow.com/questions/43335931/global-variables-in-flask-templates) and here is my code here:

### \_\_init\_\_.py of my Flask app

I added code to read and load the variables immediately after app is initialized:

{% highlight python %}
app = Flask(__name__)
app.config.from_object('config')

@app.context_processor
def inject_stage_and_region():
    here = os.path.abspath(os.path.dirname(__file__))
    about = dict()
    with open(os.path.join(here, '__version__.py'), 'r') as f:
        exec(f.read(), about)
return {'about': about}
{% endhighlight %}

The return value is a **dict**. Part of this code is borrowed from Kenneth Reitz''s **requests**.

### The base.html of my main template

In my base.html now I can add those to this code:

{% highlight jinja2 %}
{% raw %}

  <div>
      <hr/>
      Version: {{about.__version__}}<br/>
      Build Time: {{about.__build__}}
  </div>
{% endraw %}
{% endhighlight %}

It works now!

[^global variables in flask templates]: https://stackoverflow.com/questions/43335931/global-variables-in-flask-templates
