---
title: Axios intercept
slug: axios-intercept
description: A snippet to intercept axios to view requet details.
date: 2022-08-15
tags: [javascript, axios, snippet]
sources: []
---

# Axios Intercept

Add the following snippet before an Axios call to view the details before it sends a request:

```js
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    console.log(config)
    return config;
    }, function (error) {
    // Do something with request error
    return Promise.reject(error);
});
```