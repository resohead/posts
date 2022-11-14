---
title: How to implement social login in Laravel
slug: social-login-laravel
description:
date: 2022-11-13
tags: ['laravel', 'fortify', 'socialite', 'authentication', 'testing']
sources: []
---

# How to implement social login in Laravel

We will be using the following Laravel environment:

- Laravel 9.x
- PHP 8.1
- Jetstream/Breeze with teams
- Fortify
- Socialite
- Pest

## Summary

- Install `Laravel\Socialite`
- Create a social users table: `database/migrations/create_social_user_table.php`
- Create a social user model: `App/Models/SocialUser.php`
- Add required Oauth services: `config/services.php`
- Create a social login route with callback: `routes/web.php`
- Create handlers for login and callback routes: `App/Http/Controller/Auth/SocialLoginController.php`
- Make a method public: `App/Actions/Fortify/CreateNewUser.php`
- Create tests: `test/Feature/SocialLoginTest.php`

## Install Socialite

```sh
composer install Laravel\Socialite
```

## Implementation

- we want a user to be able to login by email and/or one or more social providers
- the user should be granted access to the same account when social details matches an existing user
- a personal team should be created for new accounts
- test new, existing and multiple social accounts

```php file="database/migrations/create_social_user_table.php"
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up()
    {
        Schema::create('social_user', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_id')->nullable();
            $table->string('provider');
            $table->string('provider_user_id');
            $table->string('email');
            $table->string('name');
            $table->string('avatar')->nullable();
            $table->timestamps();

            $table->unique(['provider', 'provider_user_id']);
        });
    }
};
```

```php file="app/Models/SocialUser.php"
namespace App\Models;

use App\Models\User;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class SocialUser extends Model
{
    use HasFactory;

    protected $table = 'social_user';

    protected $fillable = [
        'user_id',
        'provider',
        'provider_user_id',
        'email',
        'name',
        'avatar',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

```php file="config/services.php"
return [
    // ...
+    'github' => [
+        'client_id' => env('GITHUB_CLIENT_ID'),
+        'client_secret' => env('GITHUB_CLIENT_SECRET'),
+        'redirect' => '/login/github/callback',
+    ],
    // ...
];
```

```php file="routes/web.php"
// ...

+Route::get('login/{provider}', [SocialLoginController::class, 'redirectToProvider'])
+    ->name('login.social');
+
+Route::get('login/{provider}/callback', [SocialLoginController::class, 'handleProviderCallback'])
+    ->name('login.social.callback');

// ...
```

```php file="app/Http/Controller/Auth/SocialLoginController.php"
namespace App\Http\Controllers\Auth;

use App\Models\User;
use App\Models\SocialUser;
use Illuminate\Support\Str;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;
use App\Actions\Fortify\CreateNewUser;
use App\Providers\RouteServiceProvider;
use Laravel\Socialite\Facades\Socialite;
use Illuminate\Support\Facades\Hash;

class SocialLoginController extends Controller
{
    /**
     * List of providers configured in config/services acts as whitelist
     *
     * @var array
     */
    protected $providers = [
        'github',
        // gitlab
        // google
        // etc.
    ];

    /**
     * Where to redirect users after login.
     *
     * @var string
     */
    protected $redirectTo = RouteServiceProvider::HOME;

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }

    /**
     * Redirect the user to the GitHub authentication page.
     *
     * @return \Illuminate\Http\Response
     */
    public function redirectToProvider($provider)
    {
        if( ! $this->isProviderAllowed($provider) ) {
            return redirect(route('login'))->withError('Unable to login with this provider at the moment. Try a different provider to continue.');
        }

        return Socialite::driver($provider)->redirect();
    }

    /**
     * Obtain the user information from the social provider.
     *
     * @return \Illuminate\Http\Response
     */
    public function handleProviderCallback($provider)
    {
        try {
            $socialUser = Socialite::driver($provider)->user();
        } catch (Exception $e) {
            return redirect()->to(route('login.social'));
        }

        $user = $this->findOrCreateUser($socialUser, $provider);

        Auth::login($user, true);

        return redirect()->intended($this->redirectTo);
    }

    public function findOrCreateUser($socialite, string $provider)
    {
        $socialUser = SocialUser::firstOrCreate(
            [
                'provider' => $provider,
                'provider_user_id' => $socialite->getId()
            ],
            [
                'name' => $socialite->getName() ?? $socialite->getNickname(),
                'email' => $socialite->getEmail(),
                'avatar' => $socialite->getAvatar(),
            ]
        );

        if($socialUser->user_id){
            if($socialUser->user->profile_photo_path === null && $socialUser->avatar !== null){
                $socialUser->user->update(['profile_photo_path' => $socialUser->avatar]);
            }
            return $socialUser->user;
        }

        $user = User::firstOrCreate(
            [
                'email' => $socialUser->email
            ],
            [
                'name' => $socialUser->name,
                'email_verified_at' => now(),
                'password' => Hash::make(Str::random(16) . '' . now()),
                'profile_photo_path' => $socialUser->avatar,
            ]
        );

        if(! $user->personalTeam()){
            app(CreateNewUser::class)->createTeam($user);
        }

        if($user->profile_photo_path === null && $socialUser->avatar !== null){
            $user->update(['profile_photo_path' => $socialUser->avatar]);
        }

        $socialUser->update(['user_id' => $user->id]);

        return $user->refresh();
    }

    /**
     * Check for provider allowed and services configured
     *
     * @param $driver
     * @return bool
     */
    private function isProviderAllowed($driver)
    {
        return in_array($driver, $this->providers) && config()->has("services.{$driver}");
    }
}
```

```php file="app/Actions/Fortify/CreateNewUser.php"
    // ...

    ---protected--- +++public+++ function createTeam(User $user)
    {
        $user->ownedTeams()->save(Team::forceCreate([
            'user_id' => $user->id,
            'name' => explode(' ', $user->name, 2)[0]."'s Team",
            'personal_team' => true,
        ]));
    }

    // ...
```

```php file="test/Feature/SocialLoginTest.php"
use App\Models\User;
use Laravel\Socialite\Two\User as SocialiteUser;
use Laravel\Socialite\Facades\Socialite;

test('new users can register with social accounts', function () {
    $socialiteUser = Mockery::mock(SocialiteUser::class);

    $socialiteUser
        ->shouldReceive('getId')->andReturn('test-id')
        ->shouldReceive('getName')->andReturn('First Last')
        ->shouldReceive('getNickname')->andReturn('Foobar')
        ->shouldReceive('getEmail')->andReturn('first@example.com')
        ->shouldReceive('getAvatar')->andReturn('https://example.com/avatar.jpg');

    Socialite::shouldReceive('driver->user')
        ->andReturn($socialiteUser);

    $this->get('/login/github/callback')
        ->assertRedirect('/dashboard');

    $user = User::where('email', 'first@example.com')->first();

    expect($user)
        ->name->toBe('First Last')
        ->profile_photo_path->toBe('https://example.com/avatar.jpg');

    expect($user->socials)->toHaveCount(1);

    expect($user->socials->first())
        ->provider->toBe('github')
        ->provider_user_id->toBe('test-id')
        ->email->toBe('first@example.com')
        ->name->toBe('First Last')
        ->avatar->toBe('https://example.com/avatar.jpg');

    expect($user->allTeams())->toHaveCount(1);

    $this->assertAuthenticated();
});

test('existing users can login using social accounts', function(){

    $user = User::factory()->withPersonalTeam()->create([
        'name' => 'Sean',
        'email' => 'first@example.com',
        'profile_photo_path' => null
    ]);

    expect($user->socials)->toHaveCount(0);

    $socialiteUser = Mockery::mock(SocialiteUser::class);

    $socialiteUser
        ->shouldReceive('getId')->andReturn('test-id')
        ->shouldReceive('getName')->andReturn('First Last')
        ->shouldReceive('getNickname')->andReturn('Foobar')
        ->shouldReceive('getEmail')->andReturn('first@example.com')
        ->shouldReceive('getAvatar')->andReturn('https://example.com/avatar.jpg');

    Socialite::shouldReceive('driver->user')
        ->andReturn($socialiteUser);

    $this->get('/login/github/callback')
        ->assertRedirect('/dashboard');

    expect($user->refresh())
        ->name->toBe('Sean')
        ->profile_photo_path->toBe('https://example.com/avatar.jpg');

    expect($user->socials)->toHaveCount(1);

    expect($user->allTeams())->toHaveCount(1);

    expect($user->socials->first())
        ->provider->toBe('github')
        ->provider_user_id->toBe('test-id')
        ->email->toBe('first@example.com')
        ->name->toBe('First Last')
        ->avatar->toBe('https://example.com/avatar.jpg');

    $this->assertAuthenticated();
});

test('email users can login with existing matching social accounts', function(){

    $user = User::factory()->withPersonalTeam()->create([
        'name' => 'Sean',
        'email' => 'first@example.com',
        'profile_photo_path' => 'https://example.com/avatar.jpg'
    ]);

    expect($user->socials)->toHaveCount(0);

    expect($user->allTeams())->toHaveCount(1);

    $user->socials()->create([
        'provider' => 'github',
        'provider_user_id' => 'test-id',
        'email' => 'first@example.com',
        'name' => 'Sean 2',
        'avatar' => 'https://avatars.githubusercontent.com/u/1',
    ]);

    expect($user->socials()->get())->toHaveCount(1);

    $socialiteUser = Mockery::mock(SocialiteUser::class);

    $socialiteUser
        ->shouldReceive('getId')->andReturn('test-id')
        ->shouldReceive('getName')->andReturn('First Last')
        ->shouldReceive('getNickname')->andReturn('Foobar')
        ->shouldReceive('getEmail')->andReturn('first@example.com')
        ->shouldReceive('getAvatar')->andReturn('https://avatars.githubusercontent.com/u/1');

    Socialite::shouldReceive('driver->user')
        ->andReturn($socialiteUser);

    $this->get('/login/github/callback')
        ->assertRedirect('/dashboard');

    expect($user->refresh())
        ->name->toBe('Sean')
        ->profile_photo_path->toBe('https://example.com/avatar.jpg');

    expect($user->socials)->toHaveCount(1);

    expect($user->allTeams())->toHaveCount(1);

    expect($user->socials->first())
        ->provider->toBe('github')
        ->provider_user_id->toBe('test-id')
        ->email->toBe('first@example.com')
        ->name->toBe('Sean 2')
        ->avatar->toBe('https://avatars.githubusercontent.com/u/1');

    $this->assertAuthenticated();
});
```

> :note: Something to consider...
> You might want to change the `login.social` route to a POST request when initiating the login flow.