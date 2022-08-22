---
title: Extend Container Bindings
slug: extend-container-bindings
description: How to build upon existing container values.
date: 2022-08-16
tags: [laravel, container, extending, snippet]
sources:
    - https://twitter.com/timacdonald87/status/1309044900869140481
    - https://laravel.com/docs/container#extending-bindings
---

# Extending Laravel container bindings

If you want to decorate something bound to the container in Laravel you can `extend` it. This allows you to pipe the value through some particular changes without having to re-bind the original value.

Notice in the example below we extend a value (22) bound to the container via the 'twenty-two' key and then extend it again referencing the previously bound value.

```php
app()->bind('twenty-two'), fn() => 22);

app('twenty-two');
// 22

app()->extend('twenty-two'), fn($base) => $base * 2);

app('twenty-two');
// 44

app()->extend('twenty-two'), fn($base) => $base + 6);
// 50
```