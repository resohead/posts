---
title: Qualify column
slug: qualify-column
description:
date: 2022-08-15
tags: [php, laravel, snippet]
sources: []
---

# Qualify column


`Illuminate\Database\Eloquent\Builder`

```php
/**
 * Qualify the given column name by the model's table.
 *
 * @param  string|\Illuminate\Database\Query\Expression  $column
 * @return string
 */
public function qualifyColumn($column)
{
    return $this->model->qualifyColumn($column);
}
```

`Illuminate\Database\Eloquent\Model`

```php
/**
 * Qualify the given column name by the model's table.
 *
 * @param  string  $column
 * @return string
 */
public function qualifyColumn($column)
{
    if (str_contains($column, '.')) {
        return $column;
    }

    return $this->getTable().'.'.$column;
}
```

`getQualifiedKeyName`
```php
App\Models\User::find(1)->getQualifiedKeyName();
// posts.id
```