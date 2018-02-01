# Jobeet Day 1: Starting up the Project

Today we will setup the development environment, install Symfony 4.0 and display a page of the application in the web browser. First of all, we need to have a friendly working environment for web development. We will use [Docker][1] with nginx and php images from [Docker Hub][2]. To check the minimum requirements for running Symfony 2.8 you can access this [link][3].

[1]: https://www.docker.com/
[2]: https://hub.docker.com/
[3]: https://symfony.com/doc/4.0/reference/requirements.html

## Docker configuration
After you install [Docker][1] and [Docker Compose][2], create a new folder named jobeet and unarchive there next [archive](/files/archives/jobeet.zip).
It is initial configuration for generic php project. It includes PHP 7.2, MySQL 5.7 and nginx. To start containers just run next command:
```bash
docker-compose up -d
```
After several minutes containers will be raised and you can check it with next command:
```bash
docker-compose ps
```
the result should be the same:
```bash
      Name                   Command             State           Ports          
-------------------------------------------------------------------------------
jobeet-mysql       docker-entrypoint.sh mysqld   Up      0.0.0.0:3306->3306/tcp 
jobeet-php-fpm     /bin/sh -c /usr/bin/php-fpm   Up      9000/tcp               
jobeet-webserver   nginx -g daemon off;          Up      0.0.0.0:80->80/tcp
```

Congratulations! Now you have prepared environment for jobeet project.

## Download and install symfony 4.0
To create our Symfony project we will use method described in official [documentation][3].
First of all, enter in php container:
```bash
docker-compose exec php-fpm bash
```

After that create symfony project in `tmp` _(temporary)_ folder, copy it to your project folder, exit from container and change permission of files to be able to edit them from your IDE:
```bash
composer create-project symfony/website-skeleton:4.0 /tmp/jobeet/
cp -aR /tmp/jobeet/. .
exit;
sudo chown -R $USER:$USER .
```

## Test the Symfony installation

Now open your web browser and enter the [http://127.0.0.1][4] URL. You should see error `No route found for "GET /"` but it's ok. You don't have any routes created yet.

## Symfony console

Symfony 4 comes with the console component tool that you will use for different tasks. To see a list of things it can do for you type at the command prompt:

```bash
bin/console list
```
_Note: don't forget to execute this command from php container_

## The Environments

Symfony 4 has different environments. If you look in the project’s directory, you will see file `.env` with variable `APP_ENV=dev` inside.
Value `prod` is for production environment and `dev` is used by web developers when they work on the application in the development environment. The development environment will prove very handy because it will show you all the errors and warnings and the Web Debug Toolbar — the developer’s best friend. Check the development environment by accessing [http://127.0.0.1][4] in your browser (note the bottom debug toolbar).

![Debug toolbar](/files/images/screenshot_1.png)

That’s all for today. You can find the code from this day here: [https://github.com/gregurco/jobeet/tree/day1][5]. See you on the next day of this tutorial when we will talk about what exactly the Jobeet website will be about!

## Next Steps

Continue this tutorial here: ~~Jobeet Day 2: The Project~~

[1]: https://docs.docker.com/install/linux/docker-ce/ubuntu/
[2]: https://docs.docker.com/compose/install/
[3]: https://symfony.com/doc/4.0/setup.html
[4]: http://127.0.0.1
[5]: https://github.com/gregurco/jobeet/tree/day1
