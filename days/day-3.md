# Jobeet Day 3: The Data Model

## The Relational Model
The user stories from the previous day describe the main objects of our project: jobs, affiliates, and categories. Here is the corresponding entity relationship diagram:

![DB schema](/files/images/screenshot_6.png)

In addition to the columns described in the stories, we have also added `created_at` and `updated_at` columns. We will configure Symfony 4 to set their value automatically when an object is saved or updated.

## The Database
To store the jobs, affiliates and categories in the database, Symfony 4 uses Doctrine ORM. To define the database connection parameters you have to edit the `.env` file (for this tutorial we will use MySQL).
```dotenv
DATABASE_URL=mysql://user:password@mysql:3306/jobeet
```
Now that Doctrine knows about your database, you can have it create the database for you (if you did not already created it):

```bash
bin/console doctrine:database:create --if-not-exists
```

## Creating Entity Classes

For each type of object we need, we will create an entity class (just a simple PHP class with some properties).

src/Entity/Category.php:
```php
namespace App\Entity;

class Category
{
    private $id;
    private $name;
}
```

src/Entity/Job.php:
```php
namespace App\Entity;

class Job
{
    private $id;
    private $category;
    private $type;
    private $company;
    private $logo;
    private $url;
    private $position;
    private $location;
    private $description;
    private $howToApply;
    private $token;
    private $isPublic;
    private $isActivated;
    private $email;
    private $expiresAt;
    private $createdAt;
    private $updatedAt;
}
```

src/Entity/Affiliate.php:
```php
namespace AppBundle\Entity;

class Affiliate
{
    private $id;
    private $categories;
    private $url;
    private $email;
    private $token;
    private $isActive;
    private $createdAt;
}
```

## Adding Mapping Information

To tell Doctrine about our objects, we will create “metadata” that will describe how our objects will be stored in the database. We will use annotations for this project but you can also use YAML or XML files to achieve the same result.

src/Entity/Category.php:
```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="category")
 */
class Category
{
    /**
     * @var int
     *
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=100)
     */
    private $name;

    /**
     * @var Job[]|ArrayCollection
     *
     * @ORM\OneToMany(targetEntity="Job", mappedBy="category")
     */
    private $jobs;

    /**
     * @var Affiliate[]|ArrayCollection
     *
     * @ORM\ManyToMany(targetEntity="Affiliate", mappedBy="categories")
     */
    private $affiliates;
}
```

src/Entity/Job.php:
```php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="job")
 */
class Job
{
    /**
     * @var int
     *
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @var Category|null
     *
     * @ORM\ManyToOne(targetEntity="Category", inversedBy="jobs")
     * @ORM\JoinColumn(name="category_id", referencedColumnName="id")
     */
    private $category;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255)
     */
    private $type;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255)
     */
    private $company;

    /**
     * @var string|null
     *
     * @ORM\Column(type="string", length=255, nullable=true)
     */
    private $logo;

    /**
     * @var string|null
     *
     * @ORM\Column(type="string", length=255, nullable=true)
     */
    private $url;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255)
     */
    private $position;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255)
     */
    private $location;

    /**
     * @var string
     *
     * @ORM\Column(type="text")
     */
    private $description;

    /**
     * @var string
     *
     * @ORM\Column(type="text")
     */
    private $howToApply;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255, unique=true)
     */
    private $token;

    /**
     * @var bool|null
     *
     * @ORM\Column(type="boolean", nullable=true)
     */
    private $isPublic;

    /**
     * @var bool|null
     *
     * @ORM\Column(type="boolean", nullable=true)
     */
    private $isActivated;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255)
     */
    private $email;

    /**
     * @var \DateTime
     *
     * @ORM\Column(type="datetime")
     */
    private $expiresAt;

    /**
     * @var \DateTime
     *
     * @ORM\Column(type="datetime")
     */
    private $createdAt;

    /**
     * @var \DateTime
     *
     * @ORM\Column(type="datetime")
     */
    private $updatedAt;
}
```

src/Entity/Affiliate.php:
```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="affiliate")
 */
class Affiliate
{
    /**
     * @var int
     *
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255)
     */
    private $url;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255)
     */
    private $email;

    /**
     * @var string
     *
     * @ORM\Column(type="string", length=255, unique=true)
     */
    private $token;

    /**
     * @var bool|null
     *
     * @ORM\Column(type="boolean", nullable=true)
     */
    private $isActive;

    /**
     * @var \DateTime
     *
     * @ORM\Column(type="datetime")
     */
    private $createdAt;
    
    /**
     * @var Category[]|ArrayCollection
     *
     * @ORM\ManyToMany(targetEntity="Category", inversedBy="affiliates")
     * @ORM\JoinTable(name="affiliates_categories")
     */
    private $categories;
}
```

After creating the entities we can validate the mappings with the following command:
```bash
bin/console doctrine:schema:validate
```

This is what you should get:
```bash
Mapping
-------

 [OK] The mapping files are correct.

Database
--------

 [ERROR] The database schema is not in sync with the current mapping file.
```

Don’t worry about that error for now. We will fix it in a few minutes.

## Constructors, Getters and Setters

Fists of all we should create constructors in entities with OneToMany or ManyToMany relations.
Collection property, such as $categories, must be a collection object that implements Doctrine's Collection interface. In this case, an ArrayCollection object is used.
This looks and acts almost exactly like an array, but has some added flexibility. Just imagine that it's an array and you'll be in good shape.

src/Entity/Category.php:
```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="category")
 */
class Category
{
    // properties
    
    public function __construct()
    {
        $this->jobs = new ArrayCollection();
        $this->affiliates = new ArrayCollection();
    }
    
    // setters and getters
}
```

src/Entity/Affiliate.php:
```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="affiliate")
 */
class Affiliate
{
    // properties
    
    public function __construct()
    {
        $this->categories = new ArrayCollection();
    }
    
    // setters and getters
}
```

## Additional information
- [Databases and the Doctrine ORM][1]

## Next Steps

Continue this tutorial here: [Jobeet Day 4: The Controller and the View](/days/day-4.md)

Previous post is available here: [Jobeet Day 2: The Project](/days/day-2.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/4.0/doctrine.html
