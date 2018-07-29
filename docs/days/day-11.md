# Jobeet Day 11: The User

Yesterday was packed with a lot of information.

Today, we will discover how symfony manages persistent data between HTTP requests. As you might know, the HTTP protocol is stateless, which means that each request is independent from its preceding or proceeding ones. Modern sites need a way to persist data between requests to enhance the user experience.

A user session can be identified using a cookie. In Symfony, the developer does not need to manipulate the session directly, but rather uses the `Session` object from `HttpFoundation` component.

## User Flashes

We have already seen flash messages in action. A flash is an ephemeral message stored in the session that will be automatically deleted after the very next request.
It is very useful when you need to display a message to the user after a redirect.

![Flash message](../files/images/screenshot_13.png)

A flash is set by using the `addFlash()` method in controller:

```php
$this->addFlash('notice', 'Your job was published');
```

The first argument is the identifier of the flash and the second one is the message to display.
You can define whatever flashes you want, but **notice** and **error** are two of the more common ones.

It is up to the developer to include the flash message in the templates.
Message shown above is rendered in `templates/job/show.html.twig`:

```twig
{% for message in app.flashes('notice') %}
    <div class="alert alert-success" role="alert">
        {{ message }}
    </div>
{% endfor %}
```

## User Session

Unfortunately, the Jobeet user stories have no requirement that includes storing something in the user session.
So let’s add a new requirement: to ease job browsing, the last three jobs viewed by the user should be displayed in the menu with links to come back to the job page later on.

When a user access a job page, the displayed job object needs to be added in the user history and stored in the session.
It’s better to organize this logic in service from the beginning. Create new service `JobHistoryService` in already existing folder `/src/Service`:

```php
namespace App\Service;

use App\Entity\Job;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Session\SessionInterface;

class JobHistoryService
{
    private const MAX = 3;

    /** @var SessionInterface */
    private $session;

    /** @var EntityManagerInterface */
    private $em;

    /**
     * @param SessionInterface $session
     * @param EntityManagerInterface $em
     */
    public function __construct(SessionInterface $session, EntityManagerInterface $em)
    {
        $this->session = $session;
        $this->em = $em;
    }

    /**
     * @param Job $job
     *
     * @return void
     */
    public function addJob(Job $job) : void
    {
        // add job to session
    }

    /**
     * @return Job[]
     */
    public function getJobs() : array
    {
        // get jobs from session
    }
}
```

We already have constant `MAX` that will be used to limit the number of jobs in history.
Also we have injected `Session` class from `HttpFoundation` component that will be used to work with session storage and `EntityManager` class to fetch jobs from database.

> Do not work directly with `$_SESSION` superglobal and use services provided by Symfony!

Use `JobHistoryService` in controller to add jobs to session:

```php
// ...
use App\Service\JobHistoryService;

class JobController extends AbstractController
{
    // ...
    
    /**
     * Finds and displays a job entity.
     *
     * @Route("job/{id}", name="job.show", methods="GET", requirements={"id" = "\d+"})
     *
     * @Entity("job", expr="repository.findActiveJob(id)")
     *
     * @param Job $job
     * @param JobHistoryService $jobHistoryService
     *
     * @return Response
     */
    public function show(Job $job, JobHistoryService $jobHistoryService) : Response
    {
        $jobHistoryService->addJob($job);

        return $this->render('job/show.html.twig', [
            'job' => $job,
        ]);
    }
    
    // ...
}
```

Now let’s write the implementation for `addJob` method:

```php
// ...

class JobHistoryService
{
    // ...

    /**
     * @param Job $job
     *
     * @return void
     */
    public function addJob(Job $job) : void
    {
        $jobs = $this->getJobIds();

        // Add job id to the beginning of the array
        array_unshift($jobs, $job->getId());

        // Remove duplication of ids
        $jobs = array_unique($jobs);

        // Get only first 3 elements
        $jobs = array_slice($jobs, 0, self::MAX);

        // Store IDs in session
        $this->session->set('job_history', $jobs);
    }
    
    /**
     * @return array
     */
    private function getJobIds() : array
    {
        return $this->session->get('job_history', []);
    }
    
    // ...
}
```

> We could have feasibly stored the Job objects directly into the session. This is strongly discouraged because the session variables are serialized between requests. And when the session is loaded, the Job objects are de-serialized and can be "stalled" if they have been modified or deleted in the meantime.

Open view page of random job and there will be no visible changes but we know that something should appear in session.
To track request session open Symfony Profiler from bottom toolbar (or directly: [http://127.0.0.1/_profiler][2] and select the last request), open "Request / Response" from left menu and "Session" tab:

![Session in Profiler](../files/images/screenshot_16.png)

there you can see key we used to store our IDs and the current value *(in my case job was with ID 47)*.

Now use these IDs to fetch jobs from database:

```php
// ...

class JobHistoryService
{
    // ...

    /**
     * @return Job[]
     */
    public function getJobs() : array
    {
        $jobs = [];
        $jobRepository = $this->em->getRepository(Job::class);

        foreach ($this->getJobIds() as $jobId) {
            $jobs[] = $jobRepository->findActiveJob($jobId);
        }

        return $jobs;
    }
}
```

Note that we used `findActiveJob` method from repository and if job from history become inactive then it will not be displayed.

Use this method in controller to pass jobs to template:

```php
// ...

class JobController extends AbstractController
{
    /**
     * Lists all job entities.
     *
     * @Route("/", name="job.list", methods="GET")
     *
     * @param EntityManagerInterface $em
     * @param JobHistoryService $jobHistoryService
     *
     * @return Response
     */
    public function list(EntityManagerInterface $em, JobHistoryService $jobHistoryService) : Response
    {
        $categories = $em->getRepository(Category::class)->findWithActiveJobs();

        return $this->render('job/list.html.twig', [
            'categories' => $categories,
            'historyJobs' => $jobHistoryService->getJobs(),
        ]);
    }
    
    // ...
}
```

and render in `templates/job/list.html.twig`:

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% if historyJobs %}
        <div class="panel panel-default">
            <div class="panel-body">
                Recent viewed jobs:
                {% for historyJob in historyJobs %}
                    <a href="{{ path('job.show', {id: historyJob.id}) }}">{{ historyJob.position ~ ' - ' ~ historyJob.company}}</a>{{ loop.last ? '' : ', ' }}
                {% endfor %}
            </div>
        </div>
    {% endif %}
    
    <h4>{{ category.name }}</h4>
    
    {# ... #}
{% endblock %}
```

But don’t forget that we have one route in `CategoryController` with similar functionality and it would be good to display jobs history too.
Move the `div` with all inner content to separate file `templates/job/_job_history.html.twig`:

```twig
{% if historyJobs %}
    <div class="panel panel-default">
        <div class="panel-body">
            Recent viewed jobs:
            {% for historyJob in historyJobs %}
                <a href="{{ path('job.show', {id: historyJob.id}) }}">{{ historyJob.position ~ ' - ' ~ historyJob.company}}</a>{{ loop.last ? '' : ', ' }}
            {% endfor %}
        </div>
    </div>
{% endif %}
```

and replace displaying in `job/list.html.twig`:

```twig
{% extends 'base.html.twig' %}

{% block title %}
    Jobs in the {{ category.name }} category
{% endblock %}

{% block body %}
    {% include 'job/_job_history.html.twig' with {'historyJobs': historyJobs} only %}

    <h4>{{ category.name }}</h4>

    {# ... #}
{% endblock %}
```

Code became more reusable. Do adjustments in `CategoryController`:

```php
// ...
use App\Service\JobHistoryService;

class CategoryController extends Controller
{
    /**
     * Finds and displays a category entity.
     *
     * @Route(
     *     "/category/{slug}/{page}",
     *     name="category.show",
     *     methods="GET",
     *     defaults={"page": 1},
     *     requirements={"page" = "\d+"}
     * )
     *
     * @param Category $category
     * @param int $page
     * @param PaginatorInterface $paginator
     * @param JobHistoryService $jobHistoryService
     *
     * @return Response
     */
    public function show(
        Category $category,
        int $page,
        PaginatorInterface $paginator,
        JobHistoryService $jobHistoryService
    ) : Response {
        $activeJobs = $paginator->paginate(
            $this->getDoctrine()->getRepository(Job::class)->getPaginatedActiveJobsByCategoryQuery($category),
            $page,
            $this->getParameter('max_jobs_on_category')
        );

        return $this->render('category/show.html.twig', [
            'category' => $category,
            'activeJobs' => $activeJobs,
            'historyJobs' => $jobHistoryService->getJobs(),
        ]);
    }
}
```

and reuse `_job_history.html.twig` in `category/show.html.twig`:

```twig
{# ... #}

{% block body %}
    {% include 'job/_job_history.html.twig' with {'historyJobs': historyJobs} only %}

    <h4>{{ category.name }}</h4>

    {# ... #}
{% endblock %}
```

Now user will see recently viewed jobs:

![Job History](../files/images/screenshot_17.png)

## Application Security

### Authentication

Like many other symfony features, security is managed by the `security.yml` file.
In this file the basic security configuration for your application is contained.
For instance, you can find the default configuration for the application in the `config/packages` directory:

```yaml
security:
    providers:
        in_memory: { memory: ~ }

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            anonymous: ~
```

From the beginning we have two `firewalls`: `dev` isn’t important, it’s used only in dev mode and gives access to some dev routes, but `main` firewall handles all other routes because there is no `pattern` key.

Let’s activate HTTP basic authentication in `main` firewall:

```diff
  security:
      providers:
          in_memory: { memory: ~ }
  
      firewalls:
          dev:
              pattern: ^/(_(profiler|wdt)|css|images|js)/
              security: false
          main:
              anonymous: ~
+             http_basic: ~
```

and restrict access for routes that start with `/admin` to be accessible only by users with role `ROLE_ADMIN`:

```diff
  security:
      providers:
          in_memory: { memory: ~ }
  
      firewalls:
          dev:
              pattern: ^/(_(profiler|wdt)|css|images|js)/
              security: false
          main:
              anonymous: ~
              http_basic: ~

+     access_control:
+         - { path: ^/admin, roles: ROLE_ADMIN }
```

Open any admin route and you will be requested to provide login and password:

![Basic Auth Required](../files/images/screenshot_18.png)

We don’t have any users in database, but Symfony gives us possibility to list users directly in this `yaml` file too:

```diff
  security:
      providers:
          in_memory:
+            memory:
+                users:
+                    admin:
+                        password: someStrongPassword
+                        roles: 'ROLE_ADMIN'
 
+     encoders:
+        Symfony\Component\Security\Core\User\User: plaintext
  
      firewalls:
          dev:
              pattern: ^/(_(profiler|wdt)|css|images|js)/
              security: false
          main:
              anonymous: ~
              http_basic: ~
  
      access_control:
          - { path: ^/admin, roles: ROLE_ADMIN }
  
```

Try to login using username `admin` and password `someStrongPassword`. It should work!

User provider loads user information and put it into a `User` object. If you load users from the database or some other source, you’ll use your own custom User class. But when you use the "in memory" provider, it gives you a `Symfony\Component\Security\Core\User\User` object.

Whatever your User class is, you need to tell Symfony what algorithm was used to encode the passwords. In this case, the passwords are just `plaintext`.

If you refresh now, you’ll be logged in! The web debug toolbar even tells you who you are and what roles you have:

![Auth Information](../files/images/screenshot_19.png)

## Third Party Bundles

### Install Bundle

As we don’t like to reinvent the wheel, we won’t develop the login action from scratch. Instead, we will install a **symfony bundle**.

One of the great strengths of the symfony framework is the [bundle ecosystem][4].
As we will see in coming days, it is quite powerful, as a bundle can contain anything from configuration to controllers and assets.

Today, we will install [FOSUserBundle][5] to integrate user management system:

```bash
composer require friendsofsymfony/user-bundle "~2.1"
```

There is a chance to see next error after install:

![FOSUserBundle install error](../files/images/screenshot_20.png)

But that is critical. After composer installed the bundle, Symfony added it to `config/bundles.php` and cleared the cache.
The problem is that there is no default configuration of this bundle, but Symfony requires configuration for each connected bundle.
We gonna fix it and clear cache manually later.

Let’s begin with templating configuration in `config/packages/framework.yaml`:

```yaml
framework:
    # ...
    
    templating:
        engines: 'twig'
```

`FOSUserBundle` uses `twig` templating engine and we have to specify it explicitly.

### User Entity

Create new entity `User` in `src/Entity/` folder:

```php
namespace App\Entity;

use FOS\UserBundle\Model\User as BaseUser;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="users")
 */
class User extends BaseUser
{
    /**
     * @var int
     *
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;
}
```

Note that we extend the entity from `FOSUserBundle` and all the fields are defined in extended class.
We just define the annotations for ID field. Also you are free to add whatever fields you want.

### Security

In order for Symfony’s security component to use the `FOSUserBundle`, you must tell it to do so in the `security.yml` file.

Below is a minimal example of the configuration necessary to use the `FOSUserBundle` in your application.
Just replace existing one with next content:

```yaml
security:
    encoders:
        FOS\UserBundle\Model\UserInterface: bcrypt

    role_hierarchy:
        ROLE_ADMIN: ROLE_USER

    providers:
        fos_userbundle:
            id: fos_user.user_provider.username

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
                csrf_token_generator: security.csrf.token_manager

            logout: true
            anonymous: true

    access_control:
        - { path: ^/admin/, role: ROLE_ADMIN }
```

We changed `encoder` type from `plaintext` to `bcrypt`, to comply with current security standard.  
Option `role_hierarchy` is used to define role inheritance. In our case `ROLE_ADMIN` inherits all permissions of `ROLE_USER`.  
Under the `providers` section, we defined alias `fos_userbundle` for service `fos_user.user_provider.username` from `FOSUserBundle`. [This service][6] knows how to fetch users from database.
Next, take a look at and examine the `firewalls` section. Here we have declared a firewall named `main`. By specifying `form_login`, you have told the Symfony Framework that any time a request is made to this firewall that leads to the user needing to authenticate himself, the user will be redirected to a form where he will be able to enter his credentials.
It should come as no surprise then that you have specified the user provider service we declared earlier as the provider for the firewall to use as part of the authentication process.  
The `access_control` section is where we specify the credentials necessary for users trying to access specific parts of your application.
We want to protect admin routes and that’s why we have specified that any request beginning with `/admin` will require a user to have the `ROLE_ADMIN` role.

### Bundle Configuration

Create missed configuration file `config/packages/fos_user.yaml`:

```yaml
fos_user:
    db_driver: orm
    firewall_name: main
    user_class: App\Entity\User
    from_email:
        address: "user@email.com"
        sender_name: "Sender Name"
```

Here we specified minimum required information to start working with bundle.  
You can notice that there is path to `User` class that we created and `main` firewall we defined recently in `security.yml` file.

Try to clear cache:

```bast
bin/console cache:clear
```

and it should be successfully cleared:

![Clear Cache](../files/images/screenshot_21.png)

### Importing routes

Bundle provide us a list of routes for actions like login, registration, reset password and etc.
And we have to import routes from bundle to our routing file `config/routes.yaml`:

```yaml
fos_user:
    resource: "@FOSUserBundle/Resources/config/routing/security.xml"
```

### Create Migration

We created `User` entity but we still don’t have `users` table in database.
Run next command and create migration with difference between entities set and database state:

```bash
bin/console doctrine:migration:diff
```

New migration should appear in `src/Migrations` folder. Now run it:

```bash
bin/console doctrine:migration:migrate
```

New table should appear in database.

### Templates

Open [http://127.0.0.1/login][8] page:

![Login page without styling](../files/images/screenshot_22.png)

You can notice that this login page looks very simple, because `FOSUserBundle` doesn’t know anything about styling of our application.
Fortunately, Symfony [provides][7] us possibility easy to redefine templates and other parts of bundles.  
To redefine template of login page just create file `login.html.twig` in folder `templates/bundles/FOSUserBundle/Security` with next content:

```twig
{% extends 'base.html.twig' %}

{% trans_default_domain 'FOSUserBundle' %}

{% block body %}
    {% if error %}
        <div>{{ error.messageKey|trans(error.messageData, 'security') }}</div>
    {% endif %}

    <form class="form-horizontal col-sm-offset-2 col-sm-8" action="{{ path("fos_user_security_check") }}" method="post">

        <input type="hidden" name="_csrf_token" value="{{ csrf_token }}"/>

        <div class="form-group">
            <label class="col-sm-2 control-label" for="username">{{ 'security.login.username'|trans }}</label>
            <div class="col-sm-10">
                <input class="form-control" type="text" id="username" name="_username" value="{{ last_username }}" required="required"/>
            </div>
        </div>

        <div class="form-group">
            <label class="col-sm-2 control-label" for="password">{{ 'security.login.password'|trans }}</label>
            <div class="col-sm-10">
                <input class="form-control" type="password" id="password" name="_password" required="required"/>
            </div>
        </div>

        <div class="form-group">
            <div class="col-sm-offset-2 col-sm-10">
                <div class="checkbox">
                    <label for="remember_me">
                        <input type="checkbox" id="remember_me" name="_remember_me" value="on"/> {{ 'security.login.remember_me'|trans }}
                    </label>
                </div>
            </div>
        </div>

        <div class="form-group">
            <div class="col-sm-offset-2 col-sm-10">
                <input type="submit" class="btn btn-default" id="_submit" name="_submit" value="{{ 'security.login.submit'|trans }}"/>
            </div>
        </div>

    </form>
{% endblock %}
```

Now login page looks good:

![Login page with styling](../files/images/screenshot_23.png)

Just few small changes remained to do in `templates/base.html.twig` template:

```diff
  {# ... #}
  
  <nav class="navbar navbar-default">
      <div class="container-fluid">
          <div class="navbar-header">
              <a class="navbar-brand" href="{{ path('job.list') }}">Jobeet</a>
          </div>
  
          <div class="collapse navbar-collapse">
              <ul class="nav navbar-nav navbar-right">
+                 {% if is_granted('ROLE_ADMIN') %}
+                     <li>
+                         <div>
+                             <a href="{{ path('admin.category.list') }}" class="btn btn-default navbar-btn">Admin Panel</a>
+                         </div>
+                     </li>
+                 {% endif %}
  
                  <li>
                      <div>
                          <a href="{{ path('job.create') }}" class="btn btn-default navbar-btn">Post a Job</a>
                      </div>
                  </li>
  
+                 {% if app.user %}
+                     <li><a href="{{ path('fos_user_security_logout') }}">Logout</a></li>
+                 {% endif %}
              </ul>
          </div>
      </div>
  </nav>
  
  {# ... #}
```

We added `Admin Panel` button if user is authenticated and has role `ROLE_ADMIN` and also `logout` button that moves to route provided by `FOSUserBundle`.

That’s all for today, you can find the code here: [https://github.com/gregurco/jobeet/tree/day11][9]

## Additional information
- [Session Management][1]
- [Security][3]
- [FOSUserBundle][5]
- [How to Override any Part of a Bundle][7]

## Next Steps

Continue this tutorial here: [Jobeet Day 12: The API](day-12.md)

Previous post is available here: [Jobeet Day 10: The Admin](day-10.md)

Main page is available here: [Symfony 4.1 Jobeet Tutorial](../index.md)

[1]: https://symfony.com/doc/4.1/components/http_foundation/sessions.html
[2]: http://127.0.0.1/_profiler
[3]: https://symfony.com/doc/4.1/security.html
[4]: https://packagist.org/explore/?type=symfony-bundle
[5]: https://symfony.com/doc/master/bundles/FOSUserBundle/index.html
[6]: https://github.com/FriendsOfSymfony/FOSUserBundle/blob/v2.1.2/Security/UserProvider.php
[7]: https://symfony.com/doc/4.1/bundles/override.html
[8]: http://127.0.0.1/login
[9]: https://github.com/gregurco/jobeet/tree/day11
