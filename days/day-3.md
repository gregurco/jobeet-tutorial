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
    private $public;
    private $activated;
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
    private $active;
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
 * @ORM\Table(name="categories")
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
 * @ORM\Table(name="jobs")
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
     * @var bool
     *
     * @ORM\Column(type="boolean")
     */
    private $public;

    /**
     * @var bool
     *
     * @ORM\Column(type="boolean")
     */
    private $activated;

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

    /**
     * @var Category
     *
     * @ORM\ManyToOne(targetEntity="Category", inversedBy="jobs")
     * @ORM\JoinColumn(name="category_id", referencedColumnName="id", nullable=false)
     */
    private $category;
}
```

src/Entity/Affiliate.php:
```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="affiliates")
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
     * @var bool
     *
     * @ORM\Column(type="boolean")
     */
    private $active;

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
 * @ORM\Table(name="categories")
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
 * @ORM\Table(name="affiliates")
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

As you can notice, all properties are `private` but we should access them somehow. Let's generate setters and getters.
If you use an IDE like PhpStorm, it can generate these for you. In PhpStorm, put your cursor anywhere in the class, then go to the Code -> Generate menu and select "Getters and Setters" (Alt + Insert).
Notice #1: for boolean variables we generate `is` and `set` methods, for collections we generate `get`, `add` and `remove` but for everyone else we generate `get` and `set` methods.
Notice #2: for id we generate ONLY getter. We don't have case when to set id, but doctrine knows how to set id without setter.

src/Entity/Category.php:
```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="categories")
 */
class Category
{
    // properties
    
    // constructor

    /**
     * @return int
     */
    public function getId() : ?int
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function getName() : ?string
    {
        return $this->name;
    }

    /**
     * @param string $name
     *
     * @return self
     */
    public function setName(string $name) : self
    {
        $this->name = $name;

        return $this;
    }

    /**
     * @return Job[]|ArrayCollection
     */
    public function getJobs()
    {
        return $this->jobs;
    }

    /**
     * @param Job $job
     *
     * @return self
     */
    public function addJob(Job $job) : self
    {
        if (!$this->jobs->contains($job)) {
            $this->jobs->add($job);
        }

        return $this;
    }

    /**
     * @param Job $job
     *
     * @return self
     */
    public function removeJob(Job $job) : self
    {
        $this->jobs->removeElement($job);

        return $this;
    }

    /**
     * @return Affiliate[]|ArrayCollection
     */
    public function getAffiliates()
    {
        return $this->affiliates;
    }

    /**
     * @param Affiliate $affiliate
     *
     * @return self
     */
    public function addAffiliate($affiliate) : self
    {
        if (!$this->affiliates->contains($affiliate)) {
            $this->affiliates->add($affiliate);
        }

        return $this;
    }

    /**
     * @param Affiliate $affiliate
     *
     * @return self
     */
    public function removeAffiliate($affiliate) : self
    {
        $this->affiliates->removeElement($affiliate);

        return $this;
    }
}
```

src/Entity/Job.php:
```php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="jobs")
 */
class Job
{
    // properties

    /**
     * @return int
     */
    public function getId() : ?int
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function getType() : ?string
    {
        return $this->type;
    }

    /**
     * @param string $type
     *
     * @return self
     */
    public function setType(string $type) : self
    {
        $this->type = $type;

        return $this;
    }

    /**
     * @return string
     */
    public function getCompany() : ?string
    {
        return $this->company;
    }

    /**
     * @param string $company
     *
     * @return self
     */
    public function setCompany(string $company) : self
    {
        $this->company = $company;

        return $this;
    }

    /**
     * @return string|null
     */
    public function getLogo() : ?string
    {
        return $this->logo;
    }

    /**
     * @param string|null $logo
     *
     * @return self
     */
    public function setLogo(?string $logo) : self
    {
        $this->logo = $logo;

        return $this;
    }

    /**
     * @return string|null
     */
    public function getUrl() : ?string
    {
        return $this->url;
    }

    /**
     * @param string|null $url
     *
     * @return self
     */
    public function setUrl(?string $url) : self
    {
        $this->url = $url;

        return $this;
    }

    /**
     * @return string
     */
    public function getPosition() : ?string
    {
        return $this->position;
    }

    /**
     * @param string $position
     *
     * @return self
     */
    public function setPosition(string $position) : self
    {
        $this->position = $position;

        return $this;
    }

    /**
     * @return string
     */
    public function getLocation() : ?string
    {
        return $this->location;
    }

    /**
     * @param string $location
     *
     * @return self
     */
    public function setLocation(string $location) : self
    {
        $this->location = $location;

        return $this;
    }

    /**
     * @return string
     */
    public function getDescription() : ?string
    {
        return $this->description;
    }

    /**
     * @param string $description
     *
     * @return self
     */
    public function setDescription(string $description) : self
    {
        $this->description = $description;

        return $this;
    }

    /**
     * @return string
     */
    public function getHowToApply() : ?string
    {
        return $this->howToApply;
    }

    /**
     * @param string $howToApply
     *
     * @return self
     */
    public function setHowToApply(string $howToApply) : self
    {
        $this->howToApply = $howToApply;

        return $this;
    }

    /**
     * @return string
     */
    public function getToken() : ?string
    {
        return $this->token;
    }

    /**
     * @param string $token
     *
     * @return self
     */
    public function setToken(string $token) : self
    {
        $this->token = $token;

        return $this;
    }

    /**
     * @return bool
     */
    public function isPublic() : ?bool
    {
        return $this->public;
    }

    /**
     * @param bool $public
     *
     * @return self
     */
    public function setPublic(bool $public) : self
    {
        $this->public = $public;

        return $this;
    }

    /**
     * @return bool
     */
    public function isActivated() : ?bool
    {
        return $this->activated;
    }

    /**
     * @param bool $activated
     *
     * @return self
     */
    public function setActivated(bool $activated) : self
    {
        $this->activated = $activated;

        return $this;
    }

    /**
     * @return string
     */
    public function getEmail() : ?string
    {
        return $this->email;
    }

    /**
     * @param string $email
     *
     * @return self
     */
    public function setEmail(string $email) : self
    {
        $this->email = $email;

        return $this;
    }

    /**
     * @return \DateTime
     */
    public function getExpiresAt() : ?\DateTime
    {
        return $this->expiresAt;
    }

    /**
     * @param \DateTime $expiresAt
     *
     * @return self
     */
    public function setExpiresAt(\DateTime $expiresAt) : self
    {
        $this->expiresAt = $expiresAt;

        return $this;
    }

    /**
     * @return \DateTime
     */
    public function getCreatedAt() : ?\DateTime
    {
        return $this->createdAt;
    }

    /**
     * @return \DateTime
     */
    public function getUpdatedAt() : ?\DateTime
    {
        return $this->updatedAt;
    }

    /**
     * @return Category
     */
    public function getCategory() : ?Category
    {
        return $this->category;
    }

    /**
     * @param Category $category
     *
     * @return self
     */
    public function setCategory(Category $category) : self
    {
        $this->category = $category;

        return $this;
    }
}
```

src/Entity/Affiliate.php:
```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="affiliates")
 */
class Affiliate
{
    // properties
    
    // constructor

    /**
     * @return int
     */
    public function getId() : ?int
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function getUrl() : ?string
    {
        return $this->url;
    }

    /**
     * @param string $url
     *
     * @return self
     */
    public function setUrl(string $url) : self
    {
        $this->url = $url;

        return $this;
    }

    /**
     * @return string
     */
    public function getEmail() : ?string
    {
        return $this->email;
    }

    /**
     * @param string $email
     *
     * @return self
     */
    public function setEmail(string $email) : self
    {
        $this->email = $email;

        return $this;
    }

    /**
     * @return string
     */
    public function getToken() : ?string
    {
        return $this->token;
    }

    /**
     * @param string|null $token
     *
     * @return self
     */
    public function setToken(?string $token) : self
    {
        $this->token = $token;

        return $this;
    }

    /**
     * @return bool
     */
    public function isActive() : ?bool
    {
        return $this->active;
    }

    /**
     * @param bool $active
     *
     * @return self
     */
    public function setActive(bool $active) : self
    {
        $this->active = $active;

        return $this;
    }

    /**
     * @return \DateTime
     */
    public function getCreatedAt() : ?\DateTime
    {
        return $this->createdAt;
    }

    /**
     * @param \DateTime $createdAt
     *
     * @return self
     */
    public function setCreatedAt(\DateTime $createdAt) : self
    {
        $this->createdAt = $createdAt;

        return $this;
    }

    /**
     * @return Category[]|ArrayCollection
     */
    public function getCategories()
    {
        return $this->categories;
    }

    /**
     * @param Category[]|ArrayCollection $categories
     *
     * @return self
     */
    public function setCategories($categories) : self
    {
        $this->categories = $categories;

        return $this;
    }
}
```

## Additional information
- [Databases and the Doctrine ORM][1]
- [How to Work with Doctrine Associations / Relations][2]

## Next Steps

Continue this tutorial here: [Jobeet Day 4: The Controller and the View](/days/day-4.md)

Previous post is available here: [Jobeet Day 2: The Project](/days/day-2.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/4.0/doctrine.html
[2]: https://symfony.com/doc/4.0/doctrine/associations.html
