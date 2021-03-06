# Models

[Since 0.5.0]

<i class="fa fa-wrench fa-2x" aria-hidden="true"></i> Not implemented yet! - Planned release: 0.6.0 


- [Introduction](#introduction1)
- [Defining Models](#defining)
    - [Naming Conventions](#naming-conventions)
    - [General Settings](#general-settings)
    - [Attributes](#attributes)
    - [Getter and Setter](#getter-and-setter)
    - [Relationships](#relationships)
    - [Validation Rules](#validations)    
    - [Events](#events)    
- [Retrieving Models](#retrieving)
    - [Retrieving All Models](#all)
    - [Retrieving a Single Model](#find)
    - [Using Query Builder](#queries)
    - [Eager Loading](#eager-loading)
- [Modification Models](#modification)
    - [Inserting](#inserting)
    - [Updating](#updating)
    - [Deleting](#deleting)
- [Miscellanea Functions](#misc)
    
<a name="introduction"></a>
## Introduction

Pletfix provides a simple ActiveRecord implementation for working with your database. 
It is inspirated by [Ruby On Rails](http://guides.rubyonrails.org/active_record_basics.html).

Each database table has a corresponding "Model" which is used to interact with that table. Models allow you to query 
for data in your tables, as well as insert new records into the table.

Before getting started, be sure to [configure a database connection](database#configuration) in `config/database.php`. 

<a name="defining"></a>
## Defining Models

A Pletfix model extends the `Core\Services\Contracts\Model` contract and is stored in the `app/Models` directory.
Here is an minimal example of what such a model class might look like:

    class Flight extends Model
    {
    }
    
Yes, that's all!    
    
<a name="naming-conventions"></a>
### Naming Conventions

By default, Pletfix uses the following naming conventions:

- Database Table:  Plural with underscores separating words (snake_case, e.g., `book_clubs`).
- Model Class:     Singular with the first letter of each word capitalized (PascalCase, e.g., `BookClub`).
- Foreign keys:    These fields should be named following the pattern singularized_table_name_id (e.g., `item_id`, `order_id`). 
- Primary keys:    By default, the model will use an integer column named `id` as the table's primary key. 
- Table Column:    Underscores separating words (snake_case, e.g. `created_at`).
- Model Attribute: Like the bounded table column (snake_case, e.g. `created_at`).
- N:M Relationship Table: Singular with an underscore separated table names, alphabetical ordered (e.g. `role_user`)

<a name="general-settings"></a>
### General Settings

<div class="method-list" markdown="1">

[Database Store](#store)
[Database Table](#table)
[Identity Field](#identity)
[Timestamps](#timestamps)
[Create and Updater](#creater-and-updater)

</div>

<a name="store"></a>
#### Database Store {.method .first-method}

By default, all models will use the default database store configured for your application. 
If you would like to specify a different store for the model, use the `$store` property:

    class Flight extends Model
    {
        /**
         * The database store name for the model.
         *
         * @var string
         */
        protected $store = 'store-name';
    }    

<a name="table"></a>
#### Database Table {.method}

Note that we did not set which table to use for our `Flight` model. 
By convention, the "snake case", plural name of the class will be used as the table name unless another name is 
explicitly specified. So, in this case, Eloquent will assume the `Flight` model stores records in the `flights` table. 
You may specify a custom table by defining a `table` property on your model:

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'flights';
    }

<a name="identity"></a>
#### Identity Field {.method}

The Model will also assume that each table has a primary key column named `id`. You may define a `$primaryKey` property 
to override this convention.

    class Flight extends Model
    {
        /**
         * The primary key for the model.
         *
         * @var string
         */
        protected $primaryKey = 'id';
    }

<a name="timestamps"></a>
#### Timestamps {.method}

By default, Pletfix expects `created_at` and `updated_at` columns to exist on your tables. If you do not wish to have 
these columns automatically managed by Pletfix, set the `$timestamps` property on your model to `false`.

If you need to customize the names of the columns used to store the timestamps, you may set the `CREATED_AT` and 
`UPDATED_AT` constants in your model:

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = true;
        
        /**
         * The name of the "created at" column.
         *
         * @var string
         */
        const CREATED_AT = 'created_at';
    
        /**
         * The name of the "updated at" column.
         *
         * @var string
         */
        const UPDATED_AT = 'updated_at';
    }

<a name="creater-and-updater"></a>
#### Creater and Updater {.method}

If you wish to store the user automatically who created the model and who updated it last, set the `$creatorAndUpdater` 
property on your model to `true`. By default, this feature is disabled.

If you need to customize the names of the columns used to store the user, you may set the `CREATED_BY` and 
`UPDATED_BY` constants in your model:

    class Flight extends Model
    {
        /**
         * Indicates if the model should store the user who created the model and who updated it last.
         *
         * @var bool
         */
        protected $creatorAndUpdater = true;

        /**
         * The name of the "created by" column.
         *
         * @var string
         */
        const CREATED_BY = 'created_by';
    
        /**
         * The name of the "updated by" column.
         *
         * @var string
         */
        const UPDATED_BY = 'updated_by';
    }


<a name="attributes"></a>
### Attributes

The model automatically supplies all table columns of the tables as attributes, basically you don't have to do anything. 

<div class="method-list" markdown="1">

[Type Casting](#guarding)
[Guarding Attributes](#guarding)
[Searching Attributes](#searching)
[Accessors and Mutators](#accessors-and-mutators)

</div>

<a name="casting"></a>
#### Type Casting {.method .first-method}

To enjoy autocomplete of your IDE (e.g. PhpStorm) or to cast the type of the attribute, set the 
[PHPDoc's](https://www.phpdoc.org/docs/latest/references/phpdoc/tags/property.html) `@property` for each column:

    /**
     * @property integer $id Identifier
     * @property string $name Name of the flight
     */
    class Flight extends Model
    {
    }
        
The attributes types will be cast if you define it as above. The supported types are: 

`'int'`, `'float'`, `'string'`, `'boolean'`, `'object'`, `'array'`, `'collection'`, `'datetime'` and `'timestamp'`.

If you omit it, the attributes are treated as string.
       
<!--

TODO: Perhaps this is not needed because the type could be determine via reflection (s. "Attributes" above)

The `$casts` property lists the attributes that should be cast to native types. The supported cast types are: 

`'int'`, `'float'`, `'string'`, `'boolean'`, `'object'`, `'array'`, `'collection'`, `'datetime'` and `'timestamp'`.

    class Flight extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'foo' => 'int',
            'bar' => 'date',
        ];
    }
-->
    
<a name="guarding"></a>
#### Guarding Attributes {.method}
 
The `$guarded` property should contain an array of attributes that you do not want to be mass assignable. 
In the example below, all attributes except for 'id', 'created_by', 'updated_by', 'created_at' and 'updated_at' will 
be mass assignable via `update()` and `insert()`:

    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['id', 'created_by', 'updated_by', 'created_at', 'updated_at'];
    }
        
      
<a name="searching"></a>
#### Searching Attributes {.method}
    
The `$searchable` property is a convenient way to search for entries using search terms. For example, an application 
containing authors and books. For searching a book the user can enter one or more terms using a form like this:
  
    <form action="{{ url('books') }}" accept-charset="UTF-8" method="GET">
        <input name="search" type="text" placeholder="term1 term2 term3 ..."/>
        <button type="submit">Search</button>
    </form>
    
The relationship for this example is explained in the chapter about [one-to-many-relationship](#one-to-many).

It is to be searched for both the title and the author's name, so the `$searchable` property of the `Book` model have 
to be define like below:

    class Book extends Model
    {
        /**
         * Searchable fields.
         *
         * @var array
         */
        protected $searchable = [
            'title',
            'authors.name',
        ];      
    }
        
If you have set this property, the controller is able to call the `seach` method that takes the search terms as 
argument and returns a [`QueryBuilder`](queries) instance:

    /**
     * Lists all articles.
     *
     * @return Response|string
     */
    public function index()
    {
        $builder = new Article::builder();

        $searchTerms = request()->input('search');
        if (!empty($searchTerms)) {
            $builder = Article::search($searchTerms);
        }

        $articles = $builder->paginate(20);

        return view('articles.index', compact('articles'));
    }
    
    
<a name="accessors-and-mutators"></a>
#### Accessors and Mutators {.method}

The model stores the values of the table columns in the protected `$attributes` property. 
Accessors and mutators allow you to modifier the attribute values when you retrieve or set them on model instances. 

To define an accessor, create a `getFooAttribute` method on your model where `Foo` is the "studly" cased name of the 
column you wish to access. In the same way to define a mutator, create a `setFooAttribute` method:

    /**
     * Get the user's first name.
     *
     * @return string
     */
    public function getFirstNameAttribute()
    {
        return ucfirst($this->attributes['first_name']);
    }
    
    /**
     * Set the user's first name.
     *
     * @param  string  $value
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }  

To access the value of the accessor in the example above, you may simply access the `first_name` attribute on a model 
instance:

    $user = App\User::find(1);
    $user->first_name = 'Sally';
    echo $user->first_name;

      
<a name="relationships"></a>
### Relationships

<div class="method-list" markdown="1">

[One To One](#one-to-one)
[One To Many](#one-to-many)
[Many To Many](#many-to-many)
[Polymorphic Relations](#polymorphic)

</div>

<a name="one-to-one"></a>
#### One To One {.method .first-method}

A one-to-one relationship is a very basic relation. 
For example, each user in your application could (but does not have to) define their own avatar. So you need the 
following tables:    

![One-To-One-Relationship](https://raw.githubusercontent.com/pletfix/docs/master/images/one-to-one.png)

And your models looks like this:

    class User extends Model
    {
        /**
         * Define one-to-one relationships.
         */
        protected $hasOne = [
            App\Model\Avartar::class
        ];
        
        /**
         * Get the avatar associated with the user.
         */
        public function avartar()
        {
            return $this->hasOne(App\Model\Avartar::class);
        }
    }
    
    class Avartar extends Model
    {
        /**
         * Define inverse one-to-one or inverse one-to-many relationships.
         */
        protected $belongsTo = [
            App\Model\User::class
        ];

        /**
         * Get the user that owns the avartar.
         */
        public function user()
        {
            return $this->belongsTo(App\Model\User::class);
        }
    }
    
TODO: Remove one of the two variants in the example above.    
    
<a name="one-to-many"></a>
#### One To Many {.method}

This is similar to the one-to-one-relationship. 
For example, in an application containing authors and books, the models could be declared like this:

![One-To-Many-Relationship](https://raw.githubusercontent.com/pletfix/docs/master/images/one-to-many.png)

    class Author extends Model
    {
        /**
         * Define one-to-many relationships.
         */
        protected $hasMany = [
            App\Model\Book::class
        ];
        
        /**
         * Get the author's books.
         */
        public function books()
        {
            return $this->hasMany(App\Model\Book::class);
        }
    }
    
    class Book extends Model
    {
        /**
         * Define inverse one-to-one or inverse one-to-many relationships.
         */
        protected $belongsTo = [
            App\Model\Author::class
        ];
        
        /**
         * Get the author of the book.
         */
        public function author()
        {
            return $this->belongsTo(App\Model\Author::class);
        }
    }

TODO: Remove one of the two variants in the example above.

<a name="many-to-many"></a>
#### Many To Many {.method}

In this example, movies can be associated with one or more genres. On the other hand, you can find for a certain genre 
several movies. The relevant association declarations could look like this:

![Many-To-Many-Relationship](https://raw.githubusercontent.com/pletfix/docs/master/images/many-to-many.png)

    class Movie extends Model
    {
        /**
         * Define many-to-many relationships.
         */
        protected $belongsToMany = [
            App\Model\Genre::class
        ];
        
        /**
         * The genres of the movie.
         */
        public function genres()
        {
            return $this->belongsToMany(App\Model\Genre::class);
        }
    }
    
    class Genre extends Model
    {
        /**
         * Define many-to-many relationships.
         */
        protected $belongsToMany = [
            App\Model\Movie::class
        ];
        
        /**
         * The movies that belong to the genre.
         */
        public function movies()
        {
            return $this->belongsToMany(App\Model\Movies::class);
        }
    }
    
TODO: Remove one of the two variants in the example above.    
    
<a name="polymorphic"></a>
#### Polymorphic Relations {.method}

Polymorphic relations allow a model to belong to more than one other model on a single association.
For example, you might have a picture that belongs to either an employee or a product. 
First, let's examine the table structure required to build this relationship:

![Polymorphic-Relations](https://raw.githubusercontent.com/pletfix/docs/master/images/polymorphic.png)

Next, let's examine the model definitions needed to build this relationship:

    class Picture extends Model
    {
        /**
         * Get all of the owning imageable models.
         */
        public function imageable()
        {
            return $this->morphTo();
        }
    }
    
    class Employee extends Model
    {
        /**
         * Get all of the employee's images.
         */
        public function images()
        {
            return $this->morphMany('App\Model\Picture', 'imageable');
        }
    }
    
    class Product extends Model
    {
        /**
         * Get all of the products's images.
         */
        public function images()
        {
            return $this->morphMany('App\Model\Picture', 'imageable');
        }
    }

<a name="validations"></a>
### Validation Rules

<i class="fa fa-wrench fa-2x" aria-hidden="true"></i> Not implemented yet! - Planned release: 0.6.7

    /**
     * Validation rules.
     *
     * @var array
     */
    protected static $rules = [
    ];
    

<a name="events"></a>
### Events
    
<i class="fa fa-wrench fa-2x" aria-hidden="true"></i> Not implemented yet! - Planned release: 0.9.8

Events allow you to hook into the life cycle of your model, e.g. just before saving:  

    class Flight extends Model
    {
        /**
         * Called before a model is saved.
         */
        public function onSaving() {
            // ...
        });
    }

Pletfix support the following event handlers:
 
#### Before

- `onSaving`:   Called before a model is saved into the database.
- `onCreating`: Called before a new model is to be inserted into the database.
- `onUpdating`: Called before an existing model has been updated into the database.
- `onDeleting`: Called before a model has been deleted from the database

#### After

- `onSaved`:    Called after a model is saved into the database.
- `onCreated`:  Called after a new model has been inserted into the database.
- `onUpdated`:  Called after an existing model has been saved into the database.
- `onDeleted`:  Called after a model has been deleted from the database.


<a name="retrieving"></a>
## Retrieving Models

<a name="all"></a>
### Retrieving All Models

The `all` method will return a [`Collection`](collections) with all of the results in the model's table:

    $flights = Flight::all();
    
<a name="find"></a>
### Retrieving a Single Model

The `find` method is useful when a model is searched for its ID. The method returns a single model instance, or `null` 
if the model does not exist:

    $flight = Flight::find(4711);

You may also call the `find` method with an array of primary keys, which will return a [`Collection`](collections) of 
the matching records:

    $flights = Flight::find([1, 2, 3]);

<a name="queries"></a>
### Using Query Builder

Since each `Model` serves as a [query builder](queries), you may add constraints to queries, and then use the `all`, 
`cursor`, `first` or `get` method to retrieve the results:

    $flights = Flight::where('active', 1)
       ->orderBy('name', 'desc')
       ->take(10)
       ->all();

You may also use the `count`, `sum`, `max`, and other [aggregate methods](queries#aggregates) provided by the 
query builder. These methods return the appropriate scalar value instead of a full model instance:

    $count = App\Flight::where('active', 1)->count();
    $max   = App\Flight::where('active', 1)->max('price');

See [Query Builder](queries) for more details.

<a name="eager-loading"></a>
### Eager Loading

This point is important to understand! To explain, let's take our [book-author example](#one-to-many) once again. 
For each book you want to list their author. So you might write something like this:  

    $books = Books::all();     
    foreach($books as $book)
    {
        echo $book->author->name;
    }
    
By default the relationship attributes are **"lazy loading"**, meaning they will only load their data when you actually 
access them. So, if you run the code above, you would be execute the following database queries:

    SELECT * FROM books;
    SELECT * FROM authors WHERE id = 47;
    SELECT * FROM authors WHERE id = 98;
    SELECT * FROM authors WHERE id = 3;
    ...
    
You see, that is not very efficient. If you have 100 books, your database would be running 100 + 1 queries to run 
this little chunk of code. This is also known as the N + 1 problem.  

In this case it is better to use **"eager loading"** by calling the `with` method. 
The `with` method takes the relation table as argument and returns a [`QueryBuilder`](queries) instance, so you get 
all books as below:

    $books = Books::with('authors')->all();    // todo or ->get()? 
    foreach($books as $book)
    {
        echo $book->author->name;
    }
 
That reduce the operation to just two queries:
 
    SELECT * FROM books;
    SELECT * FROM authors WHERE id IN (47, 98, 3, ...)
 
 
<a name="modification"></a>
## Modification Models

<a name="inserting"></a>
### Inserting

To create a new record in the database, simply create a new model instance, set attributes on the model, then call the 
`save` method:

    $flight = new Flight;
    $flight->name = 'Albatros';
    $flight->save();

When we call the `save` method, a record will be inserted into the database. 
The `created_at` and `updated_at` timestamps will automatically be set when the `save` method is called, so there is 
no need to set them manually.

You can also use the `create` method to insert a new record in the database. 

    $flight = Flight::create(['name' => 'Flight 10']);

The `create` method returns the saved model instance.

> <i class="fa fa-exclamation-circle fa-2x" aria-hidden="true"></i>
> If you use the `create` method, you should specify the `guarded` attribute on the model, as all Models protect 
> against mass-assignment by default.

<a name="updating"></a>
### Updating

The `save` method may also be used to update models that already exist in the database. To update a model, you should 
retrieve it, set any attributes you wish to update, and then call the `save` method. Again, the `updated_at` timestamp 
will automatically be updated, so there is no need to manually set its value:

    $flight = Flight::find(1);
    $flight->name = 'New Flight Name';
    $flight->save();

You can also use the `update` method to update one or more model instances:

    $flights->update(['delayed' => 1]);

The `update` method expects an array of column and value pairs representing the columns that should be updated.

> <i class="fa fa-exclamation-circle fa-2x" aria-hidden="true"></i>
> If you use the `update` method, you should specify the `guarded` attribute on the model, as all Models protect
> against mass-assignment by default.

#### Fill Existing Model

If you already have a model instance, you may use the fill method to populate it with an array of attributes:

    $flight->fill(['name' => 'Flight 22']);

<a name="deleting"></a>
### Deleting

To delete a model, call the `delete` method on a model instance:

    $flight = Flight::find(1);
    $flight->delete();

In the example above, we are retrieving the model from the database before calling the `delete` method. However, if 
you know the primary key of the model, you may delete the model without retrieving it. To do so, call the `destroy` method:

    Flight::destroy(1);
    Flight::destroy([1, 2, 3]);

<a name="misc"></a>
## Miscellanea Functions

<div class="method-list" markdown="1">

[getOriginal](#method-get-original)
[isDirty](#method-is-dirty)

</div>

<a name="method-listing"></a>
### Method Listing

<a name="method-get-original"></a>
#### `getOriginal()` {.method .first-method}

Get the model's original attribute values.

    $flight = Flight::find(1);
    $flight->name = 'New Flight Name';
    echo $flight->getOriginal

<a name="method-is-dirty"></a>
#### `isDirty()`  {.method}

Determine if the model or given attribute(s) have been modified.

    $flight = Flight::find(1);
    $flight->name = 'New Flight Name';
    if ($flight->isDirty(['name'])) {
        echo 'The name was modified.!';
    }
