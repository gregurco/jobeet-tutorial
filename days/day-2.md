# Jobeet Day 2: The Project

This day is about the project specifications. They are the same as in the original [Jobeet tutorial][1] so you can see a more detailed description and a mockup design there.

> Jobeet is Open-Source job board software that only does one thing, but does it well. It is easy to use, customize, extend, and embed into your website.
> It supports multiple languages out of the box, and of course uses the latest Web 2.0 technologies to enhance user experience.
> It also provides feeds and an API to interact with it programmatically.

## The User Stories
We will have four type of users: **admin** (owns and administers the website), **user** (visits the website looking for a job), **poster** (visits the website to post jobs) and **affiliate** (re-publishes jobs on his website).


In the original tutorial we had to make two applications, the **frontend**, where the users interact with the website, and the **backend**, where admins manage the website. Using Symfony 4 we would not do that anymore. We will have only one application and, in it, a separate secured section for admins.

## Story F1: On the homepage, the user sees the latest active jobs
On the Jobeet homepage a user sees a list of 10 recent active jobs grouped by category. Only the location, the position, and the company are displayed for each job. For each category there are links that allow to list all the jobs. The user can also search for jobs or post a new job.

![Homepage mockup](/files/images/screenshot_2.png)

## Next Steps

Continue this tutorial here: ~~Jobeet Day 3: The Data Model~~

[1]: http://symfony.com/legacy/doc/jobeet/1_4/en/02?orm=Propel
