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

## Additional information

## Next Steps

Continue this tutorial here: [Jobeet Day 6: More with the Model](/days/day-6.md)

Previous post is available here: [Jobeet Day 4: The Controller and the View](/days/day-4.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)
