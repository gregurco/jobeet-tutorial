# Jobeet Day 5: The Routing

## First links

In previous day we created two actions with two templates but did not link them. It's impossible to move from list page to view job description page.
Let's do that! Let's open and do changes in file `templates/job/list.html.twig`:

```diff
- <a href="#">
+ <a href="{{ path('job.show', {id: job.id}) }}">
```

And also in `templates/job/show.html.twig`:

```diff
- <a href="#">Back to the list</a>
+ <a href="{{ path('job.list') }}">Back to the list</a>
```

Now it's possible to view job description and go back to list action.

## URLs

If you click on a job on the Jobeet homepage, the URL looks like this: `/job/1`. How does Symfony make it work? How does Symfony determine the action to call based on this URL?
Why is the job retrieved with the `$job` parameter in the action? Here, we will answer all these questions. 

Symfony uses the `path` template helper function to generate the url for the job which has the id 1. The `job.show` is the name of the route used, defined in the configuration as you will see below.

## Routing Configuration

In Symfony, routing configuration is usually done in the `config/routes/annotations.yaml`. This imports routing configuration.
In our case, the routing is imported from all controllers in folder `src/Controller`, using annotations:

```yaml
controllers:
    resource: ../../src/Controller/
    type: annotation
```

In the `JobController` you will find every job related route defined:
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
    public function listAction() : Response
    ...

    /**
     * Finds and displays a job entity.
     *
     * @Route("/{id}", name="job.show")
     */
    public function showAction(Job $job) : Response
    ...
}
```

Let’s have a closer look to the `job.show` route. The pattern defined by the this route acts like `/job/*` where the wildcard is given the name id (the `/job` part comes from the `@Route(“job”)` definition found at the beginning of the class that acts like a prefix for all the routes defined in that class).
For the URL `/job/1`, the id variable gets a value of 1, which is used by the [Doctrine Converter][1] to retrieve the corresponding job and then made available for you to use in your controller.

The route parameters are especially important because each is made available as an argument to the controller method.

## Routing Configuration in Dev Environment

The dev environment loads files from `config/routes/dev` folder which contains the routes used by the Web Debug Toolbar, among others.
These files loads, at the end, the main routing configuration file.

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

## Additional information

## Next Steps

Continue this tutorial here: [Jobeet Day 6: More with the Model](/days/day-6.md)

Previous post is available here: [Jobeet Day 4: The Controller and the View](/days/day-4.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: http://symfony.com/doc/5.0/bundles/SensioFrameworkExtraBundle/annotations/converters.html