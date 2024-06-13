---
title: Authenticating local development API with User ID tokens
slug: development-api-tokens
description: Using User ID as API tokens for local development
date: 2024-06-13
tags: [php, laravel, snippet, api, authentication]
sources: []
---

# Authenticating local development API with User ID tokens

When developing an API, it is often useful to be able to authenticate as a specific user without having to re-run seeders or go through the login and API token creation process. 

Instead we can create a middleware that will allow us to authenticate as a specific user by using the user ID as the API token. We will fetch the given user by the ID and assign a temporary Personal Access Token with all abilities.

The following request will authenticate as the user with ID `1` by passing the ID as the bearer token instead of using the actual API key.

```shell diff
curl --request GET \
  --url 'https://app.test/api/projects/my-first-project' \
-  --header 'Authorization: Bearer trPeX25EcBQ6hl8J9ilYzWfWmGNtinqVMXnXDTJ8bd8b9332' \
+  --header 'Authorization: Bearer 1' \
  --header 'Accept: application/json' \
  --header 'Content-Type: application/json'
```

Create a new middleware class using the following code:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use App\Models\User;
use Laravel\Sanctum\PersonalAccessToken;

class AuthenticateWithDevelopmentApiTokens
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure(\Illuminate\Http\Request): (\Illuminate\Http\Response|\Illuminate\Http\RedirectResponse)  $next
     * @return \Illuminate\Http\Response|\Illuminate\Http\RedirectResponse
     */
    public function handle(Request $request, Closure $next)
    {
        if(! app()->environment(['local', 'development'])) {
            return $next($request);
        }

        $token = $request->bearerToken();

        if(! $token) {
            return $next($request);
        }

        if($user = User::find($token)) {
            $token = PersonalAccessToken::make([
                'token' => 'development',
                'abilities' => ['*'],
            ]);

            app('auth')->guard('sanctum')->setUser($user->withAccessToken($token));
        }

        return $next($request);
    }
}

```