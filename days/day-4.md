# Jobeet Day 4: The Controller and the View

## The MVC Architecture
For web development, the most common solution for organizing your code nowadays is the [MVC design pattern][1]. In short, the MVC design pattern defines a way to organize your code according to its nature. This pattern separates the code into **three layers**:

- The **Model** layer defines the business logic (the database belongs to this layer). You already know that Symfony stores all the classes and files related to the Model in the `src/Entity/` directory of your bundles.
- The **View** is what the user interacts with (a template engine is part of this layer). In Symfony, the View layer is mainly made of Twig templates. They are stored in various `templates/` directories as we will see later.
- The **Controller** is a piece of code that calls the Model to get some data that it passes to the View for rendering to the client. When we installed Symfony at the beginning of this tutorial, we saw that all requests are managed by front controller (`public/index.php`). This front controller delegate the real work to actions.

## The Layout
If you have a closer look at the mockups, you will notice that much of each page looks the same. You already know that code duplication is bad, whether we are talking about HTML or PHP code, so we need to find a way to prevent these common view elements from resulting in code duplication.

One way to solve the problem is to define a header and a footer and include them in each template. A better way is to use another design pattern to solve this problem: the [decorator design pattern][2]. The decorator design pattern resolves the problem the other way around: the template is decorated after the content is rendered by a global template, called a **layout**.

If you take a look in the `templates` folder, you will find there a `base.html.twig` template. That is the default layout that decorates our job pages right now. Open it and replace it’s content with the following:

```twig
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Jobeet - Your best job board{% endblock %}</title>

    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

    {% block stylesheets %}
        <link rel="stylesheet" href="{{ asset('css/main.css') }}" type="text/css" media="all" />
    {% endblock %}

    <link rel="shortcut icon" href="{{ asset('images/favicon.ico') }}" />
</head>
<body>
<div id="container">
    <div id="header">
        <div class="content">
            <h1><a href="{{ path('job_index') }}">
                    <img src="{{ asset('images/logo.jpg') }}" alt="Jobeet Job Board" />
                </a></h1>

            <div id="sub_header">
                <div class="post">
                    <h2>Ask for people</h2>
                    <div>
                        <a href="{{ path('job_index') }}">Post a Job</a>
                    </div>
                </div>

                <div class="search">
                    <h2>Ask for a job</h2>
                    <form action="" method="get">
                        <input type="text" name="keywords" id="search_keywords" />
                        <input type="submit" value="search" />
                        <div class="help">
                            Enter some keywords (city, country, position, ...)
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <div id="content">
        {% if app.session.flashbag.has('notice') %}
            {% for message in app.session.flashBag.get('notice') %}
                <div class="flash_notice">
                    {{ message }}
                </div>
            {% endfor %}
        {% endif %}

        {% if app.session.flashbag.has('error') %}
            {% for message in app.session.flashBag.get('error') %}
                <div class="flash_error">
                    {{ message }}
                </div>
            {% endfor %}
        {% endif %}

        <div class="content">
            {% block body %}
            {% endblock %}
        </div>
    </div>

    <div id="footer">
        <div class="content">
          <span class="symfony">
            <img src="{{ asset('images/jobeet-mini.png') }}" />
            powered by <a href="http://www.symfony.com/">
              <img src="{{ asset('images/symfony.gif') }}" alt="symfony framework" />
            </a>
          </span>
            <ul>
                <li><a href="">About Jobeet</a></li>
                <li class="feed"><a href="">Full feed</a></li>
                <li><a href="">Jobeet API</a></li>
                <li class="last"><a href="">Affiliates</a></li>
            </ul>
        </div>
    </div>
</div>

{% block javascripts %}{% endblock %}

</body>
</html>

```
## Twig Blocks
In Twig, the default Symfony template engine, you can define **blocks** as we did above. A twig block can have a default content (look at the title block for example) that can be replaced or extended in the child template as you will see in a moment.


## Additional information

## Next Steps

Continue this tutorial here: [Jobeet Day 5: The Routing](/days/day-5.md)

Previous post is available here: [Jobeet Day 3: The Data Model](/days/day-3.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[2]: https://en.wikipedia.org/wiki/Decorator_pattern
