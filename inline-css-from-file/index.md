---
title: Inline CSS from file
slug: inline-css-from-file
description: How to inline a CSS from a file in a Laravel blade template.
date: 2021-08-16
tags: [php, laravel, css, snippet]
sources: []
---

# Inline CSS from file

```blade
<style>
    @php
        include(public_path('css/app.css'))
    @endphp
</style>
```