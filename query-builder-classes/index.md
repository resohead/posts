---
title: Query Builder Classes
slug: query-builder-classes
description: Replace local query scopes with dedicated query builder classes
excerpt: Builder classes offer a way of encapsulating model query logic in a class with better autocompletetion
date: 2022-08-15
tags: [php, laravel, snippet, eloquent, query]
sources:
    - https://www.manifest.uk.com/blog/overriding-eloquent-global-scopes
    - https://timacdonald.me/dedicated-eloquent-model-query-builders/
---

# Laravel Eloquent Query Scopes and Builders

You could consider replacing local query scopes with dedicated query builder classes to tidy up models. You interact with this in the same way as local query scopes.

## Builder Classes

Builder classes offer a way of encapsulating model query logic in a class with better autocompletetion (no magic prefixes!).

### Registration

Let the model know what class to use when starting an eloquent query:

```php
class Article extends Model
{
    // ...

    public function newEloquentBuilder($query): Builder
    {
        return new ArticleEloquentBuilder($query);
    }

    // ...
}
```

### Builder

Create a the builder class to be used and include the query scopes (without the `scope` prefix):

```php
namespace App\EloquentBuilders;

use Illuminate\Database\Eloquent\Builder;

class ArticleEloquentBuilder extends Builder
{
    public function published(): self
    {
        return $this->where(fn ($query) =>
            $query
                ->whereNotNull('published_at')
                ->where('published_at', '<=', now())
        );
    }
}

```

Note: we have manually grouped the two where conditions in the `published` method of the custom builder class because Laravel only applies automatic groups to local scopes, e.g. `scopePublished()`.

You can view the Tim MacDonald's full write-up: https://timacdonald.me/dedicated-eloquent-model-query-builders/

### Usage

```php
Article::published()->get();
```

## Tip - Scope Classes

If you prefer to use [query scopes](https://laravel.com/docs/eloquent#query-scopes) here's a tip about overriding global scopes by extending the builder using a macro.

```php
namespace App\Scopes;

use Illuminate\Database\Eloquent\Scope;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class AgeScope implements Scope
{
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('age', '>=', 18);
    }

    public function extend(Builder $builder)
    {
        $builder->macro('includingYouths', function (Builder $builder) {
            return $builder->withoutGlobalScope($this);
        });
    }
}
```

Now you can call `includingYouths()` to exclude the global scope:

```php
User::includingYouths()->get();
```

View the full write up here: https://www.manifest.uk.com/blog/overriding-eloquent-global-scopes