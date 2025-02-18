---
title: "Valor insensitivo en la QueryString"
date: 2025-02-17
draft: false
slug: insensitive-query-string-value
tags: ["TypeScript", "QueryString", "client-side"]
categories: ["Desarrollo"]
---
Recientemente tuve una situación donde necesitaba en el cliente el valor de un parámetro específico de la query string del pedido y necesitaba estar completamente seguro de que podría encontrar el parámetro sin importar si estuviese en mayúsculas, minúsculas o cualquier combinación de estas. Con ese propósito cree la siguiente función en TypeScript:

``` TypeScript
private getInsensitiveQueryStringValue(queryParams: Params, paramKey: string): string {
  const keys = Object.keys(queryParams);
  const key = Array.from(keys).find(item => item.toLowerCase() == paramKey.toLowerCase());
  return key ? queryParams[key] : null;
}
```
