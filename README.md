# laravel-base-repository
An abstract base repository with predefined common features.

[![Source Code](https://img.shields.io/badge/source-okipa/laravel--model--base--repository-blue.svg)](https://github.com/Okipa/laravel-base-repository)
[![Latest Version](https://img.shields.io/github/release/okipa/laravel-base-repository.svg?style=flat-square)](https://github.com/Okipa/laravel-base-repository/releases)
[![Total Downloads](https://img.shields.io/packagist/dt/okipa/laravel-base-repository.svg?style=flat-square)](https://packagist.org/packages/okipa/laravel-base-repository)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://scrutinizer-ci.com/g/Okipa/laravel-base-repository/badges/build.png?b=master)](https://scrutinizer-ci.com/g/Okipa/laravel-base-repository/build-status/master)
[![Code Coverage](https://scrutinizer-ci.com/g/Okipa/laravel-base-repository/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/Okipa/laravel-base-repository/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/Okipa/laravel-base-repository/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/Okipa/laravel-base-repository/?branch=master)

------------------------------------------------------------------------------------------------------------------------

## Before starting
The repository pattern has several objectives :
- Encourage development good practices (separation of concerns, code reusability, ...)
- Improve code testability

Before using this package, you should be familiar with the repository pattern, and especially with its Laravel implementation.  
You can know more about it by reading the several articles you'll can find on Internet about this.  
Here is one among others : https://medium.com/@jsdecena/refactor-the-simple-tdd-in-laravel-a92dd48f2cdd.

## Installation
The repository pattern setup is not complicated but requires several steps to be accomplished.  
Follow them one by one :

- Install the package with composer :
```bash
composer require okipa/laravel-base-repository
```

- Create a `app/Repositories` directory where you will store your different project repositories.

- Create your `app/Repositories/BaseRepositoryInterface.php` interface and your `app/Repositories/BaseRepository.php` abstract class :
```php
<?php

namespace App\Repositories;

interface BaseRepositoryInterface
{
    // add here your own custom method contracts (if necessary).
    // they will be implemented in all your repositories.
}
```

```php
<?php

namespace App\Repositories;

abstract class BaseRepository extends Okipa\LaravelBaseRepository\BaseRepository implements BaseRepositoryInterface
{
    // add here your own custom method declarations (if necessary).
    // they will be implemented in all your repositories.
}
```

- Create a first `UserRepositoryInterface` and its associated `UserRepository` :
```php
<?php

namespace App\Repositories\Users;

interface UserRepositoryInterface
{
    // add here the users method contracts.
    /**
     * @return void
     */
    public function test();
}
```

```php
<?php

namespace App\Repositories\Users;

use App\Repositories\BaseRepository;
use App\Models\User;

class UserRepository extends BaseRepository implements UserRepositoryInterface
{
    protected $model = User::class;

    // add here the users method declarations.
    public function test()
    {
        \Log::info('test');
        // manipulate your model as needed. Example : $this->model->create(['email' => 'whatever@email.test']);
    }
}
```

- Create your project `app/Providers/RepositoryServiceProvider.php` file. You can follow the example below :
```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Repositories\User\UserRepositoryInterface;

class RepositoryServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        // users
        $this->app->bind(UserRepositoryInterface::class, UserRepository::class);

        // then, register all your other repositories here ...
    }
}
```

- Add your `RepositoryServiceProvider` to the providers declarations of the Laravel framework, in the `config/app.php` :
```php
// ...

'providers' => [
    // other provider declarations ...

    // custom providers
    App\Providers\RepositoryServiceProvider::class,
],

// ...
```

- Add a `$repository` attribute to your `app/Http/Controllers/Controller.php` base controller that all your controllers will extends :
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Foundation\Bus\DispatchesJobs;
use Illuminate\Routing\Controller as BaseController;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Foundation\Auth\Access\AuthorizesRequests;

class Controller extends BaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;

    protected $repository;
}
```

And you're done !  
You can now use your UserRepository, empowered with the pre-defined methods provided by this package.

------------------------------------------------------------------------------------------------------------------------

## Usage
In your `app/Http/Controllers/Users/UsersController.php`, manipulate your `UserRepository` as in the example bellow :
```php
<?php

namespace App\Http\Controllers\Users;

use App\Http\Controllers\Controller;
use App\Repositories\Users\UserRepositoryInterface;

class UsersController extends Controller
{
    /**
     * UsersController constructor.
     *
     * @param \App\Repositories\Users\UserRepositoryInterface $repository
     */
    public function __construct(UserRepositoryInterface $repository)
    {
        parent::__construct();
        $this->repository = $repository;
    }

    /**
     * @param IndexUserRequest $request
     *
     * @return void
     */
    public function index(IndexUserRequest $request)
    {
        // execute your repository custom methods
        $this->repository->test();
        // execute this package methods
        $allStoredUsers = $this->repository->getAll();
    }
```

## API

### Properties

| Property | Visibility | Default value | Description |
|-----------|-----------|-----------|-----------|
| $model | protected |  | The repository associated main model. |
| $request | protected | request() | The repository associated request. |
| $defaultAttributesToExcept | protected | ['_token', '_method'] | Default attributes to automatically except from request treatments. |
| $exceptDefaultAttributes | protected | true | Automatically except defined $defaultAttributesToExcept from the request treatments. |

### Public methods

- `setModel(string $modelClass)`
    > set the repository model class to instantiate.
- `setRequest(\Illuminate\Http\Request $request)` : set the repository request to use.
- `createMultipleFromRequest(array $attributesToExcept = [], array $attributesToAddOrReplace = [])` : create multiple model instances from the request data. The use of this method suppose that your request is correctly formatted. If not, you can use the $exceptFromSaving and $addToSaving attributes to do so.
- `createMultipleFromArray(array $data)` : create one or more model instances from data array.
- `createOrUpdateFromRequest(array $attributesToExcept = [], array $attributesToAddOrReplace = [])` : create or update a model instance from the request data. The use of this method suppose that your request is correctly formatted. If not, you can use the $exceptFromSaving and $addToSaving attributes to do so.
- `createOrUpdateFromArray(array $data)` : create or update a model instance from array data. The use of this method suppose that your array is correctly formatted.
- `getModelPrimaryFromArray(array $data)` : get model primary value from a data array.
- `updateFromPrimary(int $instancePrimary, array $data)` : update a model instance from its primary key.
- `deleteFromRequest(array $attributesToExcept = [], array $attributesToAddOrReplace = [])` : destroy a model instance from the request data.
- `deleteFromArray(array $data)` : delete a model instance from a data array.
- `deleteFromPrimary(int $instancePrimary)` : delete a model instance from its primary key.
- `deleteMultipleFromPrimaries(array $instancePrimaries)` : delete multiple model instances from their primary keys.
- `paginateArrayResults(array $data, int $perPage = 20)` : paginate array results.
- `findOneFromPrimary(int $instancePrimary)` : find one model instance from its primary key value.
- `findOneFromArray(array $data)` : find one model instance from a « where » parameters array.
- `findMultipleFromArray(array $data)` : find multiple model instance from a « where » parameters array.
- `getAll($columns = ['*'], string $orderBy = 'default', string $orderByDirection = 'asc')` : get all model instances from database.

