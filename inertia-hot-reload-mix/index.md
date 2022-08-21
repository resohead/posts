---
title: Inertia hot reload with Laravel mix
slug: inertia-hot-reload-mix-with-laravel-mix
description: How to use hot module replacement with Laravel mix and Inertia.js
date: 2021-08-16
tags: [php, laravel, inertia, js, homestead, windows, snippet]
sources:
    - https://laravel-mix.com/docs/5.0/browsersync
    - https://github.com/BrowserSync/browser-sync/issues/684
---

# Inertia hot reload with Laravel mix

The following steps show how you can use hot module replacement in Laravel Mix through Homestead on Windows:

1. Add script `"hot": "mix watch --hot"` to `package.json`
2. Call browserSync from Mix using a proxy in `webpack.mix.js`

```js
mix.browserSync({
    proxy: 'laravel-app-name.test',
    // or using .env with MIX_ prefix
    // proxy: process.env.MIX_APP_URL,

    socket: {
        domain: 'localhost:3000'
    }
});

```
> This should install any additional dev dependencies.

> Remember to restart the npm process to ensure your webpack changes are picked up

Optional:
```
# use the APP_URL defined at the start of a standard Laravel installation
MIX_APP_URL="${APP_URL}"
```

1. Use the `mix()` in your base layout for styles and scripts
```html
<head>
    <link rel="stylesheet" href="{{ mix('css/app.css') }}">
    <script src="{{ mix('js/app.js') }}" defer></script>
</head>
<body>
<!-- content -->

<!-- browser sync assets -->
@env ('local')
    <script src="http://localhost:3000/browser-sync/browser-sync-client.js"></script>
@endenv
</body>
```
4. Start `npm run hot`
5. Any changes to your Laravel application code should be reflected in the current browser with any existing state.

For reference:
* https://laravel-mix.com/docs/5.0/browsersync
* https://github.com/BrowserSync/browser-sync/issues/684