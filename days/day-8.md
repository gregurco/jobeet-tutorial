# Jobeet Day 8: The Forms

Any website has forms, from the simple contact form to the complex ones with lots of fields.
Writing forms is also one of the most complex and tedious task for a web developer: you need to write the HTML form, implement validation
rules for each field, process the values to store them in a database, display error messages, repopulate fields in case of errors, and much more…

## Create the Job Form

To deal with creation of jobs we will need forms, that in Symfony are realized by [Form Component][3].
The Form component allows you to easily create, process and reuse forms.
We will crate a class that generally is `Form Builder` class. There we will define fields, that form should have, validation rules and many other things.
So, let's create a folder `src/Form` where all forms will be placed and to create our first form in file `JobType.php`:

```php
namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class JobType extends AbstractType
{
    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {

    }

    /**
     * {@inheritdoc}
     */
    public function configureOptions(OptionsResolver $resolver)
    {

    }
}
```

As you can notice, our class extends `AbstractType` class and usually you will have to work with `buildForm` and `configureOptions` methods.
That's why we defined them from the beginning.

Every form needs to know the name of the class that holds the underlying data.
Let's specify the `data_class` option by changing `configureOptions` method:

```php
public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults([
        'data_class' => Job::class
    ]);
}
```

*Note: don't forget to import `Job` class*

Now we should define form fields in `buildForm` method:

```php
// ....
use Symfony\Component\Form\Extension\Core\Type\DateTimeType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\UrlType;

class JobType extends AbstractType
{
    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('type', TextType::class)
            ->add('company', TextType::class)
            ->add('logo', TextType::class)
            ->add('url', UrlType::class)
            ->add('position', TextType::class)
            ->add('location', TextType::class)
            ->add('description', TextType::class)
            ->add('howToApply', TextType::class)
            ->add('public', TextType::class)
            ->add('activated', TextType::class)
            ->add('email', EmailType::class)
            ->add('expiresAt', DateTimeType::class)
            ->add('category', TextType::class)
            ->add('token', TextType::class);
    }
    
    // ...   
}
```

We call method `add` on form builder and add all fields. First argument is field name. It should be the same as in entity.
The second argument is field type. In our case we used [TextType][6], [EmailType][7] and [DateTimeType][8]. And 3rd parameter is options (we did not use it yet)
Field type affects rendering of field and also provides different options for configuration.

Example: `TextType` will be rendered as `<input type="text" ...>`

Let's try other types. For example, we defined `public` field as `TextType`, but in entity it is boolean.
It's better to render it as selector with YES and NO options. We will implement [ChoiceType][9]:

```diff
- ->add('public', TextType::class)
+ ->add('public', ChoiceType::class, [
+     'choices'  => [
+         'Yes' => true,
+         'No' => false,
+     ],
+     'label' => 'Public?',
+ ])
```

We changed type and also defined next options: `choices` - items, that will be used as `<options>` and `label`.
Do the same thing with `activated` field too.

Also `category` is `TextType`, but we have categories in DB and it would be good to render selector with these options.
It looks like it should be `ChoiceType`, but choice from entities is specific case and we have separate type for it: [EntityType][10].
It extends `ChoiceType` but with some additional options related to DB.

```diff
- ->add('category', TextType::class)
+ ->add('category', EntityType::class, [
+     'class' => Category::class,
+     'choice_label' => 'name',
+ ])
```

We specified `choice_label` to select a field from entity that will be shown as option in selector.
We changed only one line of code, but in template instead of simple input we will have select with all categories in options.

Now let's change `type` field. For now it's text field, but in the second day’s description we have next requirement:
> Type (full-time, part-time, or freelance)

We need defined list of options and let's do it in `Job` entity:

```php
class Job
{
    const FULL_TIME_TYPE = 'full-time';
    const PART_TIME_TYPE = 'part-time';
    const FREELANCE_TYPE = 'freelance';

    const TYPES = [
        self::FULL_TIME_TYPE,
        self::PART_TIME_TYPE,
        self::FREELANCE_TYPE,
    ];

    // ...
}
```

Currently we can change the form type of `type` field to `ChoiceType`:

```diff
- ->add('type', TextType::class)
+ ->add('type', ChoiceType::class, [
+     'choices' => array_combine(Job::TYPES, Job::TYPES),
+     'expanded' => true,
+ ])
```

Why we used `array_combine` but not directly `Job::TYPES`? We want to show options with same label and value.
For example, option with label `freelance` should have value `freelance` (`<option value="freelance">freelance</option>`) and `array_combine` helps us to do that.
Also `expanded` is true to show you how different `ChoiceType` can be rendered. We will see it later.

Let's add some labels and options. The final result should be:
```php
namespace App\Form;

use App\Entity\Category;
use App\Entity\Job;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
use Symfony\Component\Form\Extension\Core\Type\DateTimeType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\UrlType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class JobType extends AbstractType
{
    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('type', ChoiceType::class, [
                'choices' => array_combine(Job::TYPES, Job::TYPES),
                'expanded' => true,
            ])
            ->add('company', TextType::class)
            ->add('logo', TextType::class)
            ->add('url', UrlType::class)
            ->add('position', TextType::class)
            ->add('location', TextType::class)
            ->add('description', TextType::class)
            ->add('howToApply', TextType::class, [
                'label' => 'How to apply?',
            ])
            ->add('public', ChoiceType::class, [
                'choices'  => [
                    'Yes' => true,
                    'No' => false,
                ],
                'label' => 'Public?',
            ])
            ->add('activated', ChoiceType::class, [
                'choices'  => [
                    'Yes' => true,
                    'No' => false,
                ],
            ])
            ->add('email', EmailType::class)
            ->add('expiresAt', DateTimeType::class)
            ->add('category', EntityType::class, [
                'class' => Category::class,
                'choice_label' => 'name',
            ])
            ->add('token', TextType::class);
    }

    /**
     * {@inheritdoc}
     */
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => Job::class
        ]);
    }
}
```

## Create Job Action

We just created `JobType` form class. The next step is to create and render actual form.
In Symfony, this is done by building a form object and then rendering it in a template.
For now, we will create new action inside the `JobController` controller:

```php
// ...
use App\Form\JobType;

class JobController extends AbstractController
{
    // ...
    
    /**
     * Creates a new job entity.
     *
     * @Route("/job/create", name="job.create")
     * @Method({"GET"})
     *
     * @return Response
     */
    public function createAction() : Response
    {
        $job = new Job();
        $form = $this->createForm(JobType::class, $job);
    
        return $this->render('job/create.html.twig', [
            'job' => $job,
            'form' => $form->createView(),
        ]);
    }
}
```

We created new Job object and passed it to `createForm` method along with `JobType` class.
This method will create a form class based on entity object and rules described in `JobType` class.
In form we will need a special form "view" object, that's why we passed to template not the form, but the result of `createView` method.

## The Form Template

In previous step we passed the data to `job/create.html.twig` template, but we don't have it yet.
Let's create it and use a set of form helper functions:

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Job creation</h1>

    {{ form_start(form) }}
        {{ form_widget(form) }}

        <div class="form-group">
            <div class="col-sm-offset-2 col-sm-10">
                <button type="submit" class="btn btn-default">Create</button>
            </div>
        </div>
    {{ form_end(form) }}
{% endblock %}
```
That's it! Form will be rendered due to:
- `form_start` - renders `<form>` tag with all needed attributed (method, encryption, etc).
- `form_widget` - renders all the fields, labels and any validation error messages.
- `form_end` - renders `</form>` tag.

Open the browser and access `/job/create` path to see how form is rendered.
The form with all fields are rendered, but the styling is not the same as in [bootstrap][11].
The good new is that [Twig Bridge][12] component, that is responsible for integration of Twig in Symfony, comes with some [themes][13] out of the box.
We use bootstrap 3 and will choose `bootstrap_3_horizontal_layout.html.twig` theme file. Let's setup it in `config/packages/twig.yaml`:
```twig
twig:
    # ...
    form_themes:
        - 'bootstrap_3_horizontal_layout.html.twig'
```

Refresh the page. Now form should look in bootstrap 3 style.

Also you can notice that we wrote html for submit button and did't use [SubmitType][14].
It's good practice because form become more reusable. Read more [here][15].

## Form processing

Form is built and rendered. Processing is next.
If you submit the form, nothing will happen. Let's fix it:

```php
class JobController extends AbstractController
{
    // ...
    
    /**
     * Creates a new job entity.
     *
     * @Route("/job/create", name="job.create")
     * @Method({"GET", "POST"})
     *
     * @param Request $request
     * @param EntityManagerInterface $em
     *
     * @return RedirectResponse|Response
     */
    public function createAction(Request $request, EntityManagerInterface $em) : Response
    {
        $job = new Job();
        $form = $this->createForm(JobType::class, $job);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $em->persist($job);
            $em->flush();

            return $this->redirectToRoute('job.list');
        }

        return $this->render('job/create.html.twig', [
            'job' => $job,
            'form' => $form->createView(),
        ]);
    }
```

- we added `POST` method to annotation, because form will be submitted with this method.
- request object is added to method arguments because in request will be data of submitted form
- entity manager is added to method arguments because we will use it to store new job
- `handleRequest` method maps request data to form
- `isSubmitted` method checks if form was submitted or method is called with get method and we need only to show the form
- `isValid` method tracks if all requirements are met (we will add them in next step)
- `redirectToRoute` is used to redirect to list of jobs and to prevent repeated submit of form (CTRL + R)

## Validation

For now form is built, rendered and processed, but we don't validate any information.
Form accepts anything. We should add some validation rules.

> Validation rules are called `constraints` in symfony forms.

There are two places where constraints can be added: in annotations of entity or in form class.
We will choose the second option, to keep the logic of form validation in form class and not to enlarge entity class.

Our first field in form is `type` field. According to entity this field should be required:

```diff
 ->add('type', ChoiceType::class, [
     'choices' => array_combine(Job::TYPES, Job::TYPES),
     'expanded' => true,
+    'constraints' => [
+        new NotBlank(),
+    ]
 ])
```

and import `Symfony\Component\Validator\Constraints\NotBlank` class.  
Next field is `company` and it's required text field with a maximum length of 255 characters:

```diff
 ->add('company', TextType::class, [
+    'constraints' => [
+        new NotBlank(),
+        new Length(['max' => 255]),
+    ]
 ])
```

Try to submit more than 255 characters and you will see error:

![Validation error: max length](/files/images/screenshot_9.png)

We have more specific field: `expiresAt`. It's datetime field, that should be required and the value should be datetime in feature:

```php
->add('expiresAt', DateTimeType::class, [
    'constraints' => [
        new NotBlank(),
        new DateTime(),
        new GreaterThanOrEqual('now')
    ]
])
```

Constraint `GreaterThanOrEqual` works well with numbers and date/datetime too.

Symfony has a big list of constraints out of the box. You can find all them [here][16].
Review all fields and add relevant constraints and in final you should see something similar:

```php
namespace App\Form;

use App\Entity\Category;
use App\Entity\Job;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
use Symfony\Component\Form\Extension\Core\Type\DateTimeType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\UrlType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\DateTime;
use Symfony\Component\Validator\Constraints\Email;
use Symfony\Component\Validator\Constraints\GreaterThanOrEqual;
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;

class JobType extends AbstractType
{
    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('type', ChoiceType::class, [
                'choices' => array_combine(Job::TYPES, Job::TYPES),
                'expanded' => true,
                'constraints' => [
                    new NotBlank(),
                ]
            ])
            ->add('company', TextType::class, [
                'constraints' => [
                    new NotBlank(),
                    new Length(['max' => 255]),
                ]
            ])
            ->add('logo', TextType::class)
            ->add('url', UrlType::class, [
                'required' => false,
                'constraints' => [
                    new Length(['max' => 255]),
                ]
            ])
            ->add('position', TextType::class, [
                'constraints' => [
                    new NotBlank(),
                    new Length(['max' => 255]),
                ]
            ])
            ->add('location', TextType::class, [
                'constraints' => [
                    new NotBlank(),
                    new Length(['max' => 255]),
                ]
            ])
            ->add('description', TextType::class, [
                'constraints' => [
                    new NotBlank(),
                ]
            ])
            ->add('howToApply', TextType::class, [
                'label' => 'How to apply?',
                'constraints' => [
                    new NotBlank(),
                ]
            ])
            ->add('public', ChoiceType::class, [
                'choices'  => [
                    'Yes' => true,
                    'No' => false,
                ],
                'label' => 'Public?',
                'constraints' => [
                    new NotBlank(),
                ]
            ])
            ->add('activated', ChoiceType::class, [
                'choices'  => [
                    'Yes' => true,
                    'No' => false,
                ],
                'constraints' => [
                    new NotBlank(),
                ]
            ])
            ->add('email', EmailType::class, [
                'constraints' => [
                    new NotBlank(),
                    new Email()
                ]
            ])
            ->add('expiresAt', DateTimeType::class, [
                'constraints' => [
                    new NotBlank(),
                    new DateTime(),
                    new GreaterThanOrEqual('now')
                ]
            ])
            ->add('category', EntityType::class, [
                'class' => Category::class,
                'choice_label' => 'name',
                'constraints' => [
                    new NotBlank(),
                ]
            ])
            ->add('token', TextType::class, [
                'constraints' => [
                    new NotBlank(),
                    new Length(['max' => 255]),
                ]
            ]);
    }

    // ...
}
```

You can observe that we used `'required' => false` and `new NotBlank()` constraint. What is the difference?
By default, all fields have `required` set to true and this option affects only the rendering of the field: it adds `required` to html field tag:
```html
<input type="text" name="company" required>
```

But this "requirement" can be easy bypassed by developer tools provided by every browser.  
`NotBlank` constraint checks on the level of form processing and can't be bypassed.
If you want to test fully the power of constraints or simply want to disable browser validation add next parameter to `form_start` function in template:
```twig
{{ form_start(form, {'attr': {'novalidate': 'novalidate'}}) }}
```

## Handling File Uploads

To handle the actual file upload in the form, we will need to change the logo field type to `FileType` in the form:

```php
->add('logo', FileType::class, [
    'required' => false,
])
```

Also we need to add an `Image` constraint:
```php
->add('logo', FileType::class, [
    'required' => false,
    'constraints' => [
        new Image(),
    ]
])
```

When the form is submitted, the logo field will be an instance of `UploadedFile`.
It can be used to move the file to a permanent location. After this we will set the job logo property to the uploaded file name.

For this to work we need to add a new parameter, `jobs_directory`, in `config/services.yaml` file:

```yaml
parameters:
    # ...
    jobs_directory: '%kernel.project_dir%/public/uploads/jobs'
```

And to add the processing logic in `JobController`:

```php
// ...
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\File\UploadedFile;

class JobController extends Controller
{
    // ...
    public function createAction(Request $request, EntityManagerInterface $em) : Response
    {
        // ...

        if ($form->isSubmitted() && $form->isValid()) {
            /** @var UploadedFile|null $logoFile */
            $logoFile = $form->get('logo')->getData();

            if ($logoFile instanceof UploadedFile) {
                $fileName = \bin2hex(\random_bytes(10)) . '.' . $file->guessExtension();

                // moves the file to the directory where brochures are stored
                $logoFile->move(
                    $this->getParameter('jobs_directory'),
                    $fileName
                );

                $job->setLogo($fileName);
            }

            $em->persist($job);
            $em->flush();

            return $this->redirectToRoute('job.list');
        }

        // ...
    }
}
```

Notice that now `JobController` extends `Controller` and not `AbstractController` because we need `getParameter` method.


Even if this implementation works, let’s do this in a better way, moving logic to service and using Doctrine lifecycle callbacks.

To create a service, first create a new `FileUploader` class in `src/Service` folder:

```php
namespace App\Service;

use Symfony\Component\HttpFoundation\File\UploadedFile;

class FileUploader
{
    /** @var string */
    private $targetDirectory;

    /**
     * @param string $targetDirectory
     */
    public function __construct(string $targetDirectory)
    {
        $this->targetDirectory = $targetDirectory;
    }

    /**
     * @param UploadedFile $file
     *
     * @return string
     */
    public function upload(UploadedFile $file) : string
    {
        $fileName = md5(uniqid()) . '.' . $file->guessExtension();

        $file->move($this->targetDirectory, $fileName);

        return $fileName;
    }
}
```

Then, define a service for this class in `config/services.yaml`:

```yaml
# ...

services:
    # ...

    App\Service\FileUploader:
        arguments:
            $targetDirectory: '%jobs_directory%'
```

Now you're ready to use this service in the controller:

```php
// ...
use App\Service\FileUploader;

class JobController extends AbstractController
{
    // ...
    public function createAction(Request $request, EntityManagerInterface $em, FileUploader $fileUploader) : Response
    {
        // ...

        if ($form->isSubmitted() && $form->isValid()) {
            /** @var UploadedFile|null $logoFile */
            $logoFile = $form->get('logo')->getData();

            if ($logoFile instanceof UploadedFile) {
                $fileName = $fileUploader->upload($logoFile);

                $job->setLogo($fileName);
            }

            $em->persist($job);
            $em->flush();

            return $this->redirectToRoute('job.list');
        }

        // ...
    }
}
```

Now create a [Doctrine listener][17] to automatically upload the file when persisting the entity (`src/EventListener/JobUploadListener.php`):

```php
namespace App\EventListener;

use App\Entity\Job;
use App\Service\FileUploader;
use Symfony\Component\HttpFoundation\File\UploadedFile;
use Doctrine\ORM\Event\LifecycleEventArgs;
use Doctrine\ORM\Event\PreUpdateEventArgs;

class JobUploadListener
{
    /** @var FileUploader */
    private $uploader;

    /**
     * @param FileUploader $uploader
     */
    public function __construct(FileUploader $uploader)
    {
        $this->uploader = $uploader;
    }

    /**
     * @param LifecycleEventArgs $args
     */
    public function prePersist(LifecycleEventArgs $args)
    {
        $entity = $args->getEntity();

        $this->uploadFile($entity);
    }

    /**
     * @param PreUpdateEventArgs $args
     */
    public function preUpdate(PreUpdateEventArgs $args)
    {
        $entity = $args->getEntity();

        $this->uploadFile($entity);
    }

    /**
     * @param $entity
     */
    private function uploadFile($entity)
    {
        // upload only works for Job entities
        if (!$entity instanceof Job) {
            return;
        }

        $logoFile = $entity->getLogo();

        // only upload new files
        if ($logoFile instanceof UploadedFile) {
            $fileName = $this->uploader->upload($logoFile);

            $entity->setLogo($fileName);
        }
    }
}
```

Next, register this class as a Doctrine listener in `config/services.yaml`:

```yaml
# ...

services:
    # ...

    App\EventListener\JobUploadListener:
        tags:
            - { name: doctrine.event_listener, event: prePersist }
            - { name: doctrine.event_listener, event: preUpdate }
```

This listener is now automatically executed when persisting a new Job entity.
This way, you can remove everything related to uploading from the controller.

Note that this will not work, until methods `setLogo` and `getLogo` from `Job` entity are forced to work with string.
Remove this constraint and it will work:

```php
/**
 * @return string|null|UploadedFile
 */
public function getLogo()
{
    return $this->logo;
}

/**
 * @param string|null|UploadedFile $logo
 *
 * @return self
 */
public function setLogo($logo) : self
{
    $this->logo = $logo;

    return $this;
}
```

## Protecting the Job Form with a Token

Everything must work fine by now. As of now, the user must enter the token for the job.
But the job token must be generated automatically when a new job is created, as we don’t want to rely on the user to provide a unique token. 

Create a new listener `JobTokenListener.php` in `src/EventListener` folder to add the logic that generates the token before a new job is saved:

```php
namespace App\EventListener;

use App\Entity\Job;
use Doctrine\ORM\Event\LifecycleEventArgs;

class JobTokenListener
{
    /**
     * @param LifecycleEventArgs $args
     */
    public function prePersist(LifecycleEventArgs $args)
    {
        $entity = $args->getEntity();

        if (!$entity instanceof Job) {
            return;
        }

        if (!$entity->getToken()) {
            $entity->setToken(\bin2hex(\random_bytes(10)));
        }
    }
}
```

define it in `config/services.yaml`:

```yaml
#...

services:
    # ...

    App\EventListener\JobTokenListener:
        tags:
            - { name: doctrine.event_listener, event: prePersist }
```

Now you can remove the token field from the form by deleting the `add(‘token’)` line.

If you remember the user stories from day 2, a job can be edited only if the user knows the associated token.

## The Preview Page

## Job Activation and Publication

## Additional information
- [Forms][4]
- [Form Component][3]
- [How to Upload Files][1]
- [How to Customize Form Rendering][2]
- [Form Types Reference][5]
- [Securely Generating Random Values][18]

## Next Steps

Continue this tutorial here: [Jobeet Day 9: -](/days/day-9.md)

Previous post is available here: [Jobeet Day 7: Playing with the Category Page](/days/day-7.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/4.0/controller/upload_file.html
[2]: https://symfony.com/doc/4.0/form/form_customization.html
[3]: https://symfony.com/doc/4.0/components/form.html
[4]: https://symfony.com/doc/4.0/forms.html
[5]: https://symfony.com/doc/4.0/reference/forms/types.html
[6]: https://symfony.com/doc/4.0/reference/forms/types/text.html
[7]: https://symfony.com/doc/4.0/reference/forms/types/email.html
[8]: https://symfony.com/doc/4.0/reference/forms/types/datetime.html
[9]: https://symfony.com/doc/4.0/reference/forms/types/choice.html
[10]: https://symfony.com/doc/4.0/reference/forms/types/entity.html
[11]: https://getbootstrap.com/docs/3.3/css/#forms
[12]: https://github.com/symfony/twig-bridge/tree/v4.0.6
[13]: https://github.com/symfony/twig-bridge/tree/v4.0.6/Resources/views/Form
[14]: https://symfony.com/doc/4.0/reference/forms/types/submit.html
[15]: https://symfony.com/doc/4.0/best_practices/forms.html#form-button-configuration
[16]: https://symfony.com/doc/4.0/reference/constraints.html
[17]: https://symfony.com/doc/4.0/doctrine/event_listeners_subscribers.html
[18]: https://symfony.com/doc/4.0/components/security/secure_tools.html
