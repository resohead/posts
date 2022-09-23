---
title: Laravel mail config
slug: laravel-mail-config
description: A quick guide on configuring mail in Laravel.
date: 2022-08-16
tags: [laravel, snippet, mail, config]
sources: []
---

# Mail Config

## Default from address
You can set a global `from` address in `config/mail.php` instead of setting it each time on your mailables.

config/mail.php
```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

## Default to address
You might want to store email addresses of site owner in a config file so you can deliver admin emails to them through mailables.

> :warning:
> Do not create a `to` key in the `config/mail.php` file! This is reserved as a universal delivery address for development/testing. Doing this will override all delivery addresses on the mailables.

In the example below we will keep our 'internal' email addresses in a `site` key.

config/mail.php
```php
return [
    // ...

    // default sender details: config('mail.from')
    'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
        'name' => env('MAIL_FROM_NAME', 'Example'),
    ],

    // univeral delivery address: config('mail.to')
    // warning - this is not a default but a forced override!
    'to' => [
        'address' => 'override@example.com',
        'name' => 'Example'
    ],

    // custom key: config('mail.admin')
    // doesn't do anything by default
    'site' => [
        'owner' => [
            'name' => env('MAIL_OWNER_NAME', 'Site Owner'),
            'address' => env('MAIL_OWNER_ADDRESS', 'owner@example.com'
        ],
        'admin' => [
            'name' => env('MAIL_ADMIN_NAME', 'Admin'),
            'address' => env('MAIL_ADMIN_ADDRESS', 'admin@agency.com')
        ],
    ],

];
```

> Note: the email address array keys are different for senders and recipients. Use associative array for every recpient with 'email' and 'name' keys.

In your mailable you can now use the site owner email address.

```php
namespace App\Mail;

class ContactFormReceivedEmail extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    public function __construct(public Contact $contact)
    {
        //
    }

    public function build()
    {
        return $this->subject("New contact form submission")
            ->to(config('mail.site.owner'))
            ->replyTo($this->contact->email, $this->contact->name)
            ->markdown('emails.contact.received')
            ->text('emails.contact.received_text');
    }
}
```

Alternatively, you could use the `config(mail.from)` address to set the recipient as the site owner. This saves having to create a custom key in the config file but prevents you from directing message to a different address than the sender address.

## Sending emails to a list

A mail instance relates to a single email. This means the `to` method will append each address to the recipients list for that email - it will not send a new email for each address!

To send an individual email to each person you need to loop over the recipients and re-create the mailable instance for each recipient.

```php
$recipients = ['foo@example.com', 'bar@example.com'];

// send one email to all addresses
Mail::to($recipients)->send(new OrderShipped($order));

// send an individual email to each address
foreach ($recipients as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

## Gmail settings

See how to use [gmail with laravel](./laravel-gmail-setup).