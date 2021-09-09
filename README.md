Installation & utilisation
--------------------------

Création dossier app

> mkdir app

Démarrage des conteneurs
> docker-compose up -d --build

> http://localhost:8080/

Utilisation des commandes php dans le conteneur
> docker-compose exec php /bin/bash

Création d'un projet Symfony
> symfony new .

Installation de dépendance

> composer req doctrine twig

Installation de dépendance de dev

> composer req --dev maker ormfixtures fakerphp/faker

Création de l'entité Article

> symfony console make:entity Article 

Création de l'entité Comment

> symfony console make:entity Comment

Création de la relation Article OneToMany Comment

> symfony console make:entity Article 

Modification url connection bdd dans le .env

> DATABASE_URL="mysql://symfony:symfony@database:3306/symfony_course?serverVersion=8.0"

Création des tables

> symfony console make:migration

> symfony console doctrine:migrations:migrate

Pour vérifier la création, se connecter à la bdd

> docker-compose exec database /bin/bash
>> mysql -u root -p symfony_course
> 
>> show tables;

Création de fixtures

> symfony console make:fixture ArticleFixture

>       $faker = Factory::create();
>       for ($i = 0; $i < 10; $i++) {
>           $article = new Article();
>           $article->setTitle($faker->sentence());
>           $article->setContent($faker->text(2000));
>           $manager->persist($article);
>       }
>       $manager->flush();

> symfony console doctrine:fixtures:load

> SELECT * FROM article;

Création Article CRUD

> symfony console make:crud

Ajouter Profiler

> composer require --dev symfony/debug-bundle symfony/profiler-pack

Création d'une api public pour les articles 

> symfony console make:serializer:normalizer

> symfony console make:controller

Api Doc

> composer require nelmio/api-doc-bundle -W

Exercise

- Ajouter de la validation sur les articles
  - tous les champs sont obligatoires
  - username: min 2 et max 20 caractères
  - content: min 2 et max 1500 caractères
  - blacklister des mots dans le contenu

- Afficher les deux derniers commentaires dans le endpoint '/api/articles'

- Ajouter bootstrap sur le template de base
