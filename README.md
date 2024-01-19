# ASN.1 and You!

This repository contains OpenSSL configuration files
defining X.509 private and public keys, certificates, and
a few other goodies. OpenSSL's configuration file format
essentially amounts to a mechanism by which to write ASN.1
formatted data in plain text. If you're wondering
why anybody would do such a thing, well, back in the day,
programming was hard. If you're **not** wondering, then
you and and I should get on our ham radios and complain
about our glaucoma to each other.

This repository is also known as:

* Fun with DER and ASN.1.
* The case for JSON.
* `00AE1122BE7F0000:error:1608010C:STORE routines:ossl_store_handle_load_result:unsupported:crypto/store/store_result.c:151: Unable to load certificate`
* Remember when programming was hard?
* Seriously, XML wasn't all that bad after all.

In all seriousness, this repo was born out of somewhat
academic research into [RFC3161](https://datatracker.ietf.org/doc/html/rfc3161), *Internet X.509 Public Key Infrastructure Time-Stamp Protocol (TSP)*,
and an attempt to implement it with OpenSSL tooling. A link to that will be placed here
when it is release.
