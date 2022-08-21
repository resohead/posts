---
title: Qualify column
slug: qualify-column
description: Easily prepend your columns with the table name.
date: 2022-08-15
tags: [php, laravel, snippet]
sources: []
---

# Qualify column

Laravel comes with a helpful model method to easily prepend your columns with the table name.

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

`getQualifiedKeyName`
```php
App\Models\User::find(1)->getQualifiedKeyName();
// posts.id
```