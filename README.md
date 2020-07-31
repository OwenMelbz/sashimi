# Sashimi 🍣
Eloquent's missing "array" driver.

Sometimes you want to use Eloquent, but without dealing with a database.

## Important Notice

This is a fork of https://github.com/calebporzio/sushi - however it allows you to run Sushi without having a full Laravel installation, allowing you to pull it into other frameworks, it's 90% Celebs code, with additional work-arounds to avoid core Laravel features such as the IoC Container, and global helpers.

## Install
```
composer require owenmelbz/sashimi
```

## Use

Using this package consists of two steps:
1. Add the `Sashimi` trait to a model.
2. Add a `$rows` property to the model.

That's it.

```php
class State extends Model
{
    use \Sashimi\Sashimi;

    protected $rows = [
        [
            'abbr' => 'NY',
            'name' => 'New York',
        ],
        [
            'abbr' => 'CA',
            'name' => 'California',
        ],
    ];
}
```

Now, you can use this model anywhere you like, and it will behave as if you created a table with the rows you provided.
```php
$stateName = State::whereAbbr('NY')->first()->name;
```

This is really useful for "Fixture" data, like states, countries, zip codes, user_roles, sites_settings, etc...

### Relationships
Let's say you created a `Role` model, based on an array using Sashimi, that looked like this:
```php
class Role extends Model
{
    use \Sashimi\Sashimi;

    protected $rows = [
        ['id' => 1, 'label' => 'admin'],
        ['id' => 2, 'label' => 'manager'],
        ['id' => 3, 'label' => 'user'],
    ];
}
```

You can add a relationship to another standard model, just like you normally would:
```php
class User extends Model
{
    ...

    public function role()
    {
        return $this->belongsTo(Role::class);
    }
}
```

Assuming the `users` table has a `role_id` column, you can do things like this:
```php
// Grab a User.
$user = User::first();
// Grab a Role.
$role = Role::whereLabel('admin')->first();

// Associate them.
$user->role()->associate($role);

// Access like normal.
$user->role;

// Eager load.
$user->load('role');
User::with('role')->first();
```

> Note: There is one caveat when dealing with Sashimi model relationships. The `whereHas` method will NOT work. This is because the two models are spread across two separate databases.

### Custom Schema
If Sashimi's schema auto-detection system doesn't meet your specific requirements for the supplied row data, you can customize them with the `$schema` property or the `getSchema()` method.

```php
class Products extends Model
{
    use \Sashimi\Sashimi;

    protected $rows = [
        ['name' => 'Lawn Mower', 'price' => '226.99'],
        ['name' => 'Leaf Blower', 'price' => '134.99'],
        ['name' => 'Rake', 'price' => '9.99'],
    ];

    protected $schema = [
        'price' => 'float',
    ];
}
```

## How It Works
Under the hood, this package creates and caches a SQLite database JUST for this model. It creates a table and populates the rows. If, for whatever reason, it can't cache a .sqlite file, it will default to using an in-memory sqlite database.

## Using ->getRows()
You can optionally opt out of using the `protected $rows` property, and directly implement your own `getRows()` method.

This will allow you to determine the rows for the model at runtime. You can even generate the model's rows from an external source like a third-party API.

> Note: If you choose to use your own ->getRows() method, the rows will NOT be cached between requests.

```php
class Role extends Model
{
    use \Sashimi\Sashimi;

    public function getRows()
    {
        return [
            ['id' => 1, 'label' => 'admin'],
            ['id' => 2, 'label' => 'manager'],
            ['id' => 3, 'label' => 'user'],
        ];
    }
}
```
