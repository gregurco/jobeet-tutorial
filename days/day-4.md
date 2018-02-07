# Jobeet Day 4: The Controller and the View

## The MVC Architecture
For web development, the most common solution for organizing your code nowadays is the [MVC design pattern][1]. In short, the MVC design pattern defines a way to organize your code according to its nature. This pattern separates the code into **three layers**:

- The **Model** layer defines the business logic (the database belongs to this layer). You already know that Symfony stores all the classes and files related to the Model in the `src/Entity/` directory of your bundles.
- The **View** is what the user interacts with (a template engine is part of this layer). In Symfony, the View layer is mainly made of Twig templates. They are stored in various `templates/` directories as we will see later.
- The **Controller** is a piece of code that calls the Model to get some data that it passes to the View for rendering to the client. When we installed Symfony at the beginning of this tutorial, we saw that all requests are managed by front controller (`public/index.php`). This front controller delegate the real work to actions.

## Additional information

## Next Steps

Continue this tutorial here: [Jobeet Day 5: The Routing](/days/day-5.md)

Previous post is available here: [Jobeet Day 3: The Data Model](/days/day-3.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
