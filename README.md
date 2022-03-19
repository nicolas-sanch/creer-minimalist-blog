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

## L'authentification

Nous allons utiliser (Laravel Breeze)[https://laravel.com/docs/9.x/starter-kits#laravel-breeze] pour implémenter l'authentification. <br>

❗ Suivez la documentation officielle sans oublier que nous intéragissons avec nos services via Sail

## Les routes

routes/web.php

```php
<?php

use Illuminate\Support\Facades\Route;

use App\Http\Controllers\CommentController;
use App\Http\Controllers\HomeController;
use App\Http\Controllers\PostController;
use App\Http\Controllers\ReplyController;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/

Route::get('/', function () {
    return view('welcome');
});

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth'])->name('dashboard');

require __DIR__.'/auth.php';

// group the following routes by auth middleware - you have to be signed-in to proceeed
Route::group(['middleware' => 'auth'], function() {
	// Dashboard
	Route::get('/home', [HomeController::class, 'index'])->name('home');

	// Posts resourcefull controllers routes
	Route::resource('posts', PostController::class);

	// Comments routes
	Route::group(['prefix' => '/comments', 'as' => 'comments.'], function() {
        // store comment route
		Route::post('/{post}', [CommentController::class, 'store'])->name('store');
	});

	// Replies routes
	Route::group(['prefix' => '/replies', 'as' => 'replies.'], function() {
        // store reply route
		Route::post('/{comment}', [ReplyController::class, 'store'])->name('store');
	});
});
```

Les routes accessibles après s'être authentifié doivent utiliser le middleware _auth_

## Bootstrap 4

Nous allons créer l'arborescence de dossier _resources/views/posts/partials_ pour héberger nos blocks de code. <br>

home.blade.php
```php
@extends('layouts.app')

@section('content')

<div class="clearfix">
    <h2 class="float-left">List of all projects</h2>

    {{-- link to create new post --}}
    <a href="{{ route('posts.create') }}" class="btn btn-link float-right">Create new post</a>
</div>

{{-- List all posts --}}
@forelse ($posts as $post)
    <div class="card m-2 shadow-sm">
        <div class="card-body">

            {{-- post title --}}
            <h4 class="card-title">
                <a href="{{ route('posts.show', $post->id) }}">{{ $post->title }}</a>
            </h4>

            <p class="card-text">
                
                {{-- post owner --}}
                <small class="float-left">By: {{ $post->owner->name }}</small>

                {{-- creation time --}}
                <small class="float-right text-muted">{{ $post->created_at->format('M d, Y h:i A') }}</small>
                
                {{-- check if the signed-in user is the post owner, then show edit post link --}}
                @if (auth()->id() == $post->owner->id )
                    {{-- edit post link --}}
                    <small class="float-right mr-2 ml-2">
                        <a href="{{ route('posts.edit', $post->id) }}" class="float-right">edit your post</a>
                    </small>
                @endif
            </p>
        </div>
    </div>
@empty
    <p>No posts yet, stay tuned!</p>
@endforelse

@endsection
```

Dans _resources/views/layouts/app.blade.php_ nous ajoutons l'élément :
```php
<main class="container col-md-8 py-4">
    @yield('content')
</main>
```

Dans le dossier _posts_, nous créons les fichiers :
* create.blade.php
* edit.blade.php
* show.blade.php

create.blade.php
```php

{{-- extends the layouts/app.blade.php --}}
@extends('layouts.app')

@section('content')

	{{-- Start card --}}
    <div class="card shadow">
        <div class="card-body">
            <h4 class="card-title">Create new post</h4>
            <div class="card-text">
                @include('posts.partials.create_post')
            </div>
        </div>
    </div>
  {{-- End --}}
    
@endsection
```

show.blade.php
```php
@extends('layouts.app')

@section('content')

	{{-- show post --}}
	@include('posts.partials.post')

@endsection
```

edit.blade.php
```php
@extends('layouts.app')

@section('content')
    {{-- show edit post form --}}
    @include('posts.partials.edit_post')
@endsection
```
