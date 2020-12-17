---
title: "Generating signature certificates"
date: 2021-02-01
draft: false
slug: generating-signature-certificates
tags: ["OpenSSL", "security"]
categories: ["Development"]
---
[Source code of support available at GitHub](https://gist.github.com/rsotolongo/80410cee05c26a7bcb17327d0c279b86)

Sometimes assemblies (executables and libraries) are required to be sign and I don't have a certificate available for test purpose. In this post I pretend to show how to generate a certificate to sign assemblies.

First, we need to install OpenSSL that we can achive following the steps specified in the next link:

[Installing OpenSSL on Windows 10 and updating PATH](https://medium.com/swlh/installing-openssl-on-windows-10-and-updating-path-80992e26f6a1)

Second, let's create a batch script file called: "generateCertificate.cmd" with content:

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

Roughly we create a folder (called: "Certificate") and make it the working directory. After we create a file writing its content line by line. Finally, in several steps, we use tool: "openssl.exe" to generate the certificate. Notice the double quotes surrounding the password value due to it contains the special character ampersand (&). The source code for that file could be downloaded from [this GIST](https://gist.github.com/rsotolongo/80410cee05c26a7bcb17327d0c279b86).

Assuming that we have copied it into folder: "C:\Temp", let's see what it does through the analysis of figure 1 representing its execution.

![Script execution](/generating-signature-certificates/01.png)

Figure 1: Script execution.

The steps to pay more attention are indicated with red and yellow arrows while there are other optional steps indicated with blue arrows. The first red arrow is the common name that we choose for the certificate. This must to be the same value that we have to enter once the script asks for a value in the second red arrow.

Yellow arrows are the password chosen for the certificate. While the first two values are specified in batch file: "generateCertificate.cmd", the last two values have to be introduced manually when are requested. Tool: "openssl.exe" hides those values while are typed but they must to be match.

Once all steps end successfully we can see the result in figure 2.

![Resulting certificate](/generating-signature-certificates/02.png)

Figure 2: Resulting certificate.

That is a certificate we can use to sign assemblies (a process I will explain in a later post).

[Source code of support available at GitHub](https://gist.github.com/rsotolongo/80410cee05c26a7bcb17327d0c279b86)
