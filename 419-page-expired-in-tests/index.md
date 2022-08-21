---
title: 419 page expired in tests
slug: 419-page-expired-in-tests
description: Clear you cache in your testing/development environment to prevent csrf verification errors.
date: 2022-08-15
tags: [php, laravel, tests, phpunit]
sources:
    - https://github.com/laravel/framework/issues/13374#issuecomment-239600163
---

# Test 419 Page Expired HTTP Response

## Problem
You submit a form in your phpunit tests and expect an HTTP ok (200), redirect (302) or created (201) response but instead receieve a 'page expired' (419).

Your test might look something like this:

```php
$response = $this->post('/contact-us', ['email' => 'test@example.com', 'message' => 'Hello world!'])
    ->assertStatus(302);
```

## Solution
Clear the configuration cache:

```
php artisan config:clear
```

## Explanation

PHPUnit will use the cached configuration values instead of the `.env.testing`. This causes the `APP_ENV` value to remian unchanged (e.g. local, development, production etc) instead of `testing`, the `VerifyCsrfTokenMiddleware` will throw a TokenMismatchException (Status code 419).

It won't throw this exception when the APP_ENV is set to testing since the handle method of VerifyCsrfTokenMiddleware checks if you are running unit tests with `$this->runningUnitTests()`.

It is **recommended not to cache your configuration in your development environment**. When you need to cache your configuration in the environment where you are also running unit tests you could clear your cache manually in your TestCase.php:

```php
public function createApplication()
{
    ....
    Artisan::call('config:clear')
    ....
}
```