# Jobeet Day 3: The Data Model

## The Relational Model
The user stories from the previous day describe the main objects of our project: jobs, affiliates, and categories. Here is the corresponding entity relationship diagram:

![DB schema](/files/images/screenshot_6.png)

In addition to the columns described in the stories, we have also added `created_at` and `updated_at` columns. We will configure Symfony 4 to set their value automatically when an object is saved or updated.

### The Database
To store the jobs, affiliates and categories in the database, Symfony 4 uses Doctrine ORM. To define the database connection parameters you have to edit the `.env` file (for this tutorial we will use MySQL).
```dotenv
DATABASE_URL=mysql://user:password@mysql:3306/jobeet
```
Now that Doctrine knows about your database, you can have it create the database for you (if you did not already created it):

```bash
bin/console doctrine:database:create --if-not-exists
```

## Additional information
- [Databases and the Doctrine ORM][1]

## Next Steps

Continue this tutorial here: [Jobeet Day 4: The Controller and the View](/days/day-4.md)

Previous post is available here: [Jobeet Day 2: The Project](/days/day-2.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/4.0/doctrine.html
