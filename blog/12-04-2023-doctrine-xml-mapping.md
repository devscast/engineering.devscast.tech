---
title: How to configure Doctrine XML Mapping
description: How to configure Doctrine XML mapping when using symfony
slug: symfony-how-to-confirgure-doctrine-xml-mapping
authors:
  - name: Bernard Ng
    title: Devscast Community Lead
    url: https://www.linkedin.com/in/bernard-ngandu/
    image_url: https://github.com/bernard-ng.png
tags: [symfony, ddd]
hide_table_of_contents: false
---

## Symfony : How to configure Doctrine XML Mapping

This article was originally published in French [here](https://devscast.tech/posts/ddd-avec-symfony-comment-configurer-mapping-xml-doctrine-26).

[Doctrine](https://www.doctrine-project.org/) is an ORM for [PHP](https://php.net) that provides transparent persistence for PHP objects. It uses the [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html) pattern, which aims to completely separate your domain/business logic from persistence in a relational database management system.

In a classic Symfony application the configuration of Doctrine Entities is done through annotations (or attributes with PHP +8.0).

In this article we will see

- Why it is preferable to use XML mapping when doing DDD
- How to configure Doctrine XML mapping 

## Why it is preferable to use XML mapping when doing DDD

Let's start with a simple example, in a classical Symfony application here is how an entity would look like 

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use App\Repository\UserRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Table(name: '`user`')]
#[ORM\Entity(repositoryClass: UserRepository::class)]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 180, unique: true)]
    private ?string $email = null;

    #[ORM\Column(type: 'json')]
    private array $roles = ['ROLE_USER'];

    #[ORM\Column(type: 'string')]
    private ?string $password = null;

    // ...
}
```

Of course let's not forget the accessors and mutators for each property since they are private, for the sake of concision I won't put it in my examples.

One thing we can notice very quickly by looking at our user class which obviously represents a user in the context of our domain has meta data from Doctrine, which can be annoying because the ideal would be to have the domain code unaware in some way of the infrastructure, remember that this is not an obligation in our case as **Matthias Noback** explains in [this article](https://matthiasnoback.nl/2020/05/ddd-and-your-database/), because it is only meta data and not a coupling so the entity remains testable in isolation and does not depend on Doctrine.

An important precision for the purists, I am not comparing a DDD Entity to a Doctrine Entity, the two concepts are different, my goal here is to show how we can get rid of annotations (or attributes).

This said, using XML has the advantage of removing from your domain code all the Doctrine meta data and move them to a dedicated configuration which will be obviously in the infrastructure, if you use tools like [phpat](https://github.com/carlosas/phpat) to guarantee the respect of some architectural requirements this approach would be preferable.

The designers of Doctrine have thought about this very particular use case and have developed other ways of mapping outside the annotations (or attributes), we can use [PHP](https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/php-mapping.html), [YAML](https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/xml-mapping.html) or XML, unfortunately the support of YAML and PHP will be removed from version 3.0 and it is recommended to use [XML](https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/xml-mapping.html) which we will do right now.

## How to configure Doctrine Mapping with XML
The example I gave before is valid in a classical Symfony application but here we are doing DDD and our architecture should look more like this 

```
src/
├── Application/
├── Domain/
│   └── Authentication/
│       ├── Entity/
│       │   ├── User.php
│       │   └── ...
│       └── Repository/
│           └── UserRepository.php (interface)
└── Infrastructure/
    └── Authentication/
        └── Doctrine/
            ├── Mapping/
            │   └── User.orm.xml
            ├── Repository/
            │   └── UserRepository (implementation)
            └── ...
```

Okay, there is a lot of information here, first notice that we have two times the `UserRepository`, the one that is in the domain is a simple interface and its implementation (concrete class) is in the infrastructure this to avoid depending on doctrine in our Domain Repository.

In order for your XML mapping to be taken into account by doctrine before configuring anything, the filename must match this pattern `[EntityName].orm.xml`, the file extension `.orm.xml` is important

With this way of doing our entity user should now look like this

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use App\Repository\UserRepository;

class User
{
    private ?int $id = null;
    private ?string $email = null;
    private array $roles = ['ROLE_USER'];
    private ?string $password = null;

    // ...
}
```

There is no more annotations (or attributes) coming from doctrine in our domain, the next step is to transcribe in XML the mapping previously written with annotations (or attributes) into our Infrastructure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-mapping xmlns="https://doctrine-project.org/schemas/orm/doctrine-mapping"
                  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="https://doctrine-project.org/schemas/orm/doctrine-mapping
                          https://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

    <entity name="Domain\Authentication\Entity\User" repository-class="Infrastructure\Authentication\Doctrine\Repository\UserRepository" table="user">
        <id name="id" type="integer" column="id">
            <generator strategy="IDENTITY"/>
        </id>

        <field name="name" type="string"/>
        <field name="email" type="string" length="180" unique="true"/>
        <field name="roles" type="json"/>
        <field name="password" type="string"/>
</doctrine-mapping>
```

Notice that we specify the FQCN of our entity and the associated repository (implementation) directly in the mapping, you can find more informations [here](https://www.doctrine-project.org/projects/doctrine-orm/en/2.11/reference/xml-mapping.html).

We are almost there, there is one last step, we should now "tell" doctrine how to find our mapping and where to find it and this is done through the doctrine configuration located in `/config/packages/doctrine.yaml`

```yaml
    orm:
        mappings:
            Domain\Authentication\Entity:
                type: xml
                dir: '%kernel.project_dir%/src/Infrastructure/Authentication/Doctrine/Mapping'
                prefix: 'Domain\Authentication\Entity'
                alias: Authentication
                is_bundle: false
```

To test that everything works well, just type the following command : ```php bin/console doctrine:mapping:info```.

## Conclusion
To finish, let's make a small recap, when doing DDD in the case of a Symfony application for entities we use the XML configuration which has the advantage to stay in the infrastructure layer which makes our entity unaware of the ORM used and allows us to guarantee the respect of architectural requirements.