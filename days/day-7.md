# Jobeet Day 7: Playing with the Category Page

Today we will make the Category page like it is described in the second day’s requirements:
*“The user sees a list of all the jobs from the category sorted by date and paginated with 20 jobs per page“.*

## The Category Route

First, let’s think about the route we will use for our category page.
We will define a pretty URL that will contain the category slug: `/category/{slug}` named `category.show`.

## Category slug

To have the slug of categories we will need [StofDoctrineExtensionsBundle][1] that wraps [DoctrineExtensions][1] package.
It consists of different useful extensions but we will use [Sluggable][2] only for now.

First let's install the bundle:
```bash
composer require stof/doctrine-extensions-bundle
```

This bundle has recipe and symfony will ask you to run this recipe, because it's not official one. Type 'y' and accept it:
```bash
Symfony operations: 1 recipe (3c3199f3aa23ea62ee911b3d6fe61a93)
  -  WARNING  stof/doctrine-extensions-bundle (>=1.2): From github.com/symfony/recipes-contrib:master
    The recipe for this package comes from the "contrib" repository, which is open to community contributions.
    Do you want to execute this recipe?
    [y] Yes
    [n] No
    [a] Yes for all packages, only for the current installation session
    [p] Yes permanently, never ask again for this project
    (defaults to n): 
```

Read about [Flex][4] system to know more about recipes.

Activate `sluggable` extension in `config/packages/stof_doctrine_extensions.yaml`:
```yaml
stof_doctrine_extensions:
    default_locale: en_US
    orm:
        default:
            sluggable: true
```

Slug will be stored in DB and we need field for it. Add `slug` field in `Category` entity:
```php
// ...
use Gedmo\Mapping\Annotation as Gedmo;

class Category
{
    // ...

    /**
     * @var string
     *
     * @Gedmo\Slug(fields={"name"})
     *
     * @ORM\Column(type="string", length=128, unique=true)
     */
    private $slug;
    
    // ...
    
    /**
     * @return string|null
     */
    public function getSlug() : ?string
    {
        return $this->slug;
    }
    
    /**
     * @param string $slug
     */
    public function setSlug(string $slug): void
    {
        $this->slug = $slug;
    }
    
    // ...
}
```

Pay attention to `@Gedmo\Slug` annotation.

Generate migration that will add `slug` field in `category` table:
```bash
bin/console doctrine:schema:diff
```

If we run migration now, we will see error, because we have several categories in DB without slug, that is required.
First of all we should drop database:
```bash
bin/console doctrine:schema:drop --force --full-database
```

Run migrations:
```bash
bin/console doctrine:migration:migrate
```

Run fixtures:
```bash
bin/console doctrine:fixtures:load
```

And check that categories have slug:
```bash
bin/console doctrine:query:sql 'SELECT * from categories'
```

The result should be similar:
```bash
array(4) {
  [0]=>
  array(3) {
    ["id"]=>
    string(1) "1"
    ["name"]=>
    string(6) "Design"
    ["slug"]=>
    string(6) "design"
  }
...
```

The main advantage of this bundle for us is that slug is generated automatically. We don't call 'setSlug' anywhere.

## The Job Category Controller

It’s now time to create the category controller. Create a new `CategoryController.php` file in your Controller directory:

```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class CategoryController extends AbstractController
{

}
```

## The Category Page

## Including Other Twig Templates

## The Category Link

## List Pagination

## Additional information

## Next Steps

Continue this tutorial here: [Jobeet Day 8: The Forms](/days/day-8.md)

Previous post is available here: [Jobeet Day 6: More with the Entity](/days/day-6.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/current/bundles/StofDoctrineExtensionsBundle/index.html
[2]: https://github.com/Atlantic18/DoctrineExtensions
[3]: https://github.com/Atlantic18/DoctrineExtensions/blob/v2.4.x/doc/sluggable.md#setup-and-autoloading
[4]: https://symfony.com/doc/4.0/setup/flex.html
