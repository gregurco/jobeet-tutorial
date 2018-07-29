# Jobeet Day 5: The Routing

## First links

In the previous lesson we created two actions with two templates but did not link them. It’s impossible to move from list page to view job description page.
Let’s do that! Let’s edit `templates/job/list.html.twig`:

```diff
- <a href="#">
+ <a href="{{ path('job.show', {id: job.id}) }}">
```

in `templates/job/show.html.twig`:

```diff
- <a class="btn btn-default" href="#">
+ <a class="btn btn-default" href="{{ path('job.list') }}">
      <span class="glyphicon glyphicon-menu-left" aria-hidden="true"></span>
      Back to list
  </a>
```

and also in `templates/base.html.twig`:
```diff
- <a class="navbar-brand" href="#">Jobeet</a>
+ <a class="navbar-brand" href="{{ path('job.list') }}">Jobeet</a>
```

Now it’s possible to view job description and go back to the job list page.

## URLs

If you click on a job on the Jobeet homepage, the URL looks like this: `/job/1`. 
How does Symfony make it work? How does Symfony determine the action to call based on this URL?
Why is the job retrieved with the `$job` parameter in the action? Here, we will answer all these questions. 

Symfony uses the `path` template helper function to generate the URL for the job which has the ID 1.
The `job.show` is the name of the route used, defined in the configuration as you will see below.

## Routing Configuration

In Symfony, routing configuration is usually done in the `config/routes/annotations.yaml`. Let’s take a look at the default configuration:

```yaml
controllers:
    resource: ../../src/Controller/
    type: annotation
```

It imports routing configuration from all controllers in folder `src/Controller` *(and also subfolders)* and which are written in annotations.

In the `JobController` you can find all routes related to jobs *(list all, show one, etc.)*:
```php
/**
 * @Route("job")
 */
class JobController extends AbstractController
{
    /**
     * Lists all job entities.
     *
     * @Route("/", name="job.list")
     */
    public function list() : Response
    ...

    /**
     * Finds and displays a job entity.
     *
     * @Route("/{id}", name="job.show")
     */
    public function show(Job $job) : Response
    ...
}
```

First we see class annotation `@Route("job")` - it prepends `job` to all the routes defined in the class.

Thanks to these two routes in controller:  

- If user goes to `/job`, the first route is marched and `list()` method is executed.
- If user goes to `/job/*` *(\* - means any character but at least one)*, the second route is matched and `show()` method is executed. *(Example: `/job/1`, `/job/2`, etc.)*  

Let’s have a closer look at the second route. For this route we can not provide exact path, because ID field is generated and for job with ID 1 the route is `/job/1`, for the next one the route is `/job/2` and so on. 
For this purpose Routing Component provides us possibility to define **variables** and it is written in the following way: `{variableName}`. In our case it’s `id` variable in route `/job/{id}`.  

For the URL `/job/1`, the ID variable gets a value of 1, which is used by the [Doctrine Converter][1] to retrieve the job with corresponding ID and then do it available to use in controller.

The route parameters are especially important because they give us possibility to define different kinds of routes.

## Routing Configuration in Dev Environment

The dev environment loads files from `config/routes/dev` folder which contains the routes used by the Web Debug Toolbar, among others.
These files are being loaded at the end of the main routing configuration file.

`config/routes/dev/web_profiler.yaml`:
```yaml
web_profiler_wdt:
    resource: '@WebProfilerBundle/Resources/config/routing/wdt.xml'
    prefix: /_wdt

web_profiler_profiler:
    resource: '@WebProfilerBundle/Resources/config/routing/profiler.xml'
    prefix: /_profiler
```

`config/routes/dev/twig.yaml`:
```yaml
_errors:
    resource: '@TwigBundle/Resources/config/routing/errors.xml'
    prefix: /_error
```

## Route Customizations

For now, when you request the `/` URL in a browser, you will get `"No route found"` error. That’s because this URL does match any route defined in controllers. Let’s change the `job.list` route from the `JobController` it to match the `/` URL. To make this work we will need to make several changes:

- we need to remove the `@Route(“job”)` annotation from the beginning of the `JobController` class that sets a `/job` prefix for all the routes defined below
- we need to add the `/job` prefix to `job.show` route (because we just removed it and it will not work anymore).

```php
class JobController extends AbstractController
{
    /**
     * Lists all job entities.
     *
     * @Route("/", name="job.list")
     */
    public function list() : Response
    ...

    /**
     * Finds and displays a job entity.
     *
     * @Route("job/{id}", name="job.show")
     */
     public function show(Job $job) : Response
    ...
}
```

Now, if you open [http://127.0.0.1/][2] in your browser, you’ll see the Job homepage.

## Route Requirements

The routing system has a built-in validation feature. Each pattern variable can be validated by a regular expression defined using the requirements entry of a route definition:

```php
class JobController extends AbstractController
{
    ...

    /**
     * Finds and displays a job entity.
     *
     * @Route("job/{id}", name="job.show", requirements={"id" = "\d+"})
     */
    public function show(Job $job) : Response
    ...
}
```

The above `requirements` entry forces the ID to be a numeric value. If not, the route won’t match.

We can also define HTTP methods allowed for our routes. For now we should accept only `GET` methods:
```php
class JobController extends AbstractController
{
    /**
     * Lists all job entities.
     *
     * @Route("/", name="job.list", methods="GET")
     */
    public function list() : Response
    ...

    /**
     * Finds and displays a job entity.
     *
     * @Route("job/{id}", name="job.show", methods="GET", requirements={"id" = "\d+"})
     */
    public function show(Job $job) : Response
    ...
}
```

## Route Debugging

While adding and customizing routes, it’s helpful to be able to visualize and get detailed information about your routes.
A great way to see every route in your application is via the `debug:router` console command.  
 Execute the following command from the root of your project:
```bash
bin/console debug:router
```

> Note: don’t forget first to enter php container if you are not in: `docker-compose exec php-fpm bash`  
> and to execute command within container

The command will print a helpful list of all the configured routes in your application.
You can also get very specific information on a single route by including the route name after the command:
```bash
bin/console debug:router job.show
```

### Final Thoughts
That’s all for today! To learn more about the Symfony routing system read the [Routing chapter][3] from the documentation.

You can find the code from today here: [https://github.com/gregurco/jobeet/tree/day5][6]

## Additional information
- [HTTP Methods][5]
- [Routing Component][3]
- [How to Define Route Requirements][4]

## Next Steps

Continue this tutorial here: [Jobeet Day 6: More with the Entity](day-6.md)

Previous post is available here: [Jobeet Day 4: The Controller and the View](day-4.md)

Main page is available here: [Symfony 4.1 Jobeet Tutorial](../index.md)

[1]: https://symfony.com/doc/5.0/bundles/SensioFrameworkExtraBundle/annotations/converters.html
[2]: http://127.0.0.1/
[3]: https://symfony.com/doc/4.1/routing.html
[4]: https://symfony.com/doc/4.1/routing/requirements.html
[5]: https://www.w3schools.com/tags/ref_httpmethods.asp
[6]: https://github.com/gregurco/jobeet/tree/day5
