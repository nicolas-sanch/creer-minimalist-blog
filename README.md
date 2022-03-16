# Créer un blog minimaliste avec Laravel

Avant de démarrer, nous avons besoin de [Docker Desktop](https://docs.docker.com/desktop/) et, les utilisateurs de Windows doivent s'assurer d'avoir _Windows Subsystem for Linux 2_ d'installé et activé. 

On commence par créer un nouveau projet Laravel

```curl -s "https://laravel.build/minimalist-blog-laravel" | bash```

Ensuite, on lance notre environnement de développement

```bash
cd minimalist-blog-laravel
 
./vendor/bin/sail up -d     # -d pour detach mode
```

Cela créer les services présents dans _docker-compose.yml_ <br>

Nous pouvons visualiser le résultat htt://localhost <br>

On créer les models [Doc](https://laravel.com/docs/9.x/eloquent#generating-model-classes) <br>

On utilise [artisan](https://laravel.com/docs/9.x/artisan) à travers [sail](https://laravel.com/docs/9.x/sail)

```bash
sail php artisan make:model Post -mc     # migration et controller
sail php artisan make:model Comment -mc
sail php artisan make:model Reply -mc
```
