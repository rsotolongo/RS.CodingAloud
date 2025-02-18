---
title: "Insensitive QueryString value"
date: 2025-02-17
draft: false
slug: insensitive-query-string-value
tags: ["TypeScript", "QueryString", "client-side"]
categories: ["Development"]
---
Recently I got into a situation where I need in the client side the value of a specific parameter in the request's query string and I need to be sure that I could find the parameter regardless its casing. For that purpose I crate the following function in TypeScript:

``` TypeScript
private getInsensitiveQueryStringValue(queryParams: Params, paramKey: string): string {
  const keys = Object.keys(queryParams);
  const key = Array.from(keys).find(item => item.toLowerCase() == paramKey.toLowerCase());
  return key ? queryParams[key] : null;
}
```
