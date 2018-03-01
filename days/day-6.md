# Jobeet Day 6: More with the Entity

## The Doctrine Query Object

From the second day’s requirements: “On the homepage, the user sees the latest active jobs”.
But as of now, all jobs are displayed, whether they are active or not:
```php
public function listAction() : Response
{
    $jobs = $this->getDoctrine()->getRepository(Job::class)->findAll();

    return $this->render('job/list.html.twig', [
        'jobs' => $jobs,
    ]);
}
```

An active job is one that was posted less than 30 days ago.
The `$jobs = $this->getDoctrine()->getRepository(Job::class)->findAll();` method will make a request to the database to get all the jobs.
We are not specifying any condition which means that all the records are retrieved from the database.

Let’s change it to only select active jobs:
```php
public function listAction(EntityManagerInterface $em) : Response
{
    $query = $em->createQuery(
        'SELECT j FROM App:Job j WHERE j.createdAt > :date'
    )->setParameter('date', new \DateTime('-30 days'));

    return $this->render('job/list.html.twig', [
        'jobs' => $jobs,
    ]);
}
```

## Additional information

## Next Steps

Continue this tutorial here: [Jobeet Day 7: Playing with the Category Page](/days/day-7.md)

Previous post is available here: [Jobeet Day 5: The Routing](/days/day-5.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)
