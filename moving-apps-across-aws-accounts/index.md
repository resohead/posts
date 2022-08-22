---
title: Moving apps across AWS accounts
slug: moving-apps-across-aws-accounts
description: A twitter tip from Claudio Dekker and Zubair Moshin about migrating AWS apps across accounts.
date: 2022-08-15
tags: [aws, migrations, twitter]
sources:
    - https://twitter.com/Zubairmohsin33/status/1434487849093681154
    - https://twitter.com/zubairmoshin33
    - https://twitter.com/claudiodekker
---

# Moving apps across AWS accounts

I took this tip from [Claudio Dekker's](https://twitter.com/claudiodekker) response to [Zubair Moshin's](https://twitter.com/zubairmoshin33) tweet about a [checklist for migrating AWS applications](https://twitter.com/Zubairmohsin33/status/1434487849093681154) across accounts. It looks like the tweet has since been deleted but thought it was worth sharing and might be useful for further research.

If you found this helpful (or even if you didn't) give Claudio and Zubair a follow 👍

1. Assign floating IP to old instance
2. Switch DNS to floating IP (i.e. the DNS and floating IP now points to the old instance)
3. Wait for propagation, prepare new environment
4. Put old environment in maintenance mode
5. Move over data
6. Test the app in the new environment
7. Switch floating IP to new instance IP
8. Fix Issues and remove old instance after a few days

> You shouldn't have to wait for propagation in step 7 because the new environment doesn't have maintenance mode and all data. The floating IP can also be instantly re-assigned as there's no DNS prop (IP stays the same)