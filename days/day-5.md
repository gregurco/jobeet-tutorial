# Jobeet Day 5: The Routing

In previous day we created two actions with two templates but did not link them. It's impossible to move from list page to view job description page.
Let's do that! Let's open and do changes in file `templates/job/list.html.twig`

```diff
- <a href="#">
+ <a href="{{ path('job.show', {id: job.id}) }}">
```

## Additional information

## Next Steps

Continue this tutorial here: [Jobeet Day 6: More with the Model](/days/day-6.md)

Previous post is available here: [Jobeet Day 4: The Controller and the View](/days/day-4.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)
