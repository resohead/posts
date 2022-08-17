---
title: Blade Component With JavaScript
slug: blade-component-with-javascript
excerpt: A simple but powerful way ensure a blade component has all the scripts needed to render the view in one file
date: 2022-08-15
tags: [javascript, laravel, blade, snippet]
sources:
    - https://twitter.com/jeffrey_way/status/1292912263872032771
---

# Blade Component With Javascript

This is a snippet from Jeffrey Way but he has deleted the original tweet. It's a simple but powerful way ensure a blade component has all the scripts needed to render the view in one file. Do yourself a favour and get a [Laracasts](https://laracasts.com) account so you don't have to read this blog!

The following example uses Apline.js to trigger an instance of a different javascript library and `@once` to push scripts to the stack in your layout file.

```php
// <x-date-picker />
<input
    x-data
    x-init="new Cleave($el, {
        date: true,
        delimiter: '-',
        datePattern: ['Y', 'm', 'd']
    });"
    type="date"
    placeholder"yyyy-mm-dd"
    required
    {{ $attributes }}
>

@once
    @push
        <!-- you scripts here, e.g. Cleave.js from CDN -->
        <script src="cleave.js"></script>
    @endpush
@endonce

```

In Laravel `9.x` you can condense the code to push the stack using:

```
@pushOnce
    <!-- you scripts here, e.g. Cleave.js from CDN -->
    <script src="cleave.js"></script>
@pushOnce
```

`@once` is used to only push the script to the stack a single time regardless of how many times this component is used.
If you never call `<x-date-picker />`, you never pull in that script.

Alternatively, you can just manually import the script in your head tag but then it will be loaded on every request.