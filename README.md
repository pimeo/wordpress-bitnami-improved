## Project goals

- This repository builds on Bitnami's great work on Wordpress/Nginx, adding the ability to dynamically modify certain variables in the `wp-config.php` file from an auto-generated `.env` file for the project.
- Supports https with reverse proxy configurations.
- It can be used to modify nginx, php-fpm and php configurations. 
- A set of makefile commands simplify installation and configuration of the project.


Note: This project is for use with docker-compose.

## Table of contents

- [Project goals](#project-goals)
- [Table of contents](#table-of-contents)
- [Arborescence originale](#arborescence-originale)
- [Requirements](#requirements)
  - [Debian distribution](#debian-distribution)
  - [Macos](#macos)
- [Project installation](#project-installation)
  - [Standard installation](#standard-installation)
  - [Customize Wordpress configurations](#customize-wordpress-configurations)
    - [Environment variables](#environment-variables)
    - [Fichiers de configuration bitnami](#fichiers-de-configuration-bitnami)
    - [Utiliser son propre fichier wp-config.php](#utiliser-son-propre-fichier-wp-configphp)
- [Acces](#acces)
- [Administration](#administration)
- [Update protocol and host in mariadb database](#update-protocol-and-host-in-mariadb-database)
- [Troubleshooting](#troubleshooting)
  - [Permissions problem on wordpress and mariadb directories](#permissions-problem-on-wordpress-and-mariadb-directories)
- [Todolist](#todolist)


## Arborescence originale
```
tree -L 3
.
├── README.md
├── bitnami
│   ├── nginx
│   │   └── nginx.conf
│   ├── php
│   │   └── php.ini
│   └── php-fpm
│       └── www.conf
├── docker-compose.yml
├── docker-compose.yml.bak
├── makefile
├── wordpress-orig
│   ├── composer.json
│   ├── composer.lock
│   └── wp-config.php
└── wordpress
    ├── vendor
    │   ├── autoload.php
    │   ├── composer
    │   ├── graham-campbell
    │   ├── phpoption
    │   ├── symfony
    │   └── vlucas
    ├── wp-config.php
    ├── wp-config.php.bak
    └── wp-content
        ├── index.php
        ├── languages
        ├── plugins
        ├── themes
        ├── upgrade
        └── uploads
```

## Requirements


### Debian distribution

```
apt update
apt install build-essential
```

### Macos

```
xcode-select --install
```

## Project installation

### Standard installation

```sh
cp .env.example .env # Complete variables with credentials
make create_bitnami_user configure_persistent_binded_volumes
docker compose up --wait --force-recreate --remove-orphans -d
```

### Customize Wordpress configurations

#### Environment variables

| Variables    | Default | Accepted values |
| -------- | ------- | ------- |
| WORDPRESS_HOST_PROTOCOL | http | http, https |
| WORDPRESS_HOST_DOMAIN | (empty) | Ip or domain or subdomain. It serves $_SERVER['HTTP_HOST'] if empty. |
| WORDPRESS_DEBUG | false | true, false |


#### Fichiers de configuration bitnami

It is possible to modify the configurations of php, nginx and php-fpm, located in the `bitnami` directory. The containers must be stopped and restarted to take the changes into account.

#### Utiliser son propre fichier wp-config.php

```sh
make customize_wordpress
```

Warning: the `.env` file is auto-generated and will overwrite the values in the previous file. Remember to make a backup before running the command.

## Acces

```
http://localhost/
https://localhost/
```

## Administration

```
http://localhost/wp-admin
user: user
password: bitnami
```

## Update protocol and host in mariadb database

If you wish to set a different domain name for your Wordpress instance, you need to follow the commands: 

1. Set to the `.env` file the setting `WORDPRESS_HOST_DOMAIN`
2. Run command `make customize_wordpress`
3. Connect to mariadb service container instance to change in database the URL to replace with your new domain. Theses commands are available below.


```sh
docker exec -it wordpress_docker_db_2 mariadb -u user -p  # password set in .env file
use wordpress_database;

# Ensure url need to be changed
SELECT option_name, option_value FROM wp_options WHERE option_name in ('home', 'siteurl');


# Queries to apply to replace the new url
UPDATE wp_options SET option_value = replace(option_value, 'http://oldurl.com', 'https://newurl.com') WHERE option_name = 'home' OR option_name = 'siteurl';
UPDATE wp_posts SET guid = replace(guid, 'http://oldurl.com','https://newurl.com');
UPDATE wp_posts SET post_content = replace(post_content, 'http://oldurl.com', 'https://newurl.com'); 
UPDATE wp_postmeta SET meta_value = replace(meta_value,'http://oldurl.com','https://newurl.com');
exit
```

## Troubleshooting

### Permissions problem on wordpress and mariadb directories

Error: `cp: cannot create regular file '/bitnami/wordpress/wp-config.php': Permission denied`

Run the following commands to create a bitnami user with the ability to write to the directory. It is recommended to add the session's active user to the bitnami group in order to be able to modify files.


```sh
make create_bitnami_user
make configure_persistent_binded_volumes
```


## Todolist
- [ ] Improve README documentation.