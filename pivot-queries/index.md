---
title: Pivot queries
slug: pivot-queries
description: How to query additional attributes on pivot tables.
date: 2022-08-15
tags: [php, laravel, snippet, query, eloquent, pivot]
sources: []
---

# Pivot queries

In the examples below we will assume a user can be an athlete with one or more coaches via a pivot table with an addition `role` column to identify their head coach compared to the rest of the team.

- users (id, name, etc...)
- athlete_user (athlete_id, user_id, role)

```php
// App\Models\User

public function coaches(): BelongsToMany
{
    return $this->belongsToMany(User::class, 'athlete_user', 'athlete_id', 'user_id')
        ->withPivot('role');
}
```

```php
// return users with a collection of their coaches
User::with('coaches')->get();

// return users with a collection of their head coaches only
User::with([
    'coaches' => fn ($coach) => $coach->where('role','head-coach')
])->get();

// return users that have themselves as a coach
User::whereHas('coaches', fn ($coach) =>
    $coach->whereColumn('athlete_id', 'users.id')
)
->get()

// return users filtered by a pivot value with a potentially ambiguous column name
User::whereHas('coaches', fn ($coach) =>
    $coach->whereNull('athlete_user.updated_at')
)
->get()

// return users without themselves as a member
User::whereDoesntHave('coaches', fn ($coach) =>
    $coach->whereColumn('athlete_id', 'users.id')
)
->get()
```