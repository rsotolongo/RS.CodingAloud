---
title: "Generando certificados de firma"
date: 2021-02-01
draft: false
slug: generating-signature-certificates
tags: ["OpenSSL", "seguridad"]
categories: ["Desarrollo"]
---
[Código fuente de apoyo disponible en GitHub](https://gist.github.com/rsotolongo/80410cee05c26a7bcb17327d0c279b86)

A veces, los ensamblados (ejecutables y bibliotecas) deben estar firmados y no tenemos un certificado disponible con fines de prueba. En este post pretendo mostrar cómo generar un certificado para firmar ensamblados.

Primero, necesitamos instalar OpenSSL que podemos lograr siguiendo los pasos que se especifican en el siguiente enlace:

[Instalación de OpenSSL en Windows 10 y actualización de PATH](https://medium.com/swlh/installing-openssl-on-windows-10-and-updating-path-80992e26f6a1)

En segundo lugar, creemos un archivo de secuencia de comandos por lotes llamado: "generateCertificate.cmd" con contenido:

``` CMD
MD Certificate
CD Certificate

ECHO authorityKeyIdentifier=keyid,issuer>certificate.ext
ECHO basicConstraints=CA:FALSE>>certificate.ext
ECHO keyUsage=digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment>>certificate.ext
ECHO subjectAltName=@alt_names>>certificate.ext
ECHO.>>certificate.ext
ECHO [alt_names]>>certificate.ext
ECHO DNS.1=RS>>certificate.ext

openssl.exe genrsa -des3 -passout pass:"pass&1234" -out certificate.pass.key 2048
openssl.exe rsa -passin pass:"pass&1234" -in certificate.pass.key -out certificate.key
openssl.exe req -new -key certificate.key -out certificate.csr
openssl.exe x509 -req -sha256 -extfile certificate.ext -days 365 -in certificate.csr -signkey certificate.key -out certificate.crt
openssl.exe pkcs12 -export -out certificate.pfx -inkey certificate.key -in certificate.crt
```

Básicamente creamos una carpeta (llamada: "Certificate") y la convertimos en el directorio de trabajo. Después creamos un archivo escribiendo su contenido línea por línea. Finalmente, en varios pasos, usamos la herramienta: "openssl.exe" para generar el certificado. Observe las comillas dobles que rodean el valor de la contraseña debido a que contiene el carácter especial de conjunción (&). El código fuente de ese archivo se puede descargar desde [este GIST](https://gist.github.com/rsotolongo/80410cee05c26a7bcb17327d0c279b86).

Suponiendo que lo hemos copiado en la carpeta: "C:\Temp", veamos qué hace a través del análisis de la figura 1 que representa su ejecución.

![Ejecución de la secuencia de comandos](/generating-signature-certificates/01.png)

Figura 1: Ejecución de la secuencia de comandos.

Los pasos a prestar más atención se indican con flechas rojas y amarillas, mientras que hay otros pasos opcionales indicados con flechas azules. La primera flecha roja es el nombre común que elegimos para el certificado. Este debe ser el mismo valor que tenemos que ingresar una vez que la secuencia de comandos solicita un valor en la segunda flecha roja.

Las flechas amarillas son la contraseña elegida para el certificado. Si bien los dos primeros valores se especifican en el archivo por lotes: "generateCertificate.cmd", los dos últimos valores deben introducirse manualmente cuando se soliciten. La herramienta: "openssl.exe" oculta esos valores mientras se escriben, pero deben coincidir.

Una vez que todos los pasos terminan con éxito, podemos ver el resultado en la figura 2.

![Certificado resultante](/generating-signature-certificates/02.png)

Figura 2: Certificado resultante.

Ese es un certificado que podemos usar para firmar ensamblados (un proceso que explicaré en una publicación posterior).

[Código fuente de apoyo disponible en GitHub](https://gist.github.com/rsotolongo/80410cee05c26a7bcb17327d0c279b86)
