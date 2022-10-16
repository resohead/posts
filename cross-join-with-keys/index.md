---
title: Cross Join With Keys
slug: cross-join-with-keys
description: Use cross join to generate combinations of associative arrays.
date: 2022-08-15
tags: [laravel, collections, macro, snippet]
sources: []
---

# Cross Join With Keys

Laravel ships with `Arr::crossJoin` and `$collection->crossJoin` helpers that generate the cartesian product (every combination) of each array. This is useful for building a single matrix from multiple arrays.

This is simple using the array method + spread operator but requires an extra step to use keys with collections.

Skip to the [Collection macro](#collection-macro) section to get a snippet that allows you to cross join collections using keys.

## Cross join assoc array (spread)
```php title="Array helper"
$builds = [
  'php' => [7.3, 7.4, 8.0],
  'stability' => ['prefer-lowest', 'prefer-stable'],
  'laravel' => ['6.x', '7.x', '8.x']
];

$matrix = Arr::crossJoin(...$builds);
```

```php title="Output"
[
  [
    "php" => 7.3,
    "stability" => "prefer-lowest",
    "laravel" => "6.x",
  ],
  [
    "php" => 7.3,
    "stability" => "prefer-lowest",
    "laravel" => "7.x",
  ],
  // more items...
  [
    "php" => 8.0,
    "stability" => "prefer-stable",
    "laravel" => "7.x",
  ],
  [
    "php" => 8.0,
    "stability" => "prefer-stable",
    "laravel" => "8.x",
  ],
]
```

## Using Collections

If we use a similar approach with collections we do not get the same output.

```php title="Collection"
collect([
  'php' => [7.3, 7.4, 8.0],
  'laravel' => ['6.x', '7.x', '8.x']
])
->crossJoin([
  'stability' => ['prefer-lowest', 'prefer-stable']
]);
```

```php title="Output"
[
    [
      [
        7.3,
        7.4,
        8.0,
      ],
      [
        "prefer-lowest",
        "prefer-stable",
      ],
    ],
    [
      [
        "6.x",
        "7.x",
        "8.x",
      ],
      [
        "prefer-lowest",
        "prefer-stable",
      ],
    ],
  ],
```

Oops, that's not what we want.

## Cross join without keys then map

In order to produce the same output as the original `Arr::crossJoin` we could collection and cross join the array values and map the keys back in at the end.

```php title="Cross join" comment="or by using array combine..."
collect([7.3, 7.4, 8.0])
    ->crossJoin(
        ['prefer-lowest', 'prefer-stable'],
        ['6.x', '7.x', '8.x']
    )
    ->map(fn($build) => [
        'php' => $build[0],
        'stability' => $build[1],
        'laravel' => $build[2],
    ]);
```

```php title="Array combine"
collect([7.3, 7.4, 8.0])
    ->crossJoin(
        ['prefer-lowest', 'prefer-stable'],
        ['6.x', '7.x', '8.x']
    )
    ->map(fn ($build) =>
        array_combine(['php', 'stability', 'laravel'], $build)
    )
```

```php diff title="Diff"
 collect([7.3, 7.4, 8.0])
    ->crossJoin(
        ['prefer-lowest', 'prefer-stable'],
        ['6.x', '7.x', '8.x']
    )
    ->map(fn ($build) =>
-        [
-            'php' => $build[0],
-            'stability' => $build[1],
-            'laravel' => $build[2],
-        ]
+        array_combine(['php', 'stability', 'laravel'], $build)
    )
```

This would work but depending on how your original data is coming in it could get a bit messy. Let's create a collection macro to behave in a similar way to the array functions.

## Collection macro

Add the following macro to a service provider. It merges any given items with the existing collection and spreads into the array cross join function.

```sh title="Terminal"
php artisan make:provider CollectionServiceProvider
```

```php file="AppServiceProvider.php"
Collection::macro('crossJoinWithKeys', function ($items = null) {
    $items = $this->merge($this->getArrayableItems($items));

    return new Collection(
        Arr::crossJoin(...$items)
    );
});
```

This allows us to collect our builds using `crossJoinWithKeys`

```php title="crossJoinWithKeys" comment="or even without arguments..."
collect([
  'php' => [7.3, 7.4, 8.0],
  'laravel' => ['6.x', '7.x', '8.x']
])
->crossJoinWithKeys([
  'stability' => ['prefer-lowest', 'prefer-stable']
]);
```

```php title="Without arguments" comment="and still receive the same output"
collect([
  'php' => [7.3, 7.4, 8.0],
  'laravel' => ['6.x', '7.x', '8.x'],
  'stability' => ['prefer-lowest', 'prefer-stable']
])
->crossJoinWithKeys();
```s

```php title="Output"
[
  [
    "php" => 7.3,
    "stability" => "prefer-lowest",
    "laravel" => "6.x",
  ],
  [
    "php" => 7.3,
    "stability" => "prefer-lowest",
    "laravel" => "7.x",
  ],
  // more items...
  [
    "php" => 8.0,
    "stability" => "prefer-stable",
    "laravel" => "7.x",
  ],
  [
    "php" => 8.0,
    "stability" => "prefer-stable",
    "laravel" => "8.x",
  ],
]
```
