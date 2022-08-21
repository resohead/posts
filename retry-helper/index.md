---
title: Retry helper
slug: retry-helper
description: Attempt to execute a callback until it succeeds or the maximum retry threshold is met and exception is thrown.
date: 2022-08-15
tags: [php, laravel, snippet, queues, reference]
sources: []
---

# Retry

Attempt to execute a callback until it succeeds or the maximum retry threshold is met and exception is thrown.

```php
retry($times, $callback, $sleepMilliseconds, $when)
```

```php
$valueOfResult = retry(
    times: 10, // max number of retries before throwing exception
    callback: function(){
        // get result from API or other error-prone operation
        // if successful returns the value to the caller
        return $result;
    })
    sleepMilliseconds: 50, // try again after 50ms
    when: function($exception) {
        // dynamically decide if we should retry based on the exception
        // return true if you want to retry
        return true
    }
```
