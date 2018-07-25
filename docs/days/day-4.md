# Jobeet Day 4: The Controller and the View

Today, we are going to create the basic job controller. It will has the code we need for Jobeet:
- A page to list all jobs
- A page to view job details

## The MVC Architecture
For web development, the most common solution for organizing your code nowadays is the [MVC design pattern][1]. In short, the MVC design pattern defines a way to organize your code according to its nature. This pattern separates the code into **three layers**:

- The **Model** layer defines the business logic (the database belongs to this layer). You already know that Symfony stores all the classes and files related to the Model in the `src/Entity/` directory of your bundles.
- The **View** is what the user interacts with (a template engine is part of this layer). In Symfony, the View layer is mainly made of Twig templates. They are stored in various `templates/` directories as we will see later.
- The **Controller** is a piece of code that calls the Model to get some data that it passes to the View for rendering to the client. When we installed Symfony at the beginning of this tutorial, we saw that all requests are managed by front controller (`public/index.php`). This front controller delegate the real work to actions (class methods).

## Controller

Let’s create our first controller. Create file `JobController.php` in `src/Controller` folder and put there next code:
```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

/**
 * @Route("job")
 */
class JobController extends AbstractController
{

}
```

For now it has no actions, but not for long.

## The Layout
If you have a closer look at the mockups, you will notice that most pages look the same. You already know that code duplication is bad, whether we are talking about HTML or PHP code, so we need to find a way to prevent these common view elements from resulting in code duplication.

One way to solve the problem is to define a header and a footer and include them in each template. A better way is to use another design pattern to solve this problem: the [decorator design pattern][2].
The decorator design pattern resolves the problem the other way around: the template is decorated after the content is rendered by a global template, called a **layout**.

If you take a look in the `templates` folder, you will find there `base.html.twig` template. That is the default layout that decorates our job pages right now. Open it and replace it’s content with the following:

```twig
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Jobeet - Your best job board{% endblock %}</title>

    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>

    {% block stylesheets %}{% endblock %}

    {% block javascripts %}{% endblock %}

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
</head>
<body>
<nav class="navbar navbar-default">
    <div class="container-fluid">
        <div class="navbar-header">
            <a class="navbar-brand" href="#">Jobeet</a>
        </div>

        <div class="collapse navbar-collapse">
            <ul class="nav navbar-nav navbar-right">
                <li>
                    <div>
                        <a href="#" class="btn btn-default navbar-btn">Post a Job</a>
                    </div>
                </li>
            </ul>
        </div>
    </div>
</nav>

<div class="container">
    {% block body %}{% endblock %}
</div>
</body>
</html>
```

## The style sheets

As this tutorial is not about web design, we will use [bootstrap][3].
Bootstrap is one of the most popular frontend library. It provides styles for frequently used elements, such as: grid, tables, forms, buttons and etc.
We already connected it from [CDN][4] in layout from previous step:
```twig
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
```

## The Job Homepage Action

Each action is represented by a method of a class. For the jobs list, the class is `JobController` and the method will be `list()`. Let’s create this action:

```php
use App\Entity\Job;
use Symfony\Component\HttpFoundation\Response;

class JobController extends AbstractController
{
    /**
     * Lists all job entities.
     *
     * @Route("/", name="job.list")
     *
     * @return Response
     */
    public function list() : Response
    {
        $jobs = $this->getDoctrine()->getRepository(Job::class)->findAll();

        return $this->render('job/list.html.twig', [
            'jobs' => $jobs,
        ]);
    }
}
```

Let’s have a closer look at the code: the `list()` method gets the `Doctrine` object, which is responsible for working with database. `Doctrine` object is able to retrieve `repository` object that has lots of built-in methods to query database. At last `Repository` will retrieve all the jobs in the form of `ArrayCollection` of Job objects that are passed to the template (the View).

Probably you have already noticed line starting with `@Route` in comment block above method. This line is not a simple comment. This line is related to the[Routing component][6].
It tells Symfony which URL path is related to which action in controller. We will learn more about that in next lesson.

## The Job Homepage Template

In `list` we passed jobs to `job/list.html.twig` but we don’t have this file yet. Let’s create it in `templates/job` folder:

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <table class="table text-center">
        <tr>
            <th class="active text-center">City</th>
            <th class="active text-center">Position</th>
            <th class="active text-center">Company</th>
        </tr>

        {% for job in jobs %}
            <tr>
                <td>{{ job.location }}</td>
                <td>
                    <a href="#">
                        {{ job.position }}
                    </a>
                </td>
                <td>{{ job.company }}</td>
            </tr>
        {% endfor %}
    </table>
{% endblock %}
```

## Twig Blocks
In Twig, the default Symfony template engine, you can define **blocks** as we did above.
A twig block can have a default content (look at the title block for example) that can be replaced or extended in the child template as you will see in a moment.

`extends` tag used in `list.html.twig` means that this template extends base template `base.html.twig` and can redefine blocks from it. In our case we define content for `body` block.

## The Job Page Action

We have action to list all jobs, now let’s create action to see one job:
```php
class JobController extends AbstractController
{
    ...

    /**
     * Finds and displays a job entity.
     *
     * @Route("/{id}", name="job.show")
     *
     * @param Job $job
     *
     * @return Response
     */
    public function show(Job $job) : Response
    {
        return $this->render('job/show.html.twig', [
            'job' => $job,
        ]);
    }
}
```
Here you may notice a bit of a Symfony’s magic that happens behind the scenes - there is a type hinted $job parameter in the method signature, but how Symfony loads that Job object you may ask yourself? It uses `id` from the URL, automatically queries table by this `id` and returns fully fledged Job object for you. Nice!

## The Job Page Template

Now let’s create `show.html.twig` file in `templates/job` folder:

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Job</h1>

    <div class="media" style="margin-top: 60px;">
        <div class="media-body">
            <div class="row">
                <div class="col-sm-10">
                    <h3 class="media-heading"><strong>{{ job.company }}</strong> <i>({{ job.location }})</i></h3>
                </div>

                <div class="col-sm-2">
                    <i class="pull-right">posted on {{ job.createdat|date('m/d/Y') }}</i>
                </div>
            </div>

            <p>
                <strong>{{ job.position }}</strong>
                <small> - <i>{{ job.type }}</i></small>
            </p>

            <p>{{ job.description|nl2br }}</p>

            <p style="margin-top: 40px;">
                <strong>How to apply?</strong>
            </p>

            <p>{{ job.howToApply }}</p>

            <div class="row">
                <div class="col-sm-12 text-right">
                    <a class="btn btn-default" href="#">
                        <span class="glyphicon glyphicon-menu-left" aria-hidden="true"></span>
                        Back to list
                    </a>

                    <a class="btn btn-primary" href="#">
                        <span class="glyphicon glyphicon-pencil" aria-hidden="true"></span>
                        Edit
                    </a>
                </div>
            </div>
        </div>
    </div>
{% endblock %}
```

Twig comes with a long list of [filters][7] and [functions][8] that are available by default. In this template we used [date][9]. This function is used to format a date.

You can find the code from day 4 here: [https://github.com/gregurco/jobeet/tree/day4][10].

## Additional information
- [Twig Official Documentation][5]

## Next Steps

Continue this tutorial here: [Jobeet Day 5: The Routing](day-5.md)

Previous post is available here: [Jobeet Day 3: The Data Model](day-3.md)

Main page is available here: [Symfony 4.1 Jobeet Tutorial](../index.md)

[1]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[2]: https://en.wikipedia.org/wiki/Decorator_pattern
[3]: https://getbootstrap.com/docs/3.3/
[4]: https://en.wikipedia.org/wiki/Content_delivery_network
[5]: https://twig.symfony.com/doc/2.x/
[6]: https://symfony.com/doc/4.1/routing.html
[7]: https://twig.symfony.com/doc/2.x/filters/index.html
[8]: https://twig.symfony.com/doc/2.x/functions/index.html
[9]: https://twig.symfony.com/doc/2.x/filters/date.html
[10]: https://github.com/gregurco/jobeet/tree/day4
