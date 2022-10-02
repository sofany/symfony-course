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

    public function normalize($object, $format = null, array $context = []): array
    {
        return [
            'id' => $object->getId(),
            'title' => $object->getTitle(),
            'content' => $object->getContent(),
        ];
    }

> symfony console make:controller

    public function index(ArticleRepository $articleRepository, NormalizerInterface $normalizer): JsonResponse
    {
        return new JsonResponse(
            ['results' => $normalizer->normalize($articleRepository->findAll())]
        );
    }

Api Doc

> composer require nelmio/api-doc-bundle -W
> composer require symfony/asset

Exercise

- Ajouter une photo dans l'article

- Ajouter de la validation sur les articles
  - title et content sont obligatoires
  - content: min 2 et max 1500 caractères
  - blacklister des mots dans le contenu

- Afficher les deux derniers commentaires dans le endpoint '/api/article'


Ajouter l'entité User

> composer require symfony/security-bundle

> symfony console make:user

> symfony console make:migration

> symfony console doctrine:migrations:migrate

> symfony console make:fixture UserFixture
 
    $user = new User();
    $user->setUsername('admin');
    $user->setPassword($this->passwordHasher->hashPassword($user, 'the_new_password'));
    $user->setRoles(['ROLE_ADMIN']);
    $manager->persist($user);

> symfony console doctrine:fixtures:load

Ajouter un firewall admin dans security.yaml

    providers:
        admin_provider:
            entity:
                class: App\Entity\User
                property: username
    firewalls:
        admin:
            pattern: /admin
            http_basic: ~
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }

Changer la route /article => /admin/article

Ajouter la propriété author lié à l'entité User

> symfony console make:entity Article 

> symfony console make:migration

> symfony console doctrine:migrations:migrate

    //ArticleFixtures.php
    $user1 = new User();
    $user1->setUsername('user_1');
    $user1->setPassword($this->passwordHasher->hashPassword($user1, 'the_new_password'));
    $user1->setRoles(['ROLE_USER']);
    $manager->persist($user1);

    $user2 = new User();
    $user2->setUsername('user_2');
    $user2->setPassword($this->passwordHasher->hashPassword($user2, 'the_new_password'));
    $user2->setRoles(['ROLE_USER']);
    $manager->persist($user2);

    $article->setAuthor( $i % 2 === 0 ? $user2 : $user1);

> symfony console doctrine:fixtures:load

Ajouter la propriété apiToken dans l'entité User

> symfony console make:entity User

> symfony console make:migration

> symfony console doctrine:migrations:migrate

    //ArticleFixtures.php
    $user1->setApiToken('token_1');

    $user2->setApiToken('token_2');

> symfony console doctrine:fixtures:load

Créer un authenticator pour l'api

> symfony console make:auth

    public function supports(Request $request): ?bool
    {
        return $request->headers->has('X-AUTH-TOKEN');
    }

    public function authenticate(Request $request): PassportInterface
    {
        $apiToken = $request->headers->get('X-AUTH-TOKEN');
        if (null === $apiToken) {
            throw new CustomUserMessageAuthenticationException('No API token provided');
        }

        return new SelfValidatingPassport(new UserBadge($apiToken));
    }

Ajouter un firewall api dans security.yaml

    providers:
        user_provider:
            entity:
                class: App\Entity\User
                property: apiToken
    firewalls:
        api:
            pattern: /api
            lazy: true
            custom_authenticator: App\Security\TokenAuthenticator
            provider: user_provider
    access_control:
        - { path: ^/api, roles: ROLE_USER }

Tester que l'utilisateur est bien connecté

> curl -H "X-AUTH-TOKEN: token_2"  http://localhost:8080/api/articles

Vérifier dans le profiler 

> http://localhost:8080/_profiler/empty/search/results?limit=5

Faire un endpoint /api/article/{id} visible que par l'auteur

Faire une page de login pour l'admin sur les routes '/admin'

> symfony console make:auth

Ajouter le Reset Password

> symfony console make:reset-password

> http://localhost:8081

Création d'un article avec l'auteur courant

Création d'un test automatique avec phpunit

> composer require --dev phpunit/phpunit 

> https://phpunit.readthedocs.io/fr/latest/

Tester la méthode supportsNormalization du normalizer ArticleNormalizer

> enlever la dépendance ObjectNormalizer d'ArticleNormalizer

    class ArticleNormalizerTest extends TestCase
    {
        private ArticleNormalizer $normalizer;
    
        public function setUp(): void
        {
            $this->normalizer = new ArticleNormalizer();
        }
    }

Consommer une api externe (https://symfony-course.free.beeceptor.com/cities)

- Ajouter une nouvelle route /cities/{cityName}
- Récupérer les informations de la ville

> composer require symfony/http-client

    Request : http://localhost:8080/cities/Yport
    Response : {"country":"FR","name":"Yport","lat":"49.73716","lng":"0.31537"}


Exercice d'évaluation :
- Ajouter une route "/api/article/random" qui renvoie un article au hasard
- Ajouter en argument de la commande "app:visualize-city" le nom de la ville pour visualiser uniquement la ville
- Améliorer le validateur BannedWord, les mots à bannir sont initialisés lors de la création de la "Constraint"
  - ex: new BannedWord(words: ['test', 'tata'])
- Tester la méthode "normalize" d'ArticleNormalizer

Bonus:
- Créer un nouvel utilisateur avec le rôle "ROLE_MODERATOR" qui peut accéder aux routes "/admin" mais ne peut pas supprimer d'article
- Améliorer le validateur BannedWord, les mots à bannir sont récupérés via une api (https://symfony-course.free.beeceptor.com/banned-words)

Envoyer votre projet (sans le dossier vendor) à l'adresse : sofany.ong@edu.gobelins.fr