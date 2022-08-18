---
title: PHP 8 Match
slug:
excerpt: A couple of examples using PHP 8 match function
date: 2022-08-15
tags: [php, match, syntax, snippet]
sources:
    - https://ryangjchandler.co.uk/posts/all-about-match-expressions
---

## PHP 8 Match

I really liked this way of structing PHP 8 match conditional expressions from Ryan Chandler's blog post ["All about match expressions"](https://ryangjchandler.co.uk/posts/all-about-match-expressions). Thanks Ryan!

Remember expressions are evaluated in order and only evaluates the first passing expression's return value. This is unlike using array matching where all keys are evaluated and values interpretted.

With PHP 8 match function we can safely use functions as expression values.

Here's an example from Ryan's article using arrays for pattern matching / matrix conditionals.

```php
$options = [
    'monthly',
    'credit-card',
];

return match ($options) {
    ['monthly', 'direct-debit'] => $this->setupDirectDebit(),
    ['yearly', 'credit-card']   => $this->takeYearlyCardPayment(),
    ['monthly', 'credit-card']  => $this->setupCardSubscription(), // true
};
```

Of course you could also match on `true` and have the expressions contain independent conditions. But remember only the first true condition will be evaluated. Here's a silly example:

```php
return match (true) {
    'foo' === 'bar' => $this->foobar(),
    $something instanceof App\Models\User => $this->doSomething(),
    $other > 5  => 'greater than 5',
};
```

Do yourself a favour and read the article...and his entire blog while you're there!