# Créer un blog minimaliste avec Laravel

## Un nouveau projet Laravel
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

## Models, migrations et controllers

On créer les models avec les fichiers de migration et controllers ([Doc](https://laravel.com/docs/9.x/eloquent#generating-model-classes)) <br>

On utilise [artisan](https://laravel.com/docs/9.x/artisan) à travers [sail](https://laravel.com/docs/9.x/sail)

```bash
sail php artisan make:model Post -mc     # migration et controller
sail php artisan make:model Comment -mc
sail php artisan make:model Reply -mc
```
## Contenu des migrations

database/migrations/create_posts_table.php
```php
<?php

public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->bigIncrements('id');
        $table->unsignedInteger('user_id');
        $table->string('title');
        $table->text('body');
        $table->timestamps();
    });
}
```

database/migrations/create_comments_table.php
```php
<?php

public function up()
{
    Schema::create('comments', function (Blueprint $table) {
        $table->bigIncrements('id');
        $table->unsignedInteger('user_id');
        $table->unsignedInteger('post_id');
        $table->text('body');
        $table->timestamps();
    });
}
```

database/migrations/create_replies_table.php
```php
<?php

public function up()
{
    Schema::create('replies', function (Blueprint $table) {
        $table->bigIncrements('id');
        $table->unsignedInteger('user_id');
        $table->unsignedInteger('comment_id');
        $table->text('body');
        $table->timestamps();
    });
}
```

## Contenu des models et création des relations

On ajoute une relation _one-to_many_ au model User
```php

<?php

/** User model **/
/** One to Many relation with Post **/
public function posts() 
{
    return $this->hasMany(Post::class, 'user_id');
}
```

On ajoute des attributs et les relations au model Post
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    // table name to be used
    protected $table = 'posts';

    // columns to be allowed in mass-assingment 
    protected $fillable = ['user_id', 'title', 'body'];

    /* Relations */

    // One to many inverse relationship with User model
    public function owner() {
    	return $this->belongsTo(User::class, 'user_id');
    }

    // One to Many relationship with Comment model
    public function comments()
    {
    	return $this->hasMany(Comment::class, 'post_id');
    }

    /**
     * get show post route
     *
     * @return string
     */
    public function path()
    {
        return "/posts/{$this->id}";
    }
}
```

Ainsi qu'au model Comment
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    # table name to be used by model
    protected $table = 'comments';

    # columns to be allowed in mass-assingment
    protected $fillable = ['user_id', 'post_id', 'body'];

    /** Relations */

    # One-to-Many inverse relation with User model.
    public function owner()
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    # One-to-Many inverse relation with Post model.
    public function post()
    {
    	return $this->belongsTo(Post::class, 'post_id');
    }

    # One-to-Many relation with Reply model.
    public function replies()
    {
    	return $this->hasMany(Reply::class, 'comment_id');
    }
}
```

Et qu'au model Reply
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Reply extends Model
{
    # table name to be used by model.
    protected $table = 'replies';

    # columns names to be used in mass-assignment
    protected $fillable = ['user_id', 'comment_id', 'body'];

    /* Relations */

    # One-to-Many inverse relationship with User model.
    public function owner()
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    # One-to-Many inverse relationship with Comment model.
    public function comment()
    {
    	return $this->belongsTo(Comment::class, 'comment_id');
    }
}
```
