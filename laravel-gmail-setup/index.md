---
title: Laravel gmail setup
slug: laravel-gmail-setup
description: A quick tip to show how to configure gmail for laravel.
date: 2022-08-16
tags: [laravel, snippet, mail, config]
sources: []
---

# Laravel Gmail Config

1. Enable 2-step verification in Gmail (required for app-passwords)
2. Generate a new `App Password`
   1. App = Mail
   2. Device = Other (custom name)
3. Copy 16 character password into following .env variables

```
MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=465
MAIL_USERNAME=youraccount@gmail.com
MAIL_PASSWORD=yourapppassword
MAIL_ENCRYPTION=ssl
```