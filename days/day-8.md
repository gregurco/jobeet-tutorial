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

## The Form Template

## Form processing

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
