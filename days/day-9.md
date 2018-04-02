# Jobeet Day 9: Console Commands

The Symfony provides lots of commands through the `bin/console` script (e.g. the well-known `bin/console cache:clear` command).
These commands are created with the [Console component][1]. You can also use it to create your own commands.

## Creating a Command

Commands are defined in classes extending `Symfony\Component\Console\Command\Command` and usually placed in `src/Command` folder.
For example, you may want a command to create a category:

```php
namespace App\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class CreateCategoryCommand extends Command
{
    protected function configure()
    {
        // ...
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // ...
    }
}
```

## Configuring the Command

First of all, you must configure the name of the command in the `configure()` method.
Then you can optionally define a help message and the input options and arguments:

```php
// ...
    protected function configure()
    {
        $this
            // the name of the command (the part after "bin/console")
            ->setName('app:create-category')

            // the short description shown while running "php bin/console list"
            ->setDescription('Creates a new category.')

            // the full command description shown when running the command with
            // the "--help" option
            ->setHelp('This command allows you to add new category in db...');
    }
// ....
```

## Registering the Command

We are using symfony 4 and it automatically registers console commands by parent class.
Be sure that [autoconfiguration][3] is activated in `config/services.yaml`:

```yaml
services:
    _defaults:
        autoconfigure: true
```

In lower versions console commands were registered by adding `twig.extension` tag.
Read more about it [here][4].

## Executing the Command

After configuring and registering the command, you can execute it in the terminal:

```yaml
php bin/console app:create-category
```

As you might expect, this command will do nothing as you didn't write any logic yet.
Add your own logic inside the `execute()` method, which has access to the input stream (e.g. options and arguments) and the output stream (to write messages to the console):

```php
// ...

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // outputs multiple lines to the console (adding "\n" at the end of each line)
        $output->writeln([
            'Category Creator',
            '============',
            '',
        ]);

        // outputs a message followed by a "\n"
        $output->writeln('Whoa!');

        // outputs a message without adding a "\n" at the end of the line
        $output->write('You are about to ');
        $output->write('create a category.');
    }
    
// ...
```

Now, try executing the command:

```
$ bin/console app:create-category
Category Creator
============

Whoa!
You are about to create a category.
```

## Console Input

Use input options or arguments to pass information to the command:

```php
// ...
use Symfony\Component\Console\Input\InputArgument;

class CreateCategoryCommand extends Command
{
    protected function configure()
    {
        $this
            // ...

            // configure an argument
            ->addArgument('name', InputArgument::REQUIRED, 'The name of the category.');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // outputs multiple lines to the console (adding "\n" at the end of each line)
        $output->writeln([
            'Category Creator',
            '============',
            '',
        ]);

        // retrieve the argument value using getArgument()
        $output->writeln('Name: '.$input->getArgument('name'));
    }
}
```

Now, you can pass the name to the command:

```
$ bin/console app:create-category Tester
Category Creator
============

Name: Tester
```

## Additional information
- [The Console Component][1]
- [Console Input (Arguments & Options)][2]

## Next Steps

Continue this tutorial here: [Jobeet Day 10: Translations](/days/day-10.md)

Previous post is available here: [Jobeet Day 8: The Forms](/days/day-8.md)

Main page is available here: [Symfony 4.0 Jobeet Tutorial](/README.md)

[1]: https://symfony.com/doc/4.0/components/console.html
[2]: https://symfony.com/doc/4.0/console/input.html
[3]: https://symfony.com/doc/4.0/service_container.html#services-autoconfigure
[4]: https://symfony.com/doc/4.0/service_container/tags.html
