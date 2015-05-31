## CakePimpleDi - Cake 3 Plugin
A cakephp plugin for dependency injection based on [Pimple 3](http://pimple.sensiolabs.org)

## Requirements
* CakePHP 3.X
* [Pimple 3](http://pimple.sensiolabs.org)

# Installation
---------
To install this plugin you'll only have to add the plugin to your required section of composer.json then load in your bootstrap file.

```
composer require rochamarcelo/cake-pimple-di:dev-master
```

# Bootstrap
---------

Load the plugin as any other plugin:

```php
Plugin::load('RochaMarcelo/CakePimpleDi', ['bootstrap' => true, 'routes' => false]);
```

The bootstrap file must be loaded, to set up all configurations needed

## Register dependencies

In your configuration file you can define all the services:

```php
<?php
return [
    ...

    'CakePimpleDi' => [
        'services' => [
            'LibraryApp\Client' => function() {//each time you get that service, will returns the same instance
                return new \Cake\Network\Http\Client;
            },
            'LibraryApp\Finder' => function($c) {//each time you get that service, will returns  the same instance
                $finder = new \LibraryApp\Finder\SimpleFinder(
                	$c['LibraryApp\Client']
                );

                return $finder;
            },
            'random_func' => [
                'value' => function () {
                    return rand();
                },
                'type' => 'parameter'//when you get that service, will return the original closure
            ],
            'cookie_name' => 'SESSION_ID',
            [
                'id' => 'something',
                'value' => function () {
                    $std = new \stdClass;
                    $std->rand = rand();
                    return $std;
                },
                'type' => 'factory'//will return  a different instance for all calls
            ]
        ]
    ]
];
```

You could also create a provider to reuse some services that you may use in other projects.

- So create the provider that implements Pimple\ServiceProviderInterface:
```php
<?php
namespace App\Di;

use Pimple\Container;
use Pimple\ServiceProviderInterface;

class LibraryAppProvider implements ServiceProviderInterface
{

    public function register(Container $pimple)
    {

        $pimple['LibraryApp\Client'] = function() {//each time you get that service, will returns the same instance
                return new \Cake\Network\Http\Client;
        };

        $pimple['LibraryApp\Finder'] = function($c) {//each time you get that service, will returns  the same instance
            $finder = new \LibraryApp\Finder\SimpleFinder(
                $c['LibraryApp\Client']
            );
            return $finder;
        };
    }
}
```
- Then, define in your configuration file:

```php
<?php
return [
    ...

    'CakePimpleDi' => [
        'providers' => [
            'App\Di\LibraryAppProvider'
        ],
        'services' => [
            'random_func' => [
                'value' => function () {
                    return rand();
                },
                'type' => 'parameter'//when you get that service, will return the original closure
            ]
        ]
    ]
];
```
## Loading dependency - Basic
Get the shared instance then call method "get"

```php
use RochaMarcelo\CakePimpleDi\Di\Di;
$finder = Di::instance()->get('LibraryApp\Finder');
```

## Loading dependency - With DiTrait

```php

namespace App\Controller;

use RochaMarcelo\CakePimpleDi\Di\DiTrait;

class BooksController extends AppController
{
    use DiTrait;

    public function index()
    {
        $finder = $this->di()->get('LibraryApp\Finder');
    }
}
```

## Loading dependency - Injections with component
In yout controller load the Di component defining the injections you want

```php
class MyControllerController extends AppController
{
    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('RochaMarcelo/CakePimpleDi.Di', [
            'injections' => [
                'view' => [//View action
                    'MovieApp\Finder',//First argument
                    'something'//Second argument
                ],
                'edit' => [//Edit action
                    'MovieApp\Finder'//First argument
                ]
            ]
        ]);
    }

    public function view(\CakeFinder $finder, $something, $id = null)
    {
        $finder->find();
        .....
    }

    public function edit($finder, $id = null)
    {
        $finder->find();
        $something->doSomething();
    }
}
```

## What is Pimple?

Pimple is a simple PHP Dependency Injection Container, to more information visit: http://pimple.sensiolabs.org

