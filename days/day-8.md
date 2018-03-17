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

## Validation

## Handling File Uploads

## Additional information

## Protecting the Job Form with a Token

## The Preview Page

## Job Activation and Publication


## Next Steps
- [Forms][4]
- [Form Component][3]
- [How to Upload Files][1]
- [How to Customize Form Rendering][2]
- [Form Types Reference][5]

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
