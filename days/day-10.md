# Jobeet Day 10: The Admin

With the addition we made in day 9 on Jobeet, the frontend application is now fully useable by job seekers and job posters. It's time to talk a bit about the backend application.  
Today we will develop a complete backend interface for Jobeet in just one day.

## Main concept

The Admin part is another side of Jobeet application and this side includes functionality that is not available in frontend application, for example: creation and deletion of categories.  
For security and architectural reasons this logic should be **separated** from frontend application. Let's keep in mind this idea during all this day.

## First admin controller

We have to create our first admin controller and we know that this controller should be separated from existing ones.  
Create new folder `src/Controller/Admin` with new file `CategoryController.php`:

```php
namespace App\Controller\Admin;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class CategoryController extends AbstractController
{

}
```

All admin controllers will be created in this folder.

## Admin templates layout

We will create admin templates and all of them will extend some layout, like we did with all out templates:

```twig
{% extends 'base.html.twig' %}

{# page content #}
```

But we can't use this one, because admin layout have some differences:
- left menu with links to all CRUDs
- button from top menu will redirect to admin default CRUD and not to jobeet main page
- *and probably many other things will differ with time.*

Also like we did with controllers, we will keep admin templates separately in folder `/templates/admin`.
Let's create admin templates layout `base.html.twig`:

```twig
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Jobeet - Admin{% endblock %}</title>

    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>

    {% block stylesheets %}{% endblock %}

    {% block javascripts %}{% endblock %}

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
</head>
<body>
<nav class="navbar navbar-default">
    <div class="container-fluid">
        <div class="navbar-header">
            <a class="navbar-brand" href="#">Jobeet Admin</a>
        </div>
    </div>
</nav>

<div class="container">
    <div class="row">
        <div class="col-md-2">
            <ul class="nav nav-pills nav-stacked nav-pills-stacked-example">
                <li role="presentation">
                    <a href="#">Categories</a>
                </li>

                <li role="presentation">
                    <a href="#">Jobs</a>
                </li>
            </ul>
        </div>

        <div class="col-md-10">
            {% block body %}{% endblock %}
        </div>
    </div>
</div>
</body>
</html>
```

Now it's pretty similar with one we have, but it gives us possibility to change it without affecting public pages.

## Category CRUD

### List action

We have controller and layout, now let's start creating CRUD from list action:

```php
namespace App\Controller\Admin;

use App\Entity\Category;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\HttpFoundation\Response;

class CategoryController extends AbstractController
{
    /**
     * Lists all categories entities.
     *
     * @Route("/admin/categories", name="admin.category.list", methods="GET")
     *
     * @param EntityManagerInterface $em
     *
     * @return Response
     */
    public function list(EntityManagerInterface $em) : Response
    {
        $categories = $em->getRepository(Category::class)->findAll();

        return $this->render('admin/category/list.html.twig', [
            'categories' => $categories,
        ]);
    }
}
```

It's very simple and small: we get all categories from database and pass them to template.
Just pay attention to route: path starts with `/admin/` and name starts with `admin.`. It will help us to keep admin routes grouped.  

Now create a template `admin/category/list.html.twig` where all the categories will be shown:

```twig
{% extends 'admin/base.html.twig' %}

{% block body %}
    <table class="table">
        <thead>
        <tr>
            <th class="active">Name</th>
            <th class="active">Position</th>
            <th class="active">Jobs</th>
            <th class="active">Affiliates</th>
            <th class="active">Actions</th>
        </tr>
        </thead>

        <tbody>
        {% for category in categories %}
            <tr>
                <td>{{ category.name }}</td>
                <td>{{ category.slug }}</td>
                <td>{{ category.jobs.count }}</td>
                <td>{{ category.affiliates.count }}</td>
                <td>
                    <ul class="list-inline">
                        <li>
                            <a href="#" class="btn btn-default">Edit</a>
                        </li>

                        <li>
                            <a href="#" class="btn btn-danger">Delete</a>
                        </li>
                    </ul>
                </td>
            </tr>
        {% endfor %}
        </tbody>
    </table>

    <a href="#" class="btn btn-success">Create new</a>
{% endblock %}
```

### Create action

For create action first of all we have to create form. Forms that are strictly related to admin will be placed in separate folder too.  
Create `CategoryType` class in `/src/Form/Admin` folder:

```php
namespace App\Form\Admin;

use App\Entity\Category;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;

class CategoryType extends AbstractType
{
    /**
     * @param FormBuilderInterface $builder
     * @param array $options
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name', TextType::class, [
            'constraints' => [
                new NotBlank(),
                new Length(['max' => 100]),
            ]
        ]);
    }

    /**
     * @param OptionsResolver $resolver
     */
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => Category::class,
        ]);
    }
}
```

It's very simple form with only one field and some basic validation rules.
Just notice that `slug` is generated automatically and we don't have to provide it.

The next step is to create action in controller and to build form:

```php
// ...
use App\Form\Admin\CategoryType;
use Symfony\Component\HttpFoundation\Request;

class CategoryController extends AbstractController
{
    // ...

    /**
     * Create category.
     *
     * @Route("/admin/category/create", name="admin.category.create", methods="GET|POST")
     *
     * @param Request $request
     * @param EntityManagerInterface $em
     *
     * @return Response
     */
    public function create(Request $request, EntityManagerInterface $em) : Response
    {
        $category = new Category();
        $form = $this->createForm(CategoryType::class, $category);

        return $this->render('admin/category/create.html.twig', [
            'form' => $form->createView(),
        ]);
    }
}
```

Now create template and render the form:

```twig
{% extends 'admin/base.html.twig' %}

{% block body %}
    <h1>Create new category</h1>
    
    {{ form_start(form, {'attr': {'novalidate': 'novalidate'}}) }}
        {{ form_widget(form) }}
    
        <div class="form-group">
            <div class="col-sm-offset-2 col-sm-10">
                <button type="submit" class="btn btn-default">Save</button>
            </div>
        </div>
    {{ form_end(form) }}
    
    <a href="{{ path('admin.category.list') }}" class="btn btn-default">
        <span class="glyphicon glyphicon-arrow-left" aria-hidden="true"></span>
        back to list
    </a>
{% endblock %}
```

Link the "Create new" button from list page with create page in `templates/admin/category/list.html.twig`:

```diff
- <a href="#" class="btn btn-success">Create new</a>
+ <a href="{{ path('admin.category.create') }}" class="btn btn-success">Create new</a>
```

Page is accessible and form is displayed, but not handled. Do some small modification in `create` action:

```php
// ...

class CategoryController extends AbstractController
{
    // ...
    public function create(Request $request, EntityManagerInterface $em) : Response
    {
        $category = new Category();
        $form = $this->createForm(CategoryType::class, $category);

        if ($form->isSubmitted() && $form->isValid()) {
            $em->persist($category);
            $em->flush();

            return $this->redirectToRoute('admin.category.list');
        }

        return $this->render('admin/category/create.html.twig', [
            'form' => $form->createView(),
        ]);
    }
}
```

That's it. Now admin can create as much categories as wants.

### Edit action

...

### Delete action

...

## Additional information
- [The Symfony MakerBundle][1]

## Next Steps

Continue this tutorial here: [Jobeet Day 11: The User](/days/day-11.md)

Previous post is available here: [Jobeet Day 9: Console Commands](/days/day-9.md)

Main page is available here: [Symfony 4.1 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/1.0/bundles/SymfonyMakerBundle/index.html
