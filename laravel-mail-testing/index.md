---
title: Laravel mail testing
slug: laravel-mail-testing
description: A snippet to show how to assert against mailable properties in feature tests
excerpt:
date: 2022-08-16
tags: [laravel, snippet, mail, config, testing, phpunit]
sources: []
---

# Laravel mail testing

The test below will:
- fake emails to prevent sending
- submit a form saves the contact to the database and generates an email
- retrieves the contact model
- checks the mailable is queued
- checks the contact model is available to the mailable
- checks the mailable properties match the expected delivery instructions
  - delivery address (site owner's email)
  - reply to address (the customer's email)
  - subject

Notice how we `build` the mailable inside the Mail assertion callback so we can verify the properties are correct.

We then run individual assertions inside this callback so we can easily identify the reason for any failures.

```php
/**
 * @test
 */
public function it_sends_a_contact_form_message_to_the_site_owner_email_address()
{
    Mail::fake();

    $this->post('contact-us', [
        'name' => 'your name',
        'email' => 'your@email.com',
        'message' => 'This is a contact form message'
        ]);

    $contact = Contact::where('email', 'your@email.com')->first();

    Mail::assertQueued(ContactReceivedEmail::class, function ($mail) use ($contact) {
        // generate the mailable properties to assert against
        $mail->build();

        // verify the contact mailable is accessible to the mailable
        $this->assertEquals($contact->id, $mail->contact->id);
        $this->assertEquals($contact->subject, $mail->subject);

        // using mail.from as the delivery address for this admin email
        $this->assertTrue($mail->hasTo(config('mail.from.address')));

        // using the customer's email as the reply to address so the owner can reply easily
        $this->assertTrue($mail->hasReplyTo($contact->email, $contact->name));
        return true;
    });
}
```