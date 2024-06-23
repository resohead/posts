---
title: Inline CSS from file
slug: inline-css-from-file
description: How to inline a CSS from a file in a Laravel blade template.
date: 2024-06-23
tags: [php, laravel, css, snippet, vite, build, assets]
sources: []
---

# Inline CSS from file

## Build files

We usually access vite files from the blade directive

```php file="resources/views/layouts/master.php"
<html>
    <head>
        @vite(['resources/css/app/app.css'])
    </head>
    ...
</html>
```

But if you need to inline content build files from Vite you can use the `Vite` facade to get the built asset path and extract the contents. Let's create a helper function to get the contents of a built file from the public path.

```php file="app/helpers.php"
function vite_file(string $path, ?string $url = null): string
{
    $result = Str::after(Vite::asset($path), $url ?? url('/'));

    return public_file($result);
}

function public_file(string $path): string
{
    $public = public_path($path);

    if(file_exists($public)) {
        return file_get_contents($public);
    } 
    
    $exception = new Exception("Public file not found: {$path}");

    if(! app()->isLocal()) {
        report($exception);
        return '';
    }

    throw $exception;
}
```

Which can now be used in your views.

```php file="resources/views/layouts/master.php" comment="Note: we only do this in production as the build file will not be present in development mode."
<html>
    <head>
        <style>
            @if(Vite::isRunningHot())
                @vite(['resources/css/app/app.css'])
            @else
                {!! vite_file('resources/css/app.css') !!}
            @endif
        </style>
    </head>
    ...
</html>
```

## Existing files

You could also create a similar helper function to get the contents of a file from the resources directory if no build is required:

```php file="app/helpers.php" comment="Note: We report exceptions and then continue in live systemss"
function resource_file(string $path): string
{
    $resource = resource_path($path);

    if(file_exists($resource)) {
        return file_get_contents($resource);
    } 
    
    $exception = new Exception("Resource file not found: {$path}");

    if(! app()->isLocal()) {
        report($exception);
        return '';
    }

    throw $exception;
}
```

```php file="resources/views/layouts/app.blade.php"
<style>
    {!! resource_file('css/app.css') !!}
</style>
```

## Direct file include

You could also include the file directly and avoid any custom functions or build steps.

```blade
<style>
    @php
        include(resource_path('css/app.css'))
        include(public_path('css/app.css'))
    @endphp
</style>
```