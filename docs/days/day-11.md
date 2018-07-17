# Jobeet Day 11: The User

Yesterday was packed with a lot of information.

Today, we will discover how symfony manages persistent data between HTTP requests. As you might know, the HTTP protocol is stateless, which means that each request is independent from its preceding or proceeding ones. Modern websites need a way to persist data between requests to enhance the user experience.

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

## Additional information
- [Session Management][1]

## Next Steps

Continue this tutorial here: Jobeet Day 12: The Mailer

Previous post is available here: [Jobeet Day 10: The Admin](day-10.md)

Main page is available here: [Symfony 4.1 Jobeet Tutorial](../index.md)

[1]: http://symfony.com/doc/4.1/components/http_foundation/sessions.html
