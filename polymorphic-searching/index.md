---
title: Laravel polymorphic searching
slug: laravel-polymorphic-searching
description: Filter by parent polymorphic models in Laravel apps.
date: 2022-08-15
tags: [php, laravel, snippet, eloquent, query]
sources: []
---

# Laravel polymorphic searching

Filter by parent morph models types:

```php

// Model: Comment

/**
* Get the parent commentable model (post or video).
*/
public function subject(): MorphTo
{
    return $this->morphTo();
}

public function scopeBlogs()
{
    return $this->whereHasMorph('subject', [Blog::class]);
}

```

```php
// Find all comments that belong to Blog post models
Comment::whereHasMorph('subject', [Blog::class])->get();

// Search all comments that belong to Blog post models and eager load it's blog post and the user
Comment::query()
    ->where('comment', 'like', "%{$search}%")
    ->blogs()
    ->with(['subject', 'user'])
    ->get();
```

## Global search

I will be writing an article on 'global searching' and rendering results on the front-end soon but in the meantime checkout the [Laravel Cross Eloquent Search by ProtoneMedia](https://github.com/protonemedia/laravel-cross-eloquent-search) or searching across eloquent models.