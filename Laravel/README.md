# Laravel


## General

Standards: PSR-4, PSR-12



### FIRST RULE OF MRG DEV CLUB: Follow the "Laravel Way"

First and foremost, Laravel provides the most value when you write things the way Laravel intended you to write. If there's a documented way to achieve something, follow it. Whenever you do something differently, make sure you have a justification for *why* you didn't follow the defaults.



### **Single responsibility principle**

A class and a method should have only one responsibility.

Bad:

```php
public function getFullNameAttribute()
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

Good:

```php
public function getFullNameAttribute()
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient()
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong()
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort()
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```



### **Fat models, skinny controllers**

Put all DB related logic into Eloquent models or into Repository classes if you're using Query Builder or raw SQL queries.

Bad:

```php
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```

Good:

```php
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```



### **Don't repeat yourself (DRY)**

Reuse code when you can. SRP is helping you to avoid duplication. Also, reuse Blade templates, use Eloquent scopes etc.

Bad:

```php
public function getActive()
{
    return $this->where('verified', 1)->whereNotNull('deleted_at')->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->where('verified', 1)->whereNotNull('deleted_at');
        })->get();
}
```

Good:

```php
public function scopeActive($q)
{
    return $q->where('verified', 1)->whereNotNull('deleted_at');
}

public function getActive()
{
    return $this->active()->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```



### Nullable and Union Types

Whenever possible use the short nullable notation of a type, instead of using a union of the type with `null`.

Bad:

```php 
// in a class
public string | null $variable;
```

Good:

```php
// in a class
public ?string $variable;
```



###  Typed properties

You should type a property whenever possible. 

```php
class Foo
{
    public string $bar;
}
```



### Traits

Each applied trait should go on its own line, and the `use` keyword should be used for each of them. This will result in clean diffs when traits are added or removed.

Bad:

```php
class MyClass {
    use TraitA, TraitB;
}
```

Good:

```php
class MyClass {
    use TraitA;
    use TraitB;
}
```



### Strings

When possible prefer string interpolation above `sprintf` and the `.` operator.

Bad:

```php
$greeting = 'Hi, I am ' . $name . '.';
```

Good:

```php
$greeting = "Hi, I am {$name}.";
```



### Happy Path

Generally a function should have its unhappy path first and its happy path last. In most cases this will cause the happy path being in an unindented part of the function which makes it more readable.

Bad:

```php
if ($goodCondition) {
 // do work
}

throw new Exception;
```

Good:

```php
if (! $goodCondition) {
  throw new Exception;
}

// do work
```



### If Statements

Always use curly brackets.

Bad:

```php
if ($condition) ...
```

Good:

```php
if ($condition) {
   ...
}
```



### Avoid Else

In general, `else` should be avoided because it makes code less readable. In most cases it can be refactored using early returns. This will also cause the happy path to go last, which is desirable.

Bad:

```php
if ($conditionA) {
   if ($conditionB) {
      // condition A and B passed
   }
   else {
     // condition A passed, B failed
   }
}
else {
   // condition A failed
}
```

Good:

```php
if (! $conditionA) {
   // condition A failed

   return;
}

if (! $conditionB) {
   // condition A passed, B failed

   return;
}

// condition A and B passed
```

Another option to refactor an `else` away is using a ternary

Bad:

```php
if ($condition) {
    $this->doSomething();
} 
else {
    $this->doSomethingElse();
}
```

Good:
```php
$condition
    ? $this->doSomething();
    : $this->doSomethingElse();
```



### Compound Ifs

In general, separate `if` statements should be preferred over a compound condition. This makes debugging code easier.

Bad:

```php
if ($conditionA && $conditionB && $conditionC) {
  // do stuff
}
```

Good:

```php
if (! $conditionA) {
   return;
}

if (! $conditionB) {
   return;
}

if (! $conditionC) {
   return;
}

// do stuff

```

 

### Elseif

In the occasion it is necessary to use an ``else if`` statement, use the singular conditional ``elseif``

Bad:

```php
if($condition) {
  
} else if ($otherCondition) {
  
} 
```

Good:

```php
if($condition) {
  
} elseif ($otherCondition) {
  
}
```



### **Validation**

Prefer validation in Request classes instead of controllers.

Bad:

```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    ....
}
```

Good:

```php
public function store(PostRequest $request)
{    
    ....
}

class PostRequest extends Request
{
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}
```

When using multiple rules for one field in a form request, avoid using `|`, always use array notation. Using an array notation will make it easier to apply custom rule classes to a field and makes it more readable.

Bad:

```php
public function rules()
{
    return [
        'email' => 'required|email',
    ];
}
```

Good:

```php
public function rules()
{
    return [
        'email' => ['required', 'email'],
    ];
}
```

All custom validation rules must use snake_case:

```php
Validator::extend('organisation_type', function ($attribute, $value) {
    return OrganisationType::isValid($value);
});
```



### **Business logic should be in service class**

A controller must have only one responsibility, so move business logic from controllers to service classes.

Bad:

```php
public function store(Request $request)
{
    if ($request->hasFile('image')) {
        $request->file('image')->move(public_path('images') . 'temp');
    }
    
    ....
}
```

Good:

```php
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    ....
}

class ArticleService
{
    public function handleUploadedImage($image)
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```



### **Prefer to use Eloquent over using Query Builder and raw SQL queries. Prefer collections over arrays**

Eloquent allows you to write readable and maintainable code. Also, Eloquent has great built-in tools like soft deletes, events, scopes etc.

Bad:

```sql
SELECT *
FROM `articles`
WHERE EXISTS (SELECT *
              FROM `users`
              WHERE `articles`.`user_id` = `users`.`id`
              AND EXISTS (SELECT *
                          FROM `profiles`
                          WHERE `profiles`.`user_id` = `users`.`id`) 
              AND `users`.`deleted_at` IS NULL)
AND `verified` = '1'
AND `active` = '1'
ORDER BY `created_at` DESC
```

Good:

```php
Article::has('user.profile')->verified()->latest()->get();
```



### **Mass assignment**

Prefer mass assignment to setting individual attributes.

Bad:

```php
$article = new Article;
$article->title = $request->title;
$article->content = $request->content;
$article->verified = $request->verified;
// Add category to article
$article->category_id = $category->id;
$article->save();
```

Good:

```php
$category->article()->create($request->validated());
```



### **Do not execute queries in Blade templates and use eager loading (N + 1 problem)**

Bad (for 100 users, 101 DB queries will be executed):

```php
@foreach (User::all() as $user)
    {{ $user->profile->name }}
@endforeach
```

Good (for 100 users, 2 DB queries will be executed):

```php
$users = User::with('profile')->get();

...

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```



### Configuration

Configuration files must use kebab-case.

```
config/
  pdf-generator.php
```

Configuration keys must use snake_case.

```php
// config/pdf-generator.php
return [
    'chrome_path' => env('CHROME_PATH'),
];
```

Avoid using the `env` helper outside of configuration files. Create a configuration value from the `env` variable like above.

When adding config values for a specific service, add them to the `services` config file. Do not create a new config file.

Bad:

```php
//creating a new config file: `weyland-yutani.php`

return [
    'weyland_yutani' => [
        'token' => env('WEYLAND_YUTANI_TOKEN')
    ],  
]
```

Good:

```php
// adding credentials to `config/services.php`
return [
     'ses' => [
            'key' => env('SES_AWS_ACCESS_KEY_ID'),
            'secret' => env('SES_AWS_SECRET_ACCESS_KEY'),
            'region' => env('SES_AWS_DEFAULT_REGION', 'us-east-1'),
        ],
    
    'github' => [
        'username' => env('GITHUB_USERNAME'),
        'token' => env('GITHUB_TOKEN'),
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => env('GITHUB_CALLBACK_URL'),
        'docs_access_token' => env('GITHUB_ACCESS_TOKEN'),
    ],
    
    'weyland_yutani' => [
        'token' => env('WEYLAND_YUTANI_TOKEN')
    ],   
];
```



### **Do not get data from the `.env` file directly**

Pass the data to config files instead and then use the `config()` helper function to use the data in an application.

Bad:

```php
$apiKey = env('API_KEY');
```

Good:

```php
// config/api.php
'key' => env('API_KEY'),

// Use the data
$apiKey = config('api.key');
```



### **Do not put JS and CSS in Blade templates and do not put any HTML in PHP classes**

Bad:

```js
let article = `{{ json_encode($article) }}`;
```

Better:

```html
<input id="article" type="hidden" value='@json($article)'>

Or

<button class="js-fav-article" data-article='@json($article)'>{{ $article->name }}<button>
```

In a Javascript file:

```js
let article = $('#article').val();
```

The best way is to use specialized PHP to JS package to transfer the data.



### **Use standard Laravel tools accepted by community**

*Prefer* (and prefer is the operative word here) to use built-in Laravel functionality and community packages instead of using 3rd party packages and tools. Any developer who will work with your app in the future will need to learn new tools. Also, chances to get help from the Laravel community are significantly lower when you're using a 3rd party package or tool. Do not make your client pay for that.

| Task                      | Standard tools                         | 3rd party tools                                         |
| ------------------------- | -------------------------------------- | ------------------------------------------------------- |
| Authorization             | Policies                               | Entrust, Sentinel and other packages                    |
| Compiling assets          | Laravel Mix                            | Grunt, Gulp, 3rd party packages                         |
| Deployment                | Laravel Forge                          | Deployer and other solutions                            |
| Unit testing              | PHPUnit, Mockery                       | Phpspec                                                 |
| Browser testing           | Laravel Dusk                           | Codeception                                             |
| DB                        | Eloquent                               | SQL, Doctrine                                           |
| Templates                 | Blade                                  | Twig                                                    |
| Working with data         | Laravel collections                    | Arrays                                                  |
| Form validation           | Request classes                        | 3rd party packages, validation in controller            |
| Authentication            | Built-in                               | 3rd party packages, your own solution                   |
| API authentication        | Laravel Passport, Laravel Sanctum      | 3rd party JWT and OAuth packages                        |
| Creating API              | Built-in                               | Dingo API and similar packages                          |
| Working with DB structure | Migrations                             | Working with DB structure directly                      |
| Localization              | Built-in                               | 3rd party packages                                      |
| Realtime user interfaces  | Laravel Echo, Pusher                   | 3rd party packages and working with WebSockets directly |
| Generating testing data   | Seeder classes, Model Factories, Faker | Creating testing data manually                          |
| Task scheduling           | Laravel Task Scheduler                 | Scripts and 3rd party packages                          |



### **Follow Laravel naming conventions**

Follow [PSR standards](http://www.php-fig.org/psr/psr-2/).

Also, follow naming conventions accepted by Laravel community:

| What                             | How                                                          | Good                                    | Bad                                                 |
| -------------------------------- | ------------------------------------------------------------ | --------------------------------------- | --------------------------------------------------- |
| Controller                       | singular                                                     | ArticleController                       | ~~ArticlesController~~                              |
| Route                            | plural                                                       | articles/1                              | ~~article/1~~                                       |
| Named route                      | snake_case with dot notation                                 | users.show_active                       | ~~users.show-active, show-active-users~~            |
| Model                            | singular                                                     | User                                    | ~~Users~~                                           |
| hasOne or belongsTo relationship | singular                                                     | articleComment                          | ~~articleComments, article_comment~~                |
| All other relationships          | plural                                                       | articleComments                         | ~~articleComment, article_comments~~                |
| Table                            | plural                                                       | article_comments                        | ~~article_comment, articleComments~~                |
| Pivot table                      | singular model names in alphabetical order                   | article_user                            | ~~user_article, articles_users~~                    |
| Table column                     | snake_case without model name                                | meta_title                              | ~~MetaTitle; article_meta_title~~                   |
| Model property                   | snake_case                                                   | $model->created_at                      | ~~$model->createdAt~~                               |
| Foreign key                      | singular model name with _id suffix                          | article_id                              | ~~ArticleId, id_article, articles_id~~              |
| Primary key                      | -                                                            | id                                      | ~~custom_id~~                                       |
| Migration                        | -                                                            | 2017_01_01_000000_create_articles_table | ~~2017_01_01_000000_articles~~                      |
| Method                           | camelCase                                                    | getAll                                  | ~~get_all~~                                         |
| Method in resource controller    | [table](https://laravel.com/docs/master/controllers#resource-controllers) | store                                   | ~~saveArticle~~                                     |
| Method in test class             | camelCase                                                    | testGuestCannotSeeArticle               | ~~test_guest_cannot_see_article~~                   |
| Variable                         | camelCase                                                    | $articlesWithAuthor                     | ~~$articles_with_author~~                           |
| Collection                       | descriptive, plural                                          | $activeUsers = User::active()->get()    | ~~$active, $data~~                                  |
| Object                           | descriptive, singular                                        | $activeUser = User::active()->first()   | ~~$users, $obj~~                                    |
| Config and language files index  | snake_case                                                   | articles_enabled                        | ~~ArticlesEnabled; articles-enabled~~               |
| View                             | kebab-case                                                   | show-filtered.blade.php                 | ~~showFiltered.blade.php, show_filtered.blade.php~~ |
| Config                           | snake_case                                                   | google_calendar.php                     | ~~googleCalendar.php, google-calendar.php~~         |
| Contract (interface)             | adjective or noun                                            | AuthenticationInterface                 | ~~Authenticatable, IAuthentication~~                |
| Trait                            | adjective                                                    | Notifiable                              | ~~NotificationTrait~~                               |



### **Use shorter and more readable syntax where possible**

Bad:

```php
$request->session()->get('cart');
$request->input('name');
```

Good:

```php
session('cart');
$request->name;
```

More examples:

| Common syntax                                                | Shorter and more readable syntax                   |
| ------------------------------------------------------------ | -------------------------------------------------- |
| `Session::get('cart')`                                       | `session('cart')`                                  |
| `$request->session()->get('cart')`                           | `session('cart')`                                  |
| `Session::put('cart', $data)`                                | `session(['cart' => $data])`                       |
| `$request->input('name'), Request::get('name')`              | `$request->name, request('name')`                  |
| `return Redirect::back()`                                    | `return back()`                                    |
| `is_null($object->relation) ? null : $object->relation->id`  | `optional($object->relation)->id`                  |
| `return view('index')->with('title', $title)->with('client', $client)` | `return view('index', compact('title', 'client'))` |
| `$request->has('value') ? $request->value : 'default';`      | `$request->get('value', 'default')`                |
| `Carbon::now(), Carbon::today()`                             | `now(), today()`                                   |
| `App::make('Class')`                                         | `app('Class')`                                     |
| `->where('column', '=', 1)`                                  | `->where('column', 1)`                             |
| `->orderBy('created_at', 'desc')`                            | `->latest()`                                       |
| `->orderBy('age', 'desc')`                                   | `->latest('age')`                                  |
| `->orderBy('created_at', 'asc')`                             | `->oldest()`                                       |
| `->select('id', 'name')->get()`                              | `->get(['id', 'name'])`                            |
| `->first()->name`                                            | `->value('name')`                                  |



### **Use IoC container or facades instead of new Class**

new Class syntax creates tight coupling between classes and complicates testing. Use IoC container or facades instead.

Bad:

```php
$user = new User;
$user->create($request->validated());
```

Good:

```php
public function __construct(User $user)
{
    $this->user = $user;
}

....

$this->user->create($request->validated());
```



### **Store dates in the standard format. Use accessors and mutators to modify date format**

Bad:

```php
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->toDateString() }}
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->format('m-d') }}
```

Good:

```php
// Model
protected $dates = ['ordered_at', 'created_at', 'updated_at'];
public function getSomeDateAttribute($date)
{
    return $date->format('m-d');
}

// View
{{ $object->ordered_at->toDateString() }}
{{ $object->ordered_at->some_date }}
```



### **Other good practices**

Never put any logic in routes files.

Minimize usage of vanilla PHP in Blade templates.



### Artisan commands

The names given to artisan commands should all be kebab-cased.

Bad:

```php
php artisan deleteOldRecords
```

Good:

```bash
php artisan delete-old-records
```

A command should always give some feedback on what the result is. Minimally you should let the `handle` method spit out a comment at the end indicating that all went well.

```php
// in a Command
public function handle()
{
    // do some work

    $this->comment('All ok!');
}
```

When the main function of a result is processing items, consider adding output inside of the loop, so progress can be tracked. Put the output before the actual process. If something goes wrong, this makes it easy to know which item caused the error.

At the end of the command, provide a summary on how much processing was done.

```php
// in a Command
public function handle()
    {
    $this->comment("Start processing items...")

    // do some work
    $items->each(function(Item $item) {
        $this->info("Processing item id `{$item-id}`...")

        $this->processItem($item)
    });

    $this->comment("Processed {$item->count()} items.");
}
```



### Routing

Public-facing urls must use kebab-case.

```php
https://example.com/open-source
https://example.com/jobs/front-end-developer
```

Prefer to use the route tuple notation when possible.

Bad:

```php
Route::get('open-source', 'OpenSourceController@index');
```

Good:

```php
Route::get('open-source', [OpenSourceController::class, 'index']);
```



Route names must use camelCase.

Bad:

```php
Route::get('open-source', [OpenSourceController::class, 'index'])->name('open-source');
```

Good:

```php
Route::get('open-source', [OpenSourceController::class, 'index'])->name('openSource');
```



All routes have an http verb, that's why we like to put the verb first when defining a route. It makes a group of routes very readable. Any other route options should come after it.

Bad:

```php
// bad: http verbs not easily scannable
Route::name('home')->get('/', [HomeController::class, 'index']);
Route::name('openSource')->get([OpenSourceController::class, 'index']);
```

Good:

```php
// good: all http verbs come first
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('open-source', [OpenSourceController::class, 'index'])->name('openSource');
```

Route parameters should use camelCase.

```php
Route::get('news/{newsItem}', [NewsItemsController::class, 'index']);
```

A route url should not start with `/` unless the url would be an empty string.

Bad:

```php
Route::get('', [HomeController::class, 'index']);
Route::get('/open-source', [OpenSourceController::class, 'index']);
```

Good:

```php
Route::get('/', [HomeController::class, 'index']);
Route::get('open-source', [OpenSourceController::class, 'index']);
```



### Controllers

Try to keep controllers simple and stick to the default CRUD keywords (`index`, `create`, `store`, `show`, `edit`, `update`, `destroy`). Extract a new controller if you need other actions.

In the following example, we could have `PostsController@favorite`, and `PostsController@unfavorite`, or we could extract it to a separate `FavoritePostsController`.

```php
class PostsController
{
    public function create()
    {
        // ...
    }

    // ...

    public function favorite(Post $post)
    {
        request()->user()->favorites()->attach($post);

        return response(null, 200);
    }

    public function unfavorite(Post $post)
    {
        request()->user()->favorites()->detach($post);

        return response(null, 200);
    }
}
```

Here we fall back to default CRUD words, `store` and `destroy`.

```php
class FavoritePostsController
{
    public function store(Post $post)
    {
        request()->user()->favorites()->attach($post);

        return response(null, 200);
    }

    public function destroy(Post $post)
    {
        request()->user()->favorites()->detach($post);

        return response(null, 200);
    }
}
```

This is a loose guideline that doesn't need to be enforced.



### Views

View files must use camelCase.

```php
resources/
  views/
    openSource.blade.php
class OpenSourceController
{
    public function index() {
        return view('openSource');
    }
}
```



### Policies and Authorization

Prefer to use model policies for authorization



### Return Types

Add return types to function declarations.  If a method returns nothing, it should be indicated with `void`. This makes it more clear to the users of your code what your intention was when writing it.

```php
function returnIntValue(int $value): int 
{
      return $value;
}

// in a Laravel model
public function scopeArchived(Builder $query): void
{
    $query->
        ...
}
```



## Commenting and PHPDoc

### DocBlock

2. Every function must have a summary, return, and params if applicable
2. Highly encourage using descriptions unless it's a trivial function
3. Making use of tags where applicable is highly encouraged
4. Can use ``` // TODO ``` comments throughout code
5. It would be ideal to have DocBlocks at the file and class level as well (?)

```php
/**
 * Summary which must end with period or two line breaks.
 *
 * Optional longer description or discussion that may contain 
 * inline tags and some html markup. Separated by 
 * blank lines, this is used in page-level DocBlocks and in 
 * element-level DockBlocks when the element merits further discussion.
 * Section may contain markdown
 * 
 * @param <type> <name> <description>
 * @return <type> (What is being returned)
 */
```

If a variable has multiple types, the most common occurring type should be first.

Bad:

```php
/** @var null|\Foo\Bar */
```

Good:

```php
/** @var \Foo\Bar|null */
```



### **Comment your code, but prefer descriptive method and variable names over comments**

Bad:

```php
if (count((array) $builder->getQuery()->joins) > 0)
```

Better:

```php
// Determine if there are any joins.
if (count((array) $builder->getQuery()->joins) > 0)
```

Good:

```php
if ($this->hasJoins())
```



## Naming Classes

Naming things is often seen as one of the harder things in programming. That's why we've established some high level guidelines for naming classes.

### Controllers

Generally controllers are named by the singular form of their corresponding resource and a `Controller` suffix. This is to avoid naming collisions with models that are often equally named.

e.g. `UserController` or `EventDayController`

When writing non-resourceful controllers you might come across invokable controllers that perform a single action. These can be named by the action they perform again suffixed by `Controller`.

e.g. `PerformCleanupController`

### Resources and Transformers

Both Eloquent resources and Fractal transformers are plural resources suffixed with `Resource` or `Transformer` accordingly. This is to avoid naming collisions with models.

### Jobs

A job's name should describe its action.

E.g. `CreateUser` or `PerformDatabaseCleanup`

### Events

Events will often be fired before or after the actual event. This should be very clear by the tense used in their name.

E.g. `ApprovingLoan` before the action is completed and `LoanApproved` after the action is completed.

### Listeners

Listeners will perform an action based on an incoming event. Their name should reflect that action with a `Listener` suffix. This might seem strange at first but will avoid naming collisions with jobs.

E.g. `SendInvitationMailListener`

### Commands

To avoid naming collisions we'll suffix commands with `Command`, so they are easiliy distinguisable from jobs.

e.g. `PublishScheduledPostsCommand`

### Mailables

Again to avoid naming collisions we'll suffix mailables with `Mail`, as they're often used to convey an event, action or question.

e.g. `AccountActivatedMail` or `NewEventMail`





---

## Links

https://github.com/alexeymezenin/laravel-best-practices/blob/master/README.md

https://spatie.be/guidelines/laravel-php

https://www.php-fig.org/psr/psr-4/

https://www.php-fig.org/psr/psr-12/