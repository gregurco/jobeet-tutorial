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

## Test the Symfony installation

## Symfony console

## The Environments

## Next Steps

Continue this tutorial here: ~~Jobeet Day 2: The Project~~

[1]: https://docs.docker.com/install/linux/docker-ce/ubuntu/
[2]: https://docs.docker.com/compose/install/
