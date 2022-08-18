---
title: No scrollbar css
slug: no-scrollbar-css
excerpt: A couple of examples using PHP 8 match function
date: 2022-08-15
tags: [css, tailwind, snippet]
sources:
    - https://ryangjchandler.co.uk/posts/all-about-match-expressions
---

# No Scrollbar in CSS

The example below shows a simple Tailwind .css file but you can just extract the `no-scrollbar` style for any other css framework.

```css
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';

@layer utilities {
    .no-scrollbar {
        scrollbar-width: none;
    }
    .no-scrollbar::-webkit-scrollbar {
        width: 0px;
        background: transparent;
        /* display: none; */
    }
}
```

Usage

```html
<div class="overflow-y-auto no-scrollbar">
</div>
```