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

## Additional information
- [The Symfony MakerBundle][1]

## Next Steps

Continue this tutorial here: [Jobeet Day 11: The User](/days/day-11.md)

Previous post is available here: [Jobeet Day 9: Console Commands](/days/day-9.md)

Main page is available here: [Symfony 4.1 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/1.0/bundles/SymfonyMakerBundle/index.html
