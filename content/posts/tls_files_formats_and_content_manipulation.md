---
title: "TLS: Files formats and content manipulation"
description: "TLS: Files formats and content manipulation #tls #ssl"
date: 2018-06-20T13:41:46+02:00
draft: false
categories: ['security', 'TLS', 'PKI']
author: Yanis Guenane
---

The Transport Layer Security (TLS) protocol, successor of the Secure Socket Layer (SSL) after its version 3 (SSLv3) is a protocol that provides communications security over a computer network. In today's world this is the technology that encrypts your communications to web servers, mail servers, instant messaging servers,... .

Setting TLS up correctly is not a hard task but requires quite some amount of know-hows.
First timers (or not) in contact with TLS can quickly be overwhelmed by the perception of the multitude of file formats.
To simply enable TLS on a HTTP server, one might come across `.key`, `.csr`,`.crt` and`.pem` files.
Eventually based on the platform one can even come across a `.cer` file.

Don't be afraid, don't pull your hair, this article is here to demistify the files formats involved in dealing with TLS from a practical point of view and explain how it all works together.

<!--more-->

## File formats

In the introduction of this article, we've introduced no less than five different extensions. But be relieved, when dealing with TLS one will be dealing mostly with a single file format PEM - and to a much less frequent extent eventually DER. Let's do a tour of the different file formats, why they exist and explain how they all do fit together.

### Abstract Syntax One (ASN.1)

> Abstract Syntax Notation One (ASN.1) is an interface description language for defining data structures that can be serialized and deserialized in a standard, cross-platform way. It is broadly used in telecommunications and computer networking, and especially in cryptography.
>
> -- https://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One

At its core, every item involved with TLS has an ASN.1 representation: Keys, Certificate Signing Requests, Certificates,... .
As its name says it, it is an abstract syntax oriented mainly to developers aiming to realize a protocol implementation in the platform of their choice.
From a practical TLS deployment point of view, one should never have to deal directly with the ASN.1 representation of the items involed, but with its **DER** encoded version.


Yet, for curiosity, it could be interesting to know the ASN.1 representation of the various items.
Some ASN.1 syntax can be easily found on RFCs other will need more digging over the internet to find it.
For a X.509v3 certificate, the ASN.1 syntax is the following based on [RFC5280](https://tools.ietf.org/html/rfc5280)

```
Certificate  ::=  SEQUENCE  {
        tbsCertificate       TBSCertificate,
        signatureAlgorithm   AlgorithmIdentifier,
        signatureValue       BIT STRING
}

TBSCertificate  ::=  SEQUENCE  {
        version         [0]  EXPLICIT Version DEFAULT v1,
        serialNumber         CertificateSerialNumber,
        signature            AlgorithmIdentifier,
        issuer               Name,
        validity             Validity,
        subject              Name,
        subjectPublicKeyInfo SubjectPublicKeyInfo,
        issuerUniqueID  [1]  IMPLICIT UniqueIdentifier OPTIONAL,
                             -- If present, version MUST be v2 or v3
        subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL,
                             -- If present, version MUST be v2 or v3
        extensions      [3]  EXPLICIT Extensions OPTIONAL
                             -- If present, version MUST be v3
}

Version  ::=  INTEGER  {  v1(0), v2(1), v3(2)   }

CertificateSerialNumber  ::=  INTEGER

Validity ::= SEQUENCE {
        notBefore      Time,
        notAfter       Time
}

Time ::= CHOICE {
        utcTime        UTCTime,
        generalTime    GeneralizedTime
}

UniqueIdentifier  ::=  BIT STRING

SubjectPublicKeyInfo  ::=  SEQUENCE  {
        algorithm            AlgorithmIdentifier,
        subjectPublicKey     BIT STRING
}

Extensions  ::=  SEQUENCE SIZE (1..MAX) OF Extension

Extension  ::=  SEQUENCE  {
        extnID      OBJECT IDENTIFIER,
        critical    BOOLEAN DEFAULT FALSE,
        extnValue   OCTET STRING
                    -- contains the DER encoding of an ASN.1 value
                    -- corresponding to the extension type identified
                    -- by extnID
}
```

### Distinguished Encoding Rules (DER)

> DER is a subset of BER providing for exactly one way to encode an ASN.1 value. DER is intended for situations when a unique encoding is needed, such as in cryptography, and ensures that a data structure that needs to be digitally signed produces a unique serialized representation.
>
> -- https://en.wikipedia.org/wiki/X.690#DER_encoding

Put in simple words: DER is a way to encode ASN.1 syntax in binary format.
From a practical TLS deployment, one might eventually come across this file format, but more often than not will have to deal with **PEM** files.


### Privacy-Enhanced eMail (PEM)

As stated earlier DER encoding produces a binary output which can be cumbersome when transport is needed. **Enters PEM**. The PEM format is composed of the **DER** binary data encoded in base64 and surrounded by `-----BEGIN <LABEL>-----` and `-----END <LABEL>-----`

Here is the PEM represenation of the Certificate used for the blog.

```
-----BEGIN CERTIFICATE-----
MIIHJjCCBg6gAwIBAgISAwxVTPKetdiB25JeEmOw0hLLMA0GCSqGSIb3DQEBCwUA
MEoxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MSMwIQYDVQQD
ExpMZXQncyBFbmNyeXB0IEF1dGhvcml0eSBYMzAeFw0xODA2MTkwMTAwMThaFw0x
ODA5MTcwMTAwMThaMB8xHTAbBgNVBAMTFGJsb2cueWFuaXNndWVuYW5lLmZyMIIC
IjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAlrwYquspcFcvvSKuCBfvj/nD
7ZkG2RvEVf4rQ+D3iyLho0unpNNoUkf8rH++BvON9g8hnTjGmTJ5m6T8Zu/TALPA
Rm64dVzlfKYorpSpCnGSYPKpFmsnNlnvOhJeUqpZURbQ/si//SFoAqyUUGzlFW9Q
2kEC/wjltk7itpX9fBPM7VVfmHTst0az6zv59NwimqoPSM0QELVj3KO8ZKYQCmBM
x05p+atoVpI9k5JUrhG6My7vrurxYjIuWHwzHroZ1grEXvaryrm3UpRceRzgYc8A
chVzkDKSIYKdTt6aHg2VzGyEW/ANkZMlT18Fe88Y4G3h58oLeW7Z+QM/3gaEsGBP
mId8RsrVco9VVaM4Qs5qSy4pcq+Ok9PcwoeaQ3N2nipZ6CRqTh0F61KfkmSiWLsC
IYzPiu3wskIktFk+kle2oUjcJmSq5Wt42Q+rHruSgFPzbqFSxnOaJeFuFy5dUDUJ
wKMQHP1OT4ace4+ngScRQpDxcyjYHUVVS2tbR3ahLFIQI6Ac1qDHa/R43d0KJjfK
5QfWrdLLGIBrqACDOLdaHtoFY+1J+3kHguGhYJHq6YTEFpt1XKfl3YusEEo3lw0J
ai+36ZRr2RtWjTOgVarBKOef3F4pn7q4VgRIKkqKAtueFjLLyXLFmT+bi5Oo3eSL
OY8vkrDq792Vq2FQQk0CAwEAAaOCAy8wggMrMA4GA1UdDwEB/wQEAwIFoDAdBgNV
HSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwDAYDVR0TAQH/BAIwADAdBgNVHQ4E
FgQUjBeZ/qYLH22qwQ76w5BjMNIRQmswHwYDVR0jBBgwFoAUqEpqYwR93brm0Tm3
pkVl7/Oo7KEwbwYIKwYBBQUHAQEEYzBhMC4GCCsGAQUFBzABhiJodHRwOi8vb2Nz
cC5pbnQteDMubGV0c2VuY3J5cHQub3JnMC8GCCsGAQUFBzAChiNodHRwOi8vY2Vy
dC5pbnQteDMubGV0c2VuY3J5cHQub3JnLzAfBgNVHREEGDAWghRibG9nLnlhbmlz
Z3VlbmFuZS5mcjARBggrBgEFBQcBGAQFMAMCAQUwgf4GA1UdIASB9jCB8zAIBgZn
gQwBAgEwgeYGCysGAQQBgt8TAQEBMIHWMCYGCCsGAQUFBwIBFhpodHRwOi8vY3Bz
LmxldHNlbmNyeXB0Lm9yZzCBqwYIKwYBBQUHAgIwgZ4MgZtUaGlzIENlcnRpZmlj
YXRlIG1heSBvbmx5IGJlIHJlbGllZCB1cG9uIGJ5IFJlbHlpbmcgUGFydGllcyBh
bmQgb25seSBpbiBhY2NvcmRhbmNlIHdpdGggdGhlIENlcnRpZmljYXRlIFBvbGlj
eSBmb3VuZCBhdCBodHRwczovL2xldHNlbmNyeXB0Lm9yZy9yZXBvc2l0b3J5LzCC
AQQGCisGAQQB1nkCBAIEgfUEgfIA8AB2ANt0r+7LKeyx/so+cW0s5bmquzb3hHGD
x12dTze2H79kAAABZBXFcgwAAAQDAEcwRQIgStrf++1YiaCN1N9vW5oAZ889LZwo
rFns5CB2fRvPEqACIQDYJTqY71MpQbk+OXAM0gu6hvyeXSGT5dxlJQQ0k0rUEwB2
ACk8UZZUyDlluqpQ/FgH1Ldvv1h6KXLcpMMM9OVFR/R4AAABZBXFchUAAAQDAEcw
RQIhAP+S/XB3h8AHh2Bf+gWWQK8NKvYSdR4SU6SN++9HnCqvAiAElhleDhqYuTcr
54VOVJq1nOoOXRdiXwRWgNsRlWTgITANBgkqhkiG9w0BAQsFAAOCAQEARBljcayl
p0cmd38IIrZhvwiCJWW2BzfljZcPj8QzB4QrB6T3N2364agrRMevDXDntkp94RQZ
OG0C0AVDZSnblAG+5ppxcgIByJ3GA9UzpVcR+CDNthaecn5BCLTZq9z/PYZISX+h
OjP13My54WAgocCd4Mt5uneu6jl5AqjWEQ1/rUKDuj9s/HpSK43OfA8o/OvaczaU
Z/PZ3eueXKsa2h/xiNlUPY6opb+WVKx0JuMZmXlaezGkLe/McuOrryHtahprhIjz
cqdh2imXu4RRQJm38j/ReyxI8n/YzuL2Lwr+nPS7NCzIT24ySxZjx93VASoqZRD/
HXPF9Kju6YwMqQ==
-----END CERTIFICATE-----
```

Here is the PEM representation of the Certificate Signing Request (CSR) used for the blog.

```
-----BEGIN CERTIFICATE REQUEST-----
MIIEyzCCArMCAQMwQTEdMBsGA1UEAwwUYmxvZy55YW5pc2d1ZW5hbmUuZnIxIDAe
BgkqhkiG9w0BCQEWEXlhbmlzQGd1ZW5hbmUub3JnMIICIjANBgkqhkiG9w0BAQEF
AAOCAg8AMIICCgKCAgEAlrwYquspcFcvvSKuCBfvj/nD7ZkG2RvEVf4rQ+D3iyLh
o0unpNNoUkf8rH++BvON9g8hnTjGmTJ5m6T8Zu/TALPARm64dVzlfKYorpSpCnGS
YPKpFmsnNlnvOhJeUqpZURbQ/si//SFoAqyUUGzlFW9Q2kEC/wjltk7itpX9fBPM
7VVfmHTst0az6zv59NwimqoPSM0QELVj3KO8ZKYQCmBMx05p+atoVpI9k5JUrhG6
My7vrurxYjIuWHwzHroZ1grEXvaryrm3UpRceRzgYc8AchVzkDKSIYKdTt6aHg2V
zGyEW/ANkZMlT18Fe88Y4G3h58oLeW7Z+QM/3gaEsGBPmId8RsrVco9VVaM4Qs5q
Sy4pcq+Ok9PcwoeaQ3N2nipZ6CRqTh0F61KfkmSiWLsCIYzPiu3wskIktFk+kle2
oUjcJmSq5Wt42Q+rHruSgFPzbqFSxnOaJeFuFy5dUDUJwKMQHP1OT4ace4+ngScR
QpDxcyjYHUVVS2tbR3ahLFIQI6Ac1qDHa/R43d0KJjfK5QfWrdLLGIBrqACDOLda
HtoFY+1J+3kHguGhYJHq6YTEFpt1XKfl3YusEEo3lw0Jai+36ZRr2RtWjTOgVarB
KOef3F4pn7q4VgRIKkqKAtueFjLLyXLFmT+bi5Oo3eSLOY8vkrDq792Vq2FQQk0C
AwEAAaBFMEMGCSqGSIb3DQEJDjE2MDQwHwYDVR0RBBgwFoIUYmxvZy55YW5pc2d1
ZW5hbmUuZnIwEQYIKwYBBQUHARgEBTADAgEFMA0GCSqGSIb3DQEBCwUAA4ICAQCV
GkntVdVXLUOJiUXqnwAXp462j+wG2lFDq3Jzglzw4qgDIdZEGukRXQOI8YMJZEO5
RA3cDNOJl4sCzH1cuPcpCtnrw9US0rMsZYwfa7pVly8aXxL+0VQGme3kyvuKI5At
Qw9tER6A+pxNQPKG5AzVll7N7OX3fgJ1EgIObLICkdOVRg/1yN3BiEHubNHDXz1l
vl5WEpv+Db/oLDSNs1lSBKc+3DqogNkgpdrEwZCjvIt6qU7yz1DEiYcbQ3KMrVVI
KjX9auEaoq8Lb+mrMvk+f0znMribHTXqJxt2C3L9x+UXF1iiSc0EeA9wJwFGL6b5
yMjdZbhmsvnyCLMEN7atANlOR8iUqUqpYhS33Q1FTAFUx1T9MirN5ntBKS+H+Phy
IeODmqcvPo9I1nUiWa5pElLS6e6h5xdUPd6eKq6z6M5BEZDrk2DyX4YY13mSWeVr
IXIP1wuZwRqzoiJ0rMjPw13PEfKCnuu6SR3TQH/pp/HxNlGUC9apZgwUS2J5FSn9
kgcPImbSaGSXrMbyX6Ex5SyyHVFP9HgI13rOLuQ8Lbbc9bOf3PzvdEztD52Fchtf
YsxloZZl32kMUm3uNbWVDzmhbLUHI5dWW1Btfhq7NzWUHrSP3PWfJZuMVzQe5xWo
TTh529xbJRV5pTFIPwexH1WfafK6fbJt4e/FE3Yv7w==
-----END CERTIFICATE REQUEST-----
```

### Summary

Now go and open all your `.csr`, `.crt`, `.pem`, `.key` and `.cer` files, They should all be in PEM format. From a practical - TLS deployment, PEM format is the format one will encouter the most and hence will have to deal with the most.

| Role        | Extensions             |
| ----------- |------------------------|
| Key         | `.key`                 |
| CSR         | `.csr`                 |
| Certificate | `.crt`, `.cer`, `.pem` |

**Note**: A PEM file can contains one or many items. A Certificate chain can have many certificates in PEM format concatenated in a single file. The same goes for the trust-store.

## Content Manipulation

Now, we've seen that most files that need to be dealt with are in a PEM encoded format. Well... good to know, but that doesn't give me content.

Those files can easily be transformed in human readable text using the `openssl` command line.

### Private key

```
#> openssl rsa -in <MYPRIVATEKEY> -text -noout
```

### Public key

```
#> openssl rsa -pubin <MYPUBLICKEY> -text -noout
```

### Certificate Signing Request

```
#> openssl req -in <MYCSR> -text -noout
```

### Certificate

```
#> openssl x509 -in <MYCERTIFICATE> -text -noout
```

**Note**: If by any chance you end up having to deal with DER files, simply add `-inform DER` to the above commands.
```
#> openssl rsa -in <MYPRIVATEKEY> -inform DER -text -noout
```

## Conclusion

Do not get overwhelmed. It really is simple. It mainly boils down to deal with PEM format using `openssl` command line and the proper arguments.
Files can have the extension they want, it is the format that defines what they are.
Even if not necessarily practical on every day use, I hope this article helped understand the relation between ASN.1 -> DER -> PEM.
