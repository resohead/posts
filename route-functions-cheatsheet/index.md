---
title: Route functions cheatsheet
slug: route-functions-cheatsheet
description: A cheatsheet for request and route functions in Laravel applications.
date: 2022-08-15
tags: [php, laravel, snippet, cheatsheet, routes, request, parameters, query string, macros]
sources: []
---

# Route functions cheatsheet

Laravel comes with a number of helpers around the routes and current request which we can use to compare the current page with navigation links and breadcrumbs to determine active states etc.

We will mostly be using the route class `Illuminate\Routing\Route` via the facade `Illuminate\Support\Facades\Route` and the request class `Illuminate\Http\Request` with the `request()` helper.

## Route

### Current

```php
request()->route() // Illuminate\Routing\Route
```

### Name

```php
Route::currentRouteName(); // "welcome", "api-tokens.index"
```

### Is
```php
Route::is('articles.index') // boolean - exact
Route::is('articles.*') // boolean  - wildcard
```

### Action

```php

Route::current()->action
[
  "domain" => null
  "middleware" => [
    "web",
    "auth:sanctum",
    "Laravel\Jetstream\Http\Middleware\AuthenticateSession",
    "verified"
  ]
  "uses" => "Laravel\Jetstream\Http\Controllers\Livewire\ApiTokenController@index"
  "controller" => "Laravel\Jetstream\Http\Controllers\Livewire\ApiTokenController@index"
  "namespace" => "Laravel\Jetstream\Http\Controllers"
  "prefix" => ""
  "where" => []
  "as" => "api-tokens.index"
]

Route::currentRouteAction()

//'Laravel\Jetstream\Http\Controllers\Livewire\ApiTokenController@index'
```

### Parameters

`\Request::route()->parameters`

```php
// routes/web.php
Route::view('/', 'welcome',
    ['foo' => 'bar'],
    200,
    ['x-custom' => true]
)->name('home');

\Request::route()->parameters;

// returns
[
  "view" => "welcome"
  "data" => [
    "foo" => "bar"
  ]
  "status" => 200
  "headers" => [
    "x-custom" => true
  ]
]
```

### List

```php

Route::getRoutes()->getRoutes()
[
    Illuminate\Routing\Route {287 ▶},
    Illuminate\Routing\Route {288 ▶},
    // ...
]

Route::getRoutes()->getRouteByName()
[
    'login' => Illuminate\Routing\Route {287 ▶},
    'password.request' => Illuminate\Routing\Route {288 ▶},
    'api-tokens.index' => Illuminate\Routing\Route {289 ▶},
    // ...
]

Route::getRoutes()->getRoutesByMethod()
[
    'GET' => [
        '/route/uri/with/{parameters}' => Illuminate\Routing\Route {321 ▶},
    ],
    'HEAD' => [],
    'POST' => [],
    'PUT' => [],
    'DELETE' => [],
]
```

### Previous route name

```php
$previousRoute = app('router')->getRoutes()->match(
        app('request')->create(url()->previous())
    );

-$previousRouteName = $previousRoute->getName();
```

You could add this to the Route macros (or helpers file) to make it available as a shortcut:
```php
// in a service provider
Illuminate\Support\Facades\Route::macro('previousRoute', function(): Illuminate\Routing\Route {
    $url = url()->previous();

    return app('router')->getRoutes()->match(
        app('request')->create($url)
    );
});

// use
Route::previousRoute()->getName();
```


## URL

### Current
```php
request()->url()
```

### Is
```php
request()->is('articles')
request()->is('articles/*')
request()->routeIs('articles.index')

request()->fullUrl() // http://example.com/articles/my-first-post?foo=bar
request()->fullUrlIs('http://example.com/articles/my-first-post?foo=bar')

```

### Segment
```php
// example.com/admin/cities/create?foo=bar

request()->segments() // ['admin', 'cities', 'create']

request()->segment(2) //cities
```

### Query string

```php
// example.com/search?country=GB&category=Business

request()->all()
// ['country' => 'GB', 'category' => 'Business']

request()->get('country')
request()->get('country', 'GB')
request()->get('country') ?? 'GB'

```

## Components

```php
// NavigationLink.blade.php

<li class="{{ \Illuminate\Support\Str::startsWith(request()->url(), $href) ? 'active' : ''  }}">
    <a href="{{ $href }}" @isset($dataDirtyWarn) data-dirty-warn @endisset>
        {{ $slot }}
    </a>
</li>
```

```php
// link with text
<x-navigation-item :href="route('home')" :label="home"></x-navigation-item>


// link with icon
<x-navigation-item :href="route('forum.threads')">
    <x-icon-label icon="fa-users" text="New" :count="$threads->unread()->count() ?? 0" />
</x-navigation-item>
```

## Further reading

Keep an eye out for an article about route parameter binding in Laravel soon where we will look at how Laravel resolves a route, binds parameters (including implicit model binding) and executes middleware before passing to a controller or closure.