---
title: Laravel redis cache tips
slug: laravel-redis
description: Clearing the redis cache in Laravel apps.
date: 2022-08-15
tags: [laravel, redis, snippet, cache, deployment, queue]
sources: []
---

# Laravel redis cache tips

Learn how to use redis databases along with tags to isolate cache for sites sharing a redis server.

- [Redis CLI](#redis-cli)
- [Prefixes](#prefixes)
- [Flushing](#flushing)
- [Database number](#database-number)
- [Tags](#tags)
- [Caching paginated results](#caching-paginated-results)

## Redis CLI
```bash
$ redis-cli

127.0.0.1:6379> keys *
(empty list or set)
```

Redis has 16 databases indexed 0 - 15, and the default database index is 0.
As of Laravel > 5.7, Laravel stores all the cache data in database index 1.

In order to query database 1. You can either use the -n switch on the command line to specify the database index, or use the select command at the redis prompt to change the active database.

```bash
redis-cli -n 1 keys "*"
```
or
```bash
$ redis-cli
127.0.0.1:6379> select 1
127.0.0.1:6379[1]> keys *

#1) "{project}_database_{project}_cache_:{key}"
```

## Prefixes

Laravel cache default:
```php
// config/cache.php

'prefix' => env('CACHE_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_cache'),
// demo_cache
```

Laravel redis default:
```php
// config/database.php
'redis' => [
        // ...
        'options' => [
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
            // demo_database_
        ],
]
```

Laravel will use the redis and cache prefix as the full cache key:

`redis prefix` = `{project}_database`

`cache prefix` = `{project}_cache`

`path` = `{redis_prefix}_{cache_prefix}`

For example, `{project}_database_{project}_cache_:{key}`

## Flushing

The Laravel `cache:clear` artisan command does not take cache prefixes, defined in your app config, into account. The prefix is not a cache namespace but just a string on the front of a key name.

If you are deploying a staging and production site to the same server they will share the same redis cache database (i.e. 1) and will wipe both caches even if the artisan command is run via the other site or on deployment.

To ensure you only clear the cache in the correct environment you can use on of the following options:

## Database number

In your `.env` file add the following key that maps to config `database.redis.cache`
```
# Production
REDIS_CACHE_DB=1

# Staging
REDIS_CACHE_DB=2
```

Note: you might need to restart redis for this to take effect using your server management tool or:
```
/etc/init.d/redis-server {start|stop|restart|force-reload}
```

Clearing the cache will now only clear the cache in the current database.

## Tags

Alternatively, you could use [cache tags](https://laravel.com/docs/cache#storing-tagged-cache-items) to separate caches instead of using databases.

Be careful! This is not a simple config change - you will need to implement this in your business logic by setting/flushing the cache tag on the item and scoping the artisan command.

Cache tags are usually used against a domain (e.g. products, reviews). But you could also use the environment as a tag. Let's take a look

> not all cache drivers support tags - check the Laravel documentation for an updated list.

Set the tags in your application code:
```php
$env = app()->environment(); // production

Cache::tags([$env, 'reviews'])
    ->put('reviews', $reviews, $seconds);
```

Now we can target any tags with the `tags` parameter on artisan command
```
php artisan cache:clear --tags=production,reviews
```

or in your app:
```php
Cache::tags([$env])->flush();
```

Maybe you could set a default env tag on every push to cache?

## Caching paginated results

Here's a basic example of using cache tags and a dynamic key name to cache reviews by page.

Each request to a new page will store the records forever until a new review has been approved and the cache will be cleared across all pages by targeting the 'reviews' tag.

> Note: caching forever is not ideal as it can lead to stale records hanging around if cache keys change or code is removed etc.

```php
class ReviewController extends Controller
{
    public function index(Request $request)
    {
        $page = $request->get('page', 1);

        $reviews = Cache::tags(['reviews'])
            ->rememberForever(
                "reviews:{$page}",
                fn () => Review::approved()->paginate(25);
            );

        return view('reviews.index')->with([
            'reviews' => $reviews,
        ]);
    }

    // ...

    // Signed URL action to approve new reviews
    public function approve($id)
    {
        Review::findOrFail($id)->approve();

        Cache::tags(['reviews'])->flush();

        return redirect(route('reviews.index'))
            ->with('success', 'Approved');
    }
}
```