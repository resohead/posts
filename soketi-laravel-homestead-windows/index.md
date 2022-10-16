---
title: Using Soketi with Laravel Homestead on Windows
slug: soketi-laravel-homestead-windows
description: Broadcast events over websockets using Soketi with Laravel Homestead on Windows.
date: 2022-08-26
tags: [websockets, homestead, windows, vite, pusher, echo]
sources:
    - https://laravel.com/docs/broadcasting
    - https://docs.soketi.app/getting-started/installation/cli-installation#installing-with-npm
---

# Using Soketi with Laravel Homestead on Windows

## Introduction
This is a quick guide to show you how to configure Laravel and soketi to allow broadcasting events via websockets. Soketi is a "simple, fast, and resilient open-source WebSockets server".

- implements the `Pusher Protocol v7`
- any Pusher-maintained or compatible client can connect to it
- use private channels, presence channels, and client event

You just need to point the Pusher compatible client to the soketi server address.

> :warning: Before you start...
> This article is strictly and reference guide and will not go into detail about websockets, how to set up Homestead or how to create a Laravel application

### Prerequisites
- Homestead provisioned and running
- Laravel application responding to requests
- npm installed on your local machine

### Summary

1. Install Soketi using Yarn [in Homestead]
2. Install Push PHP Server using Composer [in Homestead]
3. Configure your `.env`, `config\broadcasting.php` and JavaScript files
4. Create a broadcastable event [via Laravel Artisan]
5. Start socketi, queues and build assets [via CLI]
6. Verify socketi connection
7. Visit your application in the browser and trigger the event

## Getting started

### Install Soketi

Homestead has problems with NPM so I always use NPM in a local window when installing packages or running scripts.

In this case we need to run this command inside our Homestead VM so we will use `yarn` instead:
```
yarn install -g @soketi/soketi
```

### Pusher PHP Server

Soketi support using [Pusher channels](https://pusher.com/channels) such as private, prescence etc. We want to use that so let's install the Pusher PHP SDK:
```
composer require pusher/pusher-php-server
```

### Laravel Echo and Pusher JS

In a local project terminal install Laravel Echo and Pusher packages:
```
npm install --save-dev laravel-echo pusher-js
```

## Configuration

First update your project's `.env` to ensure we are using Pusher, redis and our soketi connection details. Remember, we need to point the Soketi (i.e. Pusher) host to our Homestead IP, i.e. 192.168.10.10 instead of 127.0.0.1.

Note - Soketi uses the following config values by default:
- Pusher App Id: `app-id`
- Pusher App Key: `app-key`
- Pusher App Secret: `app-secret`
- Port: 6001


### Environment
```.env
# ...

BROADCAST_DRIVER=pusher
QUEUE_CONNECTION=redis

# ...

PUSHER_APP_ID=app-id
PUSHER_APP_KEY=app-key
PUSHER_APP_SECRET=app-secret
PUSHER_HOST=192.168.10.10 # not 127.0.0.1
PUSHER_PORT=6001
PUSHER_PORT=443
PUSHER_SCHEME=http
PUSHER_APP_CLUSTER=mt1


MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_HOST="${PUSHER_HOST}"
MIX_PUSHER_PORT="${PUSHER_PORT}"

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

> :tip: Did you know?
> You can change the socketi port by running `SOKETI_PORT=9001 soketi start` on whatever port you want.

### Broadcast driver

There shouldn't be too much to change from a standard Laravel install on 9.x:

```php file="config\broadcasting.php"
return [
    'default' => env('BROADCAST_DRIVER', 'null'),

    'connections' => [

        'pusher' => [
            'driver' => 'pusher',
            'key' => env('PUSHER_APP_KEY'),
            'secret' => env('PUSHER_APP_SECRET'),
            'app_id' => env('PUSHER_APP_ID'),
            'options' => [
                'host' => env('PUSHER_HOST', 'api-'.env('PUSHER_APP_CLUSTER', 'mt1').'.pusher.com') ?: 'api-'.env('PUSHER_APP_CLUSTER', 'mt1').'.pusher.com',
                'port' => env('PUSHER_PORT', 443),
                'scheme' => env('PUSHER_SCHEME', 'https'),
                'encrypted' => true,
                'useTLS' => env('PUSHER_SCHEME', 'https') === 'https',
            ],
            'client_options' => [
                // Guzzle client options: https://docs.guzzlephp.org/en/stable/request-options.html
            ],
        ],

        // ...
    ]
```

### JavaScript

We also need to provide the connection details to Laravel Echo to enable use to start listening to events:

```js file="resource\js\bootstrap.js"
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    wsHost: import.meta.env.VITE_PUSHER_HOST ?? `ws-${import.meta.env.VITE_PUSHER_APP_CLUSTER}.pusher.com`,
    wsPort: import.meta.env.VITE_PUSHER_PORT ?? 80,
    wssPort: import.meta.env.VITE_PUSHER_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_PUSHER_SCHEME ?? 'https') === 'https',
    encrypted: true,
    disableStats: true,
    enabledTransports: ['ws', 'wss'],
});

// example listener on public channel
window.Echo
    .channel('room.1')
    .listen('UserJoinedRoom', (e) => {
        console.log(e);
    });
```

Notice we have also set a listener on a public channel `room.1` that will respond to `UserJoinedRoom` events. See 'listening to events with Vue' for more information about incorporating Echo into your Vue.

Next we need to create this backend event in Laravel and trigger it.

## Broadcasting

### Create an event
```
php artisan make:event UserJoinedRoom
```

In this example we will create a public channel with a dynamic 'room' id and also pass some extra data (user model) through the constructor.

```php file="App\Events\UserJoinedRoom.php"
class UserJoinedRoom implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(public $room, public User $user) {}

    public function broadcastOn()
    {
        return new Channel("room.{$this->room}");
    }
}
```

### Start Soketi, queues and build assets

You will need a few terminal windows open (both locally and in Homestead).

1. [SSH]: `php artisan queue:listen`
2. [SSH]: `soketi start`
3. [Local]: `npm run dev`

### Verify Soketi connection

Your should receive `ok` after visiting `http://192.168.10.10:6001` in your browser.

### Receive an event

You can dispatch the event [we created earlier](#create-a-broadcastable-event) in any way you normally would but in order to quickly test everything is wired up correctly you can:
- create a route to dispatch the event from
- create a command to trigger the event from the command line
- trigger the event directly through tinker (or Tinkerwell!)

```php title="Laravel"
UserJoinedRoom::dispatch($room = 1, $user = User::find(1));
```

> In a production app it is likely this event would be fired as a result of an action a user has taken such as commenting on a post, booking tickets for an event etc.

At this point you will:
- have your redis queue listening
- have Soketi running
- your web app loaded in the browser
- just dispatched an event

Check the dev tools console and check you see an event with the expected payload: `{room: 1, user: {…}}`

### Debugging

- [verify the soketi connection](#verify-soketi-connection)
- ensure the js assets are loaded into your web page
- check any console connection errors show the correct port and scheme

> :tip:
> If it says 'wss' is being used despite your config then double check your are not accessing your homestead development site via HTTPS, otherwise Soketi will force 'wss'!

## Using Echo within Vue component

Let's remove our listener from our bootstrap file and plug it into our Vue components.

The theory remains unchanged - we are going to set a listener for an event `UserJoinedRoom` on a specific channel `room.1` but we will define this in our Vue component's mounted hook.

> Examples below are using Vue 3

We will listen for events in our dashboard and pass the results to a separate component to render the message. This example uses Laravel's Breeze template but this is the same as any other Vue component.

Remove the listener from our `resources\js\bootstrap.js` file.

```js diff file="resources\js\bootstrap.js"

 window.Echo = new Echo({
     broadcaster: 'pusher',
     key: import.meta.env.VITE_PUSHER_APP_KEY,
     wsHost: import.meta.env.VITE_PUSHER_HOST ?? `ws-${import.meta.env.VITE_PUSHER_APP_CLUSTER}.pusher.com`,
     wsPort: import.meta.env.VITE_PUSHER_PORT ?? 80,
     wssPort: import.meta.env.VITE_PUSHER_PORT ?? 443,
     forceTLS: (import.meta.env.VITE_PUSHER_SCHEME ?? 'https') === 'https',
     encrypted: true,
     disableStats: true,
     enabledTransports: ['ws', 'wss'],
 });

- window.Echo
-    .channel('room.1')
-    .listen('UserJoinedRoom', (e) => {
-        console.log(e);
-    });
```


```js diff file="resources\js\Pages\Dashboard.vue"
 <script setup>
     import BreezeAuthenticatedLayout from '@/Layouts/Authenticated.vue'
     import { Head } from '@inertiajs/inertia-vue3'
     import UserJoined from '@/Components/UserJoined.vue'
     import { onMounted, reactive } from 'vue'

     let messages = reactive([])

+    onMounted(() => {
+        window.Echo
+            .channel(`room.1`)
+            .listen('UserJoinedRoom', (e) => {
+                console.log(e)
+                messages.push(e)
+            })
+    })
 </script>

 <template>
     <Head title="Dashboard" />

     <BreezeAuthenticatedLayout>
         <template #header>
             <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                 Dashboard
             </h2>
         </template>

         <div class="py-12">
             <div class="max-w-7xl mx-auto sm:px-6 lg:px-8 space-y-2">
                 <UserJoined
                     v-for="{user, room} in messages"
                     :key="user.id"
                     :user="user"
                     :room="room"
                 />
             </div>
         </div>
     </BreezeAuthenticatedLayout>
 </template>
```

```js diff file="resources\js\Components\UserJoined.vue"
! <script setup>
!     defineProps({
!         user: {
!             type: Object,
!             default: {}
!         },
!         room: {
!             type: String,
!             default: '',
!         },
!     });
! </script>

 <template>
     <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
         <div class="p-6 bg-white border-b border-gray-200">
             {{ user.name }} joined room {{ room }}
         </div>
     </div>
 </template>
```

## What's next?

Keep an eye out for following blog posts where we will discuss private and prescence channels, production settings as well as alternative ways to use Vue components around websocket messages.