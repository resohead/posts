---
title: The Ultimate Checklist for Your Laravel App Service Provider
slug: app-service-provider-checklist
description: Tips and tricks to help you get the most out of your Laravel App Service Provider.
excerpt: The App Service Provider is the heart of your Laravel application. It is the first thing that gets loaded when your application starts and is responsible for bootstrapping your application. This checklist will help you make sure you are getting the most out of your App Service Provider.
date: 2024-06-23
tags: [php, laravel, providers]
sources: []
---

# App Service Provider Checklist

The App Service Provider is the heart of your Laravel application. It is the first thing that gets loaded when your application starts and is responsible for bootstrapping your application. This checklist will help you make sure you are getting the most out of your App Service Provider.

## Register vs boot

- `register` is for bindings and singletons
- `boot` is for everything else

## Models

```php file="app/Providers/AppServiceProvider.php"
class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $this
            ->configureDates()
            ->configureModels()
            ->configureResponses();
    }

    protected function configureDates(): self
    {
        Date::use(CarbonImmutable::class);

        return $this;
    }

    protected function configureResponses(): self
    {
        Http::preventStrayRequests();

        JsonResource::withoutWrapping();

        return $this;
    }

    protected function configureModels(): self
    {
        Model::unguard();
        
        Model::preventLazyLoading(app()->isLocal());
        
        Relation::enforceMorphMap([
            'user' => \App\Models\User::class,
        ]);

        // Model::preventAccessingMissingAttributes(app()->isLocal());

        // Model::preventSilentlyDiscardingAttributes(app()->isLocal());

        // DB::whenQueryingForLongerThan(100, function ($query, $time) {
        //     logger()->warning('Slow query', [
        //         'query' => $query,
        //         'time' => $time,
        //     ]);
        // });

        return $this;
    }
}
```

## Request IDs via context

```php file="app/Providers/AppServiceProvider.php"
public function register(): void
{
    $this->app->singleton('request-id', fn() => (string) Str::uuid());
}

public function boot(): void
{
    Log::withContext(['request_id' => app('request-id')]);
}
```

In Laravel 11 you can now use the `Context` facade designed to help capture and share information throughout requests, jobs, commands and logs. https://laravel.com/docs/11.x/context

For example, creating a `AddRequestContextMiddleware`:

```php file="app/Http/Middleware/AddRequestContextMiddleware.php"
class AddRequestContextMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('request_id', (string) Str::uuid());
 
        return $next($request);
    }
}
```

This will share context automatically instead of remembering to pass it to across your application:
```php
ProcessPodcast::dispatch($podcast);

Log::info('Processing podcast.', [
    'podcast_id' => $this->podcast->id,
]);
```

or can be retireved manually:
```php
Context::get('url');
Context::get('request_id');
```

## Helpers
Register `helpers.php` file:

```json file="composer.json"
{
    "autoload": {
        "files": [
            "app/helpers.php"
        ]
    }
}
```

## Local development API token middleware

As a bonus, I also like to add a middleware that allows me to authenticate as a specific user by using the user ID as the API token. This is useful for local development when you want to test the application as a specific user without having to generate a new API token each time.

Find out how to [authenticate with dev tokens](./development-api-tokens).
