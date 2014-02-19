# Keeper - Laravel authentication driver "eloquent" extension

Keeper is Laravels native authentication driver "eloquent" extension. It adds roles and permissions functionality by extending the Guard class. All methods provided by Guard via Auth facade will work!

Keeper is not CRUD for user/role/permission database manipulation. There are just too many ways people would like to do it. We would end up with another Sentry and tonns of wierd exceptions which need to be catched. And that is no fun...

Keeper requires atleast Laravel 4.1 and PHP 5.4

## Todo

- cache db queries

## Install

To install Keeper package with composer edit composer.json file and run `composer update`.

	"require": {
	    "laravel/framework": "4.1.*",
	    "keevitaja/keeper": "dev-master"
	},

Change Auth driver in app/config/auth.php

	'driver' => 'keeper',

Add service provider to app/config/app.php

	'Keevitaja\Keeper\KeeperServiceProvider'

Add facade to app/config/app.php

	'Keeper' => 'Keevitaja\Keeper\Facades\Keeper'

Migrate tables

	php artisan migrate --package=keevitaja/keeper

After that you have users, roles and permissions tables with pivot tables.

	users:          id, email, password, created_at, updated_at
	roles:          id, name, created_at, updated_at
	permissions:    id, name, created_at, updated_at

To get a hint, how to name roles and permissions, take a look at the Usage example below.

If you need extra columns you can add them by creating your own migrations or copy migrations shipped with this package to your `app/database/migrations` folder and update them as needed and then do `php artisan migrate`.

## Usage

- `Keeper::hasRole($userId, $roleName)` - Determine if user has a role

- `Keeper::hasPermission($userId, $permissionName)` - Determine if user has a permission

- `Auth::hasRole($roleName)` - Determine if logged user has a role

- `Auth::hasPermission($permissionName)` - Determine if logged user has a permission

All above methods return boolean.

`::hasRole` checks only, if user has a role. This method does not check permissions!

`::hasPermission` checks, if user has permission permission directly or by a role.

User can have roles and permissions. Role can have permissions as well. User will have also all permissions from the role he/she has. Roles and permission work extremly well with laravel route and filter system. It just makes sense to use them together. See the Usage example below.

## Managing roles and permissions

Keeper does not provide CRUD for database manipulation. All model methods are abstracted into traits, so it would be possible to use relations really easy in other Eloquent models in your project. Just for the small example, the next line will give user with ID of 1 the role with ID of 3.

	Keevitaja\Keeper\Models\User::findUser(1)->roles()->attach(3)

For findUser() please see Custom exceptions below.

## Usage example

```php
Route::filter('finance', function()
{
	if ( ! Auth::hasRole('finance')) dd('no finance role');
});

Route::filter('invoices.create', function()
{
	if ( ! Auth::hasPermission('invoices.create')) dd('no create permission');
});

Route::filter('invoices.update', function()
{
	if ( ! Auth::hasPermission('invoices.update')) dd('no update permission');
});

Route::filter('invoices.destroy', function()
{
	if ( ! Auth::hasPermission('invoices.destroy')) dd('no destroy permission');
});

Route::group(['prefix' => 'invoices', 'before' => 'finance'], function()
{
	Route::get('show', 'InvoicesController@show');
	Route::get('create', 'InvoicesController@create')->before('invoices.create');
	Route::get('destroy', 'InvoicesController@destroy')->before('invoices.destroy');
});

Route::get('invoices/update', 'InvoicesController@update')->before('invoices.update');
```

Filter names in this example are the role and permission names. You can name permissions any way you like, but `controller.permission` seems to make sense. At least for me.

For this example to work you need a role 

- finance 

and permissions

- invoices.create
- invoices.update
- invoices.destroy

These routes and filters give you the following setup:

- `invoices/show` can be accessed by all users who have `finance` role
- `invoices/create` can be accessed by all users who have `finance` role and `invoices.create` permission
- `invoices/destroy` can be accessed by all users who have `finance` role and `invoices.destroy` permission
- `invoices/update` can be accessed by all users who have `invoices.update` permission

It does not matter, if permission is given to user directly (permission_user pivot) or through a role (permission_role pivot).

## Custom exceptions

If provided user ID does not have match in database, following exception will be thrown.

	Keevitja\Keeper\Models\Exceptions\UserNotFoundException

If you want to use this exception, swap User::find() with User::findUser().

Same goes for role and permission model.

	Keevitja\Keeper\Models\Exceptions\RoleNotFoundException
	Keevitja\Keeper\Models\Exceptions\PermissionNotFoundException

## If you like this 

please follow me [@keevitaja](https://twitter.com/keevitaja)