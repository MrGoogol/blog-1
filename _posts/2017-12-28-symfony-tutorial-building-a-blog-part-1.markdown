---
layout: post
title: "Symfony Tutorial: Building a Blog (Part 1)"
description: "Let's create a secure blog engine with Symfony."
longdescription: "Creating applications with Symfony is easy and can be scaled to be used in any requirement. The tools that it provides to create and maintain web applications is amazing and replaces repetitive tasks. Let's use Symfony to create a blog engine."
date: 2017-12-28 08:30
category: Technical Guide, PHP, Symfony
author:
  name: Greg Holmes
  url: https://github.com/GregHolmes
  mail: iam@gregholmes.co.uk
  avatar: "https://avatars0.githubusercontent.com/u/2411269?s=460&v=4"
design:
  bg_color: "#000000"
  image: https://cdn.auth0.com/blog/symfony-blog/logo.png
tags:
- symfony
- authentication
- auth0
- php
related:
- 2017-07-26-creating-your-first-symfony-app-and-adding-authentication
- 2016-06-23-creating-your-first-laravel-app-and-adding-authentication
---

**TL;DR:** Symfony is a PHP framework as well as a set of reusable PHP components and libraries. It uses the Model-View-Controller design pattern, and can be scaled to be used in any requirement. It aims to speed up the creation and maintenance of web applications, replacing repetitive code. In this tutorial, I'll show you how to create your very own blog, with content stored in the database and authentication through Auth0. The finished code can be found at this [repository](https://github.com/GregHolmes/symfony-blog).

---

## Symfony Tutorial: Introduction

### What is Symfony?

[Symfony](https://symfony.com) is an Open Source PHP framework originally created by [Sensiolabs](https://sensiolabs.com/), an interactive agency. The [Symfony PHP framework](https://symfony.com) comes with a very large and active community. This framework has become very popular, mainly due to the vast number of reusable PHP components. These components are not limited to the framework itself but can be found throughout the PHP community, such as Drupal, WordPress, phpBB, and Laravel.

[Symfony Components](https://symfony.com/components) is a set of decoupled and reusable PHP libraries. They are becoming the standard foundation on which the best PHP applications are built on. You can use any of these components in your own applications independently from the Symfony Framework. Below you will find a list of some of the components that are used in this article as an example.

* [Asset](https://symfony.com/components/Asset) - Manages URL generation and versioning of web assets such as CSS stylesheets, JavaScript files and image files.
* [Config](https://symfony.com/components/Config) - Helps you find, load, combine, autofill and validate configuration values.
* [Console](https://symfony.com/components/Console) - Eases the creation of beautiful and testable command line interfaces.
* [Dotenv](https://symfony.com/components/Dotenv) - Parses `.env` files to make environment variables stored in them accessible via `getenv()`, `$_ENV` or `$_SERVER`.
* [Form](https://symfony.com/components/Form) - Provides tools to easy creating, processing and reusing HTML forms.
* [HttpKernel](https://symfony.com/components/HttpKernel) - Provides the building blocks to create flexible and fast HTTP-based frameworks.
* [Routing](https://symfony.com/components/Routing) - Maps an HTTP request to a set of configuration variables.
* [Security](https://symfony.com/components/Security) - Provides an infrastructure for sophisticated authorization systems.
* [Templating](https://symfony.com/components/Templating) - Provides all the tools needed to build any kind of template system.
* [Yaml](https://symfony.com/components/Yaml) - Loads and dumps YAML files.

### What Will We Build

In this article, we will be looking at how to install a new version of the Symfony PHP framework, along with making use of Doctrine to create two new database tables (`Author` and `BlogPosts`) in order to store our blog data in. Following this, we will be making use of Doctrine Migrations to pre-populate our newly created database tables with some dummy data to allow you to see the blog at work. Once the initial set up is complete, we will cover user authentication with Auth0 allowing users to create their own author entry in the database.

## Bootstrapping Symfony

### Installing Symfony

Install Symfony via [Composer](https://getcomposer.org/) with the following command:

```bash
composer create-project symfony/framework-standard-edition:"3.3.11" blog
```

Once Composer has finished downloading all the required third-party libraries, it will ask you to input a number of parameters. Please just leave these empty, pressing return on each line.

Change directory into your project with: `cd blog`

### Create & Populate DotEnv File

In your root directory create file called `.env` and paste the following into there:

```yml
DATABASE_HOST={DATABASE_HOST}
DATABASE_PORT=3306
DATABASE_NAME={DATABASE_NAME}
DATABASE_USER=root
DATABASE_PASSWORD={DATABASE_PASSWORD}
```

Then you have to replace any of the values with the correct settings for your MySQL database. For example, `{DATABASE_HOST}` could be replaced by `127.0.0.1`. Don't have MySQL installed? [An easy way to bootstrap one is with this script, it just needs Docker installed on the host machine](https://gist.github.com/brunokrebs/af77f582f0e650a62a6f06e84cd06f91).

Next. Let's find the `app/config/config.yml` file. Within there you'll find the following:

```yml
# Doctrine Configuration
doctrine:
    dbal:
        driver: pdo_mysql
        host: '%database_host%'
        port: '%database_port%'
        dbname: '%database_name%'
        user: '%database_user%'
        password: '%database_password%'
        charset: UTF8
```

Replace anything that is wrapped around `'%` and `%'` to contain the DotEnv configurations. So it will look like:

```yml
# Doctrine Configuration
doctrine:
    dbal:
        driver: pdo_mysql
        host: '%env(DATABASE_HOST)%'
        port: '%env(DATABASE_PORT)%'
        dbname: '%env(DATABASE_NAME)%'
        user: '%env(DATABASE_USER)%'
        password: '%env(DATABASE_PASSWORD)%'
        charset: UTF8
```

__NOTE__: All this does is retrieve the database details from the `.env` file rather than `app/config/parameters.yml`.

Now need to just make some minor configuration changes so that Symfony knows to read from the .env file.

Edit the `bin/console`, `web/app_dev.php`, `web/app.php` files. Above the line `$kernel = new AppKernel($env, $debug)` in each file, paste the following:

```php
try {
    (new \Symfony\Component\Dotenv\Dotenv())->load(__DIR__.'/../.env');
} catch (\Symfony\Component\Dotenv\Exception\PathException $e) {

}
```

If needed, run the following command `php bin/console doctrine:database:create`, which will create a database with the value of `DATABASE_NAME` in the `.env` file.

Now that the basic configuration has been set up, let's run the following command: `php bin/console server:start`.

You will see something similar to: `[OK] Server listening on http://127.0.0.1:8000`. So in your browser copy in that URL and you'll be shown a "Welcome to Symfony" page.

### Creating a New Author Entity

Create new `Author` entity by running the following command `php bin/console doctrine:generate:entity`

When it asks for the Entity shortcut name, type in: `AppBundle:Author`

Keep the default on `Configuration format`, but this next step we need to add all of the properties of our Author. Please refer to the image below for the entries required for an Author table:

![Creating an Author entity with Symfony and Doctrine](https://cdn.auth0.com/blog/symfony-blog/create-author-entity.png)

__NOTE__: If you cannot see the image, the full entity can be found [here](https://github.com/GregHolmes/symfony-blog/blob/master/part-1/src/AppBundle/Entity/Author.php) and its repository can be found [here](https://github.com/GregHolmes/symfony-blog/blob/master/part-1/src/AppBundle/Repository/AuthorRepository.php).

{% include tweet_quote.html quote_text="Doctrine is a great tool to have around when developing with PHP and Symfony." %}

### Creating a New BlogPost Entity

Create new `BlogPost` entity by running the following command `php bin/console doctrine:generate:entity`

When it asks for the Entity shortcut name, type in: `AppBundle:BlogPost`

Keep the default on `Configuration format`, but the next step is to add all of the properties on the BlogPost. Please refer to the image below for the entries required for a BlogPost table:

![Creating an Author entity with Symfony and Doctrine](https://cdn.auth0.com/blog/symfony-blog/create-blogpost-entity.png)

__NOTE__: If you cannot see the image, the full entity can be found [here](https://github.com/GregHolmes/symfony-blog/blob/master/part-1/src/AppBundle/Entity/BlogPost.php) and its repository can be found [here](https://github.com/GregHolmes/symfony-blog/blob/master/part-1/src/AppBundle/Repository/BlogPostRepository.php).

__NOTE__: If you copied the file directly from GitHub, please skip to the _Install Doctrine-Migrations_ section.

You may have noticed that we've added the `created_at`, `updated_at` and `author` fields. These fields all need some extra changes to be made in the entity file itself. So open `src/AppBundle/Entity/BlogPost.php`

Find `private $author;` and replace the annotation above this from:

```php
/**
 * @var int
 *
 * @ORM\Column(name="author", type="integer")
 */
```

to:

```php
/**
 * @var Author
 *
 * @ORM\ManyToOne(targetEntity="Author")
 * @ORM\JoinColumn(name="author_id", referencedColumnName="id")
 */
```

Next. Replace the Author getter and setter from:

```php
/**
 * Set author
 *
 * @param integer $author
 *
 * @return BlogPost
 */
public function setAuthor($author)
{
    $this->author = $author;

    return $this;
}

/**
 * Get author
 *
 * @return int
 */
public function getAuthor()
{
    return $this->author;
}
```

to:

```php
/**
 * Set author
 *
 * @param Author $author
 *
 * @return BlogPost
 */
public function setAuthor(Author $author)
{
    $this->author = $author;

    return $this;
}

/**
 * Get author
 *
 * @return Author
 */
public function getAuthor()
{
    return $this->author;
}
```

All this does is make sure the entity knows that `Author` has a [`ManyToOne` relationship](https://www.ibm.com/support/knowledgecenter/en/SSWU4L/Data/imc_Data/What_is_a_many-to-one_relationship.html) with `BlogPost`.

At the bottom of the `BlogPost` class, MySQL needs to know to populate and update the `created_at` and `updated_at` columns when a persist or update is made to the database. So paste the following at the bottom of the class (right before the last `}` in the file):

```php
/**
 * @ORM\PrePersist
 */
public function prePersist()
{
    if (!$this->getCreatedAt()) {
        $this->setCreatedAt(new \DateTime());
    }

    if (!$this->getUpdatedAt()) {
        $this->setUpdatedAt(new \DateTime());
    }
}

/**
 * @ORM\PreUpdate
 */
public function preUpdate()
{
    $this->setUpdatedAt(new \DateTime());
}
```

Now this is useless until the class has lifecycle call backs so in the annotation above the class, you'll see:

```php
/**
 * BlogPost
 *
 * @ORM\Table(name="blog_post")
 * @ORM\Entity(repositoryClass="AppBundle\Repository\BlogPostRepository")
```

under `* @ORM\Entity()` add a new line and place this in there:

```php
 * @ORM\HasLifecycleCallbacks
```

Our entities are all set up! But.. we don't have the database tables yet.

### Install Doctrine-Migrations

Run: `composer require doctrine/doctrine-migrations-bundle "^1.0"`

In `app/AppKernel.php` under the method `registerBundles` in the `$bundles` array, add a new row and place the following in there:

```php
new Doctrine\Bundle\MigrationsBundle\DoctrineMigrationsBundle(),
```

At the bottom of `app/config/config.yml` add:

```yml
doctrine_migrations:
    dir_name: "%kernel.root_dir%/DoctrineMigrations"
    namespace: Application\Migrations
    table_name: migration_versions
    name: Application Migrations
    organize_migrations: false # Version >=1.2 Possible values are: "BY_YEAR", "BY_YEAR_AND_MONTH", false
```

Run a migrations diff `php bin/console doctrine:migrations:diff`

This will generate a file in `app/DoctrineMigrations/` starting with `Version` then the date and time it was created. So for this instance, there'll only be the one file. This file contains the migration queries for creating the new MySQL tables.

Run the migration, which will run all the SQL queries found in the generated file with: `php bin/console doctrine:migrations:migrate`

### Install Doctrine-Fixtures

We want to just populate some data into your newly created tables as examples during the creation of the blog. So install [doctrine-fixtures](https://symfony.com/doc/master/bundles/DoctrineFixturesBundle/index.html).

```bash
composer require --dev doctrine/doctrine-fixtures-bundle
```

In your `app/AppKernel.php` you want to add the bundle but specifically only for development and testing environments. So add this line `$bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle();` as shown below.

```php
public function registerBundles()
{
    // ...
    if (in_array($this->getEnvironment(), array('dev', 'test'), true)) {
        // ...
        $bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle();
    }

    return $bundles;
}
```

### Create Author and BlogPost Fixtures

Create a new file (and the directories the file is stored in) under `src/AppBundle/DataFixtures/ORM/Fixtures.php` and insert the following into the file:

```php
<?php

namespace AppBundle\DataFixtures\ORM;

use AppBundle\Entity\Author;
use AppBundle\Entity\BlogPost;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\Persistence\ObjectManager;

class Fixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        $author = new Author();
        $author
            ->setName('Joe Bloggs')
            ->setTitle('Developer')
            ->setUsername('auth0-username')
            ->setCompany('The Writing Company')
            ->setShortBio('Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry\'s standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages.')
            ->setPhone('070000000')
            ->setFacebook('joebloggs')
            ->setTwitter('joe.bloggs')
            ->setGithub('joe-bloggs');

        $manager->persist($author);

        $blogPost = new BlogPost();
        $blogPost
            ->setTitle('Your first blog post example')
            ->setSlug('first-post')
            ->setDescription('Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry\'s standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.')
            ->setBody('Contrary to popular belief, Lorem Ipsum is not simply random text. It has roots in a piece of classical Latin literature from 45 BC, making it over 2000 years old. Richard McClintock, a Latin professor at Hampden-Sydney College in Virginia, looked up one of the more obscure Latin words, consectetur, from a Lorem Ipsum passage, and going through the cites of the word in classical literature, discovered the undoubtable source. Lorem Ipsum comes from sections 1.10.32 and 1.10.33 of "de Finibus Bonorum et Malorum" (The Extremes of Good and Evil) by Cicero, written in 45 BC. This book is a treatise on the theory of ethics, very popular during the Renaissance. The first line of Lorem Ipsum, "Lorem ipsum dolor sit amet..", comes from a line in section 1.10.32. The standard chunk of Lorem Ipsum used since the 1500s is reproduced below for those interested. Sections 1.10.32 and 1.10.33 from "de Finibus Bonorum et Malorum" by Cicero are also reproduced in their exact original form, accompanied by English versions from the 1914 translation by H. Rackham.')
            ->setAuthor($author);
        $manager->persist($blogPost);
        $manager->flush();
    }
}
```

This is basically just a file that, when loaded, will create an author and a blog post, and will link the author to the blog post.

### Running Fixtures

Let's run the fixtures! `php bin/console doctrine:fixtures:load`

Now we have our entities, database, database tables, and some dummy data in the tables.

## Managing Identity with Symfony and Auth0

### Installing HWIOAuth Bundle

To make Symfony integrate with Auth0, we are going to use  [HWIOAuth Bundle](https://github.com/hwi/HWIOAuthBundle), an OAuth client that supports OAuth2. In order to install HWIOAuth Bundle, which uses a virtual package `php-http/client-implementation`, we need to install several third party libraries. However, this can be easily done with this command: `composer require php-http/httplug-bundle php-http/curl-client guzzlehttp/psr7`

Once Composer has finished, we need to enable it in our `app/AppKernel.php`. In the `$bundles` array, add a new row placing the following in there:

```php
new Http\HttplugBundle\HttplugBundle(),
```

Now, open your `composer.json` file and add `HWIOAuthBundle` (as well as adding the `minimum-stability` and `prefer-stable` options):

```
"minimum-stability": "dev",
"prefer-stable": true,
"require": {
    ...
    "hwi/oauth-bundle": ">=0.6",
},
```

__NOTE__: You may have an issue if your version of PHP is too high. If you encounter an error similar to `overridden by "config.platform.php" version (5.5.9) does not satisfy that requirement`, remove the PHP requirement in config in `composer.json` so find and remove:

```
"platform": {
    "php": "5.5.9"
},
```

Now run `composer update`.

In `app/AppKernel.php` under the method `registerBundles` in the `$bundles` array, add a new row and place the following in there:

`new HWI\Bundle\OAuthBundle\HWIOAuthBundle(),`

We also need to add the routes to your routing file: `app/config/routing.yml`:

```yml
# ...

hwi_oauth_redirect:
    resource: "@HWIOAuthBundle/Resources/config/routing/redirect.xml"
    prefix:   /connect

hwi_oauth_login:
    resource: "@HWIOAuthBundle/Resources/config/routing/login.xml"
    prefix:   /login

auth0_login:
    path:    /auth0/callback

auth0_logout:
    path: /auth0/logout
```

### Including DotEnv & Configs

In order to get the `HWIOAuthBundle` to connect to Auth0, we need to create an Auth0 resource owner. So create a new file `src/AppBundle/Auth0ResourceOwner.php`:

```php
<?php

namespace AppBundle;

use Symfony\Component\OptionsResolver\Options;
use Symfony\Component\OptionsResolver\OptionsResolver;

use HWI\Bundle\OAuthBundle\OAuth\ResourceOwner\GenericOAuth2ResourceOwner;

class Auth0ResourceOwner extends GenericOAuth2ResourceOwner
{
    /**
     * {@inheritdoc}
     */
    protected $paths = array(
        'identifier' => 'user_id',
        'nickname' => 'nickname',
        'realname' => 'name',
        'email' => 'email',
        'profilepicture' => 'picture',
    );

    /**
     * {@inheritdoc}
     */
    public function getAuthorizationUrl($redirectUri, array $extraParameters = array())
    {
        return parent::getAuthorizationUrl($redirectUri, array_merge(array(
            'audience' => $this->options['audience'],
        ), $extraParameters));
    }

    /**
     * {@inheritdoc}
     */
    protected function configureOptions(OptionsResolver $resolver)
    {
        parent::configureOptions($resolver);

        $resolver->setDefaults(array(
            'authorization_url' => '{base_url}/authorize',
            'access_token_url' => '{base_url}/oauth/token',
            'infos_url' => '{base_url}/userinfo',
            'audience' => '{base_url}/userinfo',
        ));

        $resolver->setRequired(array(
            'base_url',
        ));

        $normalizer = function (Options $options, $value) {
            return str_replace('{base_url}', $options['base_url'], $value);
        };

        $resolver->setNormalizer('authorization_url', $normalizer);
        $resolver->setNormalizer('access_token_url', $normalizer);
        $resolver->setNormalizer('infos_url', $normalizer);
        $resolver->setNormalizer('audience', $normalizer);
    }
}
```

We're going to need API keys for Auth0, so <a href="https://auth0.com/signup" data-amp-replace="CLIENT_ID" data-amp-addparams="anonId=CLIENT_ID(cid-scope-cookie-fallback-name)">create your free account</a>. Once signed up:

- In the dashboard, click `Clients` on the left:
  * Create Client
  * Add a name
  * Choose _Regular Web Applications_ type
- Configure callback URL:
  * In the new Auth0 `Client`, go to the settings tab.
  * Find the text box labeled `Allowed Callback URLs`.
  * Paste the following in: `http://localhost:8000/auth0/callback`.
- Configure Auth0 Client to require usernames:
  * In the navigation bar find and click `Connections`
  * Then click `Database`
  * Click on `Username-Password-Authentication`
  * Toggle `Requires Username` to on.
- Go to the [Clients](https://manage.auth0.com/#/clients) section again and pick your `Client`.

Then in your `.env` file paste the following, but replace the brackets and their contents with the details found in your client:

```
AUTH0_CLIENT_ID=(Client ID on Auth0)
AUTH0_CLIENT_SECRET=(Client Secret on Auth0)
AUTH0_DOMAIN=(Domain on Auth0)
```

To use all of this, we need to add details to the `app/config/config.yml` file. At the bottom paste the following:

```
hwi_oauth:
    firewall_names: [secured_area]
    resource_owners:
        auth0:
            type:                oauth2
            class:               'AppBundle\Auth0ResourceOwner'
            client_id:           "%env(AUTH0_CLIENT_ID)%"
            client_secret:       "%env(AUTH0_CLIENT_SECRET)%"
            base_url:            "https://%env(AUTH0_DOMAIN)%"
            scope: "openid profile"
```

Finally, replace the contents of your `app/config/security.yml` file with:

```
security:
    # https://symfony.com/doc/current/security.html#b-configuring-how-users-are-loaded
    providers:
        in_memory:
            memory: ~
        hwi:
            id: hwi_oauth.user.provider

    firewalls:
        # disables authentication for assets and the profiler, adapt it according to your needs
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        secured_area:
            anonymous: ~
            oauth:
                resource_owners:
                    auth0: "/auth0/callback"
                login_path:        /login
                use_forward:       false
                failure_path:      /login

                oauth_user_provider:
                    service: hwi_oauth.user.provider
            logout:
                path:   /auth0/logout
                target: /
        main:
            anonymous: ~
            # activate different ways to authenticate

            # https://symfony.com/doc/current/security.html#a-configuring-how-your-users-will-authenticate
            #http_basic: ~

            # https://symfony.com/doc/current/security/form_login_setup.html
            #form_login: ~

    access_control:
        - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin, roles: ROLE_OAUTH_USER }
```

The config above is setting up URLs/sections in your blog that require the user to be authenticated.

### Create the Author Page

Let's create our admin blog controller by running the following command: `php bin/console generate:controller`.

![Creating a controller on Symfony](https://cdn.auth0.com/blog/symfony-blog/create-admin-controller.png)

If you follow the instructions as shown in the image above, you'll find that you have a new Controller class in `src/AppBundle/Controllers/` called `AdminController`. You'll also have a new template in `src/AppBundle/Resources/views/Admin/`.

__NOTE__: If you cannot see the image, the full controller can be found [here](https://github.com/GregHolmes/symfony-blog/blob/master/part-1/src/AppBundle/Controller/AdminController.php)

The first thing we will want to do is create a new form. This is a class that allows us to validate the users' input, such as creating an author. So, in the `src/AppBundle` path, create a new directory called `Form`.
Within that directory create a new file named `AuthorFormType.php`. Add the code below to your new file:

```php
<?php

namespace AppBundle\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\NotBlank;

class AuthorFormType extends AbstractType
{
    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // Below is all of the inputs that the author will see when creating their new author access. Each one contains its own attributes where applicable, such as the field `name` being required.
        // Please note the naming of these form elements is the same naming as the entity. We will be passing in the Author entity so the form knows what the data is for.
        $builder
            ->add(
                'name',
                TextType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'title',
                TextType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'company',
                TextType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'shortBio',
                TextareaType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'phone',
                TextType::class,
                [
                    'attr' => ['class' => 'form-control'],
                    'required' => false
                ]
            )
            ->add(
                'facebook',
                TextType::class,
                [
                    'attr' => ['class' => 'form-control'],
                    'required' => false
                ]
            )
            ->add(
                'twitter',
                TextType::class,
                [
                    'attr' => ['class' => 'form-control'],
                    'required' => false
                ]
            )
            ->add(
                'github',
                TextType::class,
                [
                    'attr' => ['class' => 'form-control'],
                    'required' => false
                ]
            )
            ->add(
                'submit',
                SubmitType::class,
                [
                    'attr' => ['class' => 'form-control btn-primary pull-right'],
                    'label' => 'Become an author!'
                ]
            );
    }

    /**
     * {@inheritdoc}
     */
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => 'AppBundle\Entity\Author'
        ]);
    }

    /**
     * {@inheritdoc}
     */
    public function getName()
    {
        return 'author_form';
    }
}
```

Back in your new AdminController (`src/AppBundle/Controller/AdminController.php`), we need to make use the entity manager and the repositories for the entities in order to retrieve database data. At the top of this class, inject these services:

```php
/** @var EntityManagerInterface */
private $entityManager;

/** @var \Doctrine\Common\Persistence\ObjectRepository */
private $authorRepository;

/** @var \Doctrine\Common\Persistence\ObjectRepository */
private $blogPostRepository;

/**
 * @param EntityManagerInterface $entityManager
 */
public function __construct(EntityManagerInterface $entityManager)
{
    $this->entityManager = $entityManager;
    $this->blogPostRepository = $entityManager->getRepository('AppBundle:BlogPost');
    $this->authorRepository = $entityManager->getRepository('AppBundle:Author');
}
```

As you can see, there is a class declared here so we need to add it to the namespaces at the top of the file. Where it says:

```php
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
```

Add: `use Doctrine\ORM\EntityManagerInterface;` above the two.

Great, in our entire controller we can call `blogPostRepository`, `authorRepository` or `entityManager` when needed. The first two are used for retrieving data from the database, whereas the third will be used for inserting, updating, or deleting data.

In the `AdminController` class, you will find your new `createAuthorAction` method. Change the annotation so that the method has a service name. So above the controller, where you see `@Route`, add the service name. So it will look like:

```php
* @Route("/admin/author/create", name="author_create")
```

At the top of the controller, we need to include this class so where it shows:

```php
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Doctrine\ORM\EntityManagerInterface;
```

Paste the following below those:

```php
use Symfony\Component\HttpFoundation\Request;
```

The `Request` class allows you to gather data, for example, if there are any parameters in a POST or GET request.

Then change `createAuthorAction()` to `createAuthorAction(Request $request)`.

We want to make sure the user isn't trying to create themselves as duplicate authors for the same user. So at the top of the `createAuthorAction` let's run a query to check whether one already exists.
Paste the code below into the action. This retrieves the authenticated user with Auth0, checks their username with the `authors` table to see if one exists, if it does then redirect the user to the index page.

```php
// Check whether user already has an author.
if ($this->authorRepository->findOneByUsername($this->getUser()->getUserName())) {
   // Redirect to dashboard.
   $this->addFlash('error', 'Unable to create author, author already exists!');

   return $this->redirectToRoute('homepage');
}
```

Below that you can now create an empty `Author` entity. Create a new `AuthorFormType` object and pass the `Author` entity into the form.

```php
$author = new Author();
$author->setUsername($this->getUser()->getUserName());

$form = $this->createForm(AuthorFormType::class, $author);
$form->handleRequest($request);

if ($form->isValid()) {
    $this->entityManager->persist($author);
    $this->entityManager->flush($author);

    $request->getSession()->set('user_is_author', true);
    $this->addFlash('success', 'Congratulations! You are now an author.');

    return $this->redirectToRoute('homepage');
}
```

We are using two new classes here. The first one is the entity `Author` and the second one `AuthorFormType` class. Include these namespaces at the top of our file:

```php
namespace AppBundle\Controller;
//...
//...
use AppBundle\Entity\Author;
use AppBundle\Form\AuthorFormType;
```

The above code also checks whether the form has been submitted and whether it passes all validations, if it does it will then redirect the user to the index page.

__NOTE__: You may have noticed that it sets a session to `true` for `user_is_author` this will make sense when we reach the part that discusses and implements event listeners (next).

Finally, we want to pass the form into the template that the user will see. So, at the bottom of the method, let's change:

```php
return $this->render('AppBundle:Admin:create_author.html.twig', array(
    // ...
));
```

to:

```php
return $this->render('AppBundle:Admin:create_author.html.twig', array(
    'form' => $form->createView()
));
```

In our template for this action, we just want the user to have all of the form fields displayed as their correct form elements. So, in the `create_author.html.twig` file, copy and paste the following:

{% highlight html %}
{% raw %}
{% extends '::base.html.twig' %}

{% block title %}{% endblock %}

{% block body %}
    <div class="container">
        <div class="blog-header">
            <h2 class="blog-title">Creating your author</h2>
        </div>

        <div class="row">
            <div class="col-md-12 col-lg-12 col-xl-12">
                {% for label, messages in app.flashes %}
                    {% for message in messages %}
                        <div class="alert alert-{{ label }}" role="alert">
                            <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                            {{ message }}
                        </div>
                    {% endfor %}
                {% endfor %}
            </div>
            <div class="col-sm-12 blog-main">
                {{ form_start(form) }}
                <div class="col-md-12">
                    <div class="form-group col-md-4">
                        {{ form_row(form.name) }}
                    </div>
                    <div class="form-group col-md-4">
                        {{ form_row(form.title) }}
                    </div>
                    <div class="form-group col-md-4">
                        {{ form_row(form.company) }}
                    </div>
                    <div class="form-group col-md-12">
                        {{ form_row(form.shortBio) }}
                    </div>
                    <div class="form-group col-md-4">
                        {{ form_row(form.phone) }}
                    </div>
                    <div class="form-group col-md-4">
                        {{ form_row(form.facebook) }}
                    </div>
                    <div class="form-group col-md-4">
                        {{ form_row(form.twitter) }}
                    </div>
                    <div class="form-group col-md-4">
                        {{ form_row(form.github) }}
                    </div>
                    <div class="form-group col-md-4">
                        {{ form_row(form.shortBio) }}
                    </div>
                </div>
                <div class="form-group col-md-4 pull-right">
                    {{ form_row(form.submit) }}
                </div>
                {{ form_end(form) }}
            </div>
        </div>
    </div>
{% endblock %}
{% endraw %}
{% endhighlight %}

You can restart now your Symfony application to see if everything is there no errors. Note that there is nothing to see yet, we just want to guarantee that the configuration is correct.

```bash
php bin/console server:stop
php bin/console server:start
```

### Including Bootstrap and Styles

As it is, this template will look very ugly in your browser. So let's create a new CSS file. `web/css/style.css`

Copy the contents of the file [found here](https://github.com/GregHolmes/symfony-blog/blob/master/part-1/web/css/style.css) into your new `style.css` file.

Now, it's time to include this file into the base template alongside with Bootstrap's CSS and Javascript files.

Open `app/Resources/views/base.html.twig` and replace its contents with:

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>{% block title %}Welcome!{% endblock %}</title>
    {% block stylesheets %}{% endblock %}
    <link rel="icon" type="image/x-icon" href="{{ asset('favicon.ico') }}" />
    <!-- Latest compiled and minified CSS -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">

    <!-- jQuery library -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>

    <!-- Latest compiled JavaScript -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

    <!-- Custom CSS File -->
    <script src="css/style.css"></script>
</head>
<body>
{% block body %}{% endblock %}
{% block javascripts %}{% endblock %}
</body>
</html>
{% endraw %}
{% endhighlight %}

This just includes your custom CSS file as well as Bootstrap's CSS and Javascript files into your page.

### Asking for "About Me" Information

Although a user is authenticated through Auth0, we don't really want them to access the rest of the author section until they've filled some basic "About me" details. So, what we want is to ensure that any `/author/` route they try to access, if they don't have an "Author" entry, redirects them to the form to write about themselves.

An event listener is, as the name states, a listener to specific programmed events which then run predefined code once the conditions are met.

Let's create the `CheckIsAuthorListener.php` file in the `src/AppBundle/EventListener/Author` directory. In this file, let's place the following code:

```php
<?php

namespace AppBundle\EventListener\Author;

use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Routing\Router;
use Symfony\Component\HttpFoundation\RedirectResponse;
use Symfony\Component\HttpFoundation\Session\Session;
use Symfony\Component\HttpKernel\Event\FilterControllerEvent;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorage;

class CheckIsAuthorListener
{
    /** @var Router */
    protected $router;

    /** @var Session */
    protected $session;

    /** @var TokenStorage */
    private $tokenStorage;

    /** @var EntityManagerInterface */
    private $entityManager;

    /**
     * @param Router $router
     * @param Session $session
     * @param TokenStorage $tokenStorage
     * @param EntityManagerInterface $entityManager
     */
    public function __construct(
        Router $router,
        Session $session,
        TokenStorage $tokenStorage,
        EntityManagerInterface $entityManager
    ) {
        $this->router = $router;
        $this->session = $session;
        $this->tokenStorage = $tokenStorage;
        $this->entityManager = $entityManager;
    }

    /**
     * On kernel.controller
     *
     * @param FilterControllerEvent $event
     *
     * @return void
     */
    public function onKernelController(FilterControllerEvent $event)
    {
        // Don't add to the flasher if the current path does not begin with /admin
        if (!preg_match('/^\/admin/i', $event->getRequest()->getPathInfo())) {
            return;
        }

        if (null === $user = $this->tokenStorage->getToken()->getUser()) {
            return;
        }

        // Use the session to exit this listener early, if the relevant checks have already been made
        if (true === $this->session->get('user_is_author')) {
            return;
        }

        $route = $this->router->generate('author_create');

        // Check we are not already attempting to create an author!
        if (0 === strpos($event->getRequest()->getPathInfo(), $route)) {
            return;
        }

        // Check if authenticated user has an author associated with them.
        if ($author = $this->entityManager
                ->getRepository('AppBundle:Author')
                ->findOneByUsername($user->getUsername())
        ) {
            $this->session->set('user_is_author', true);
        }

        if (!$author && $this->session->get('pending_user_is_author')) {
            $this->session->getFlashBag()->add(
                'warning',
                'Your author access is being set up, this may take up to 30 seconds. Please try again shortly.'
            );

            $route = $this->router->generate('homepage');
        } else {
            $this->session->getFlashBag()->add(
                'warning',
                'You cannot access the author section until you become an author. Please complete the form below to proceed.'
            );
        }

        $event->setController(function () use ($route) {
            return new RedirectResponse($route);
        });
    }
}
```

We now need to add this class as a service so that Symfony runs it on each controller request. In `app/config/services.yml` add the following as the last item of `services`:

```yml
AppBundle\EventListener\Author\CheckIsAuthorListener:
    tags:
        - { name: kernel.event_listener, event: kernel.controller }
```

It's great setting up HWIOAuth Bundle and configuring Auth0 to allow users to log in, but we don't yet have anywhere in the Symfony installation to actually log in. So, for the time being, we're going to produce a link on the Homepage page.

Open the file `app/Resources/views/default/index.html.twig` and replace its entire content with:

{% highlight html %}
{% raw %}
{% extends 'base.html.twig' %}

{% block body %}
    <div id="wrapper">
        <div id="container">
            {% if app.user %}
                <li><a href="{{ path("author_create") }}">Admin</a></li>
                <li><a href="{{ logout_url("secured_area") }}">Logout</a></li>
            {% else %}
                <li class="active"><a href="/connect/auth0">Login</a></li>
            {% endif %}
        </div>
    </div>
{% endblock %}
{% endraw %}
{% endhighlight %}

As you can see, now we provide a link so that unauthenticated users can log in. If they are logged in, two links are shown: one to log out and one to access the admin section.

{% include tweet_quote.html quote_text="Using Symfony and OAuth together is quite easy." %}

## Next Steps

In the next article, we will be covering installing Bootstrap, a framework, to make the blog nicer visually. We will also enhance our blog engine to allow visitors to:

* see a list of blog posts;
* read a specific blog post;
* and find out more about the author.

Authenticated authors will be able to:

* create a new blog post;
* see all of their own blog posts;
* delete their own blog posts from the system.
