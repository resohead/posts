---
title: Fail Validation
slug: fail-validation
description: Throw validation errors anywhere
excerpt: Laravel's validation system takes care of all the dirty work for you, including redirecting the user back to the previous page
date: 2022-08-16
tags: [laravel, snippet, validation, errors, exceptions]
sources: []
---

# Fail Validation

Laravel's validation system takes care of all the dirty work for you, including redirecting the user back to the previous page with the correct status code and storing your errors on the session for easier display. This allows you to push validation error messages anywhere in the application.

```php
use Illuminate\Validation\ValidationException;

throw ValidationException::withMessages([
    'field_name' => ['Custom message here'],
]);
```