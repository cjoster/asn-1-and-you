# RSA keys in PKCS#8 Format

*(the password for every encrypted key in this repository is:* **testing***)*

If you haven't read up on the [PKCS#1](../) format, it might be wise to do so.
PKCS#8 is the default private key format for OpenSSL and has been so for a little
while (as of this writing). It is mostly just a wrapper around the PKCS#1 format
(for RSA keys, at least), but it has an advantage. PKCS#8 allows for the payload
in the wrapper to be encrypted, and has places for the other PEM headers, which we'll see
later.

In any case, if you look at the key in this directory:

```
openssl rsa -in rsa.key | openssl asn1parse
```

...which gives...

```
    0:d=0  hl=4 l= 340 cons: SEQUENCE          
    4:d=1  hl=2 l=   1 prim: INTEGER           :00
    7:d=1  hl=2 l=  13 cons: SEQUENCE          
    9:d=2  hl=2 l=   9 prim: OBJECT            :rsaEncryption
   20:d=2  hl=2 l=   0 prim: NULL              
   22:d=1  hl=4 l= 318 prim: OCTET STRING     [HEX DUMP]:3082013A020100024100E6409D0AA3B3B94FEA1547AD950335ABA3F1364468A779580B4FF6E72538B53C21482779E9AD5D06D95DFD1D4F219C2749B4A992766E97FF5FA4F9E32BE6FEBF0203010001024060DD9FF3A0E8F43605818C551F52695ADB2E9828F16A3B6769E2EB3954F46571A92CA4FDA4226F850D81A6FF57EF865F4BB0AD61EE9AFCF3E15C71073E53BAE1022100F8968B6EFF8AA66D2B2F4D5906AD0655C96A5CE0990D63C307ED3A85532AB1B1022100ED1E1D28A9B6F8D39768A15D94B97F1BCF320711BE35D65D48889A760F14E36F022031362C5847027DBBF2E6A45B517503620C43A02B5E6146349FE718C4B81825A102207448EC6BE0AF46E01DC4C63E2A8DBDF4596C636324312AEB9C82C19D5C5016290221009097D6C30246D00AD2342B29EE085BDD3BA68448A7E48575883B4F97480D5AD8
```

...You'll notice one thing. (Ok, maybe you won't but you will be able to afterwards. That first two letters (first byte) of the hex dump (0x30) just
happens to be (one of[^2]) the ASN.1 tags for `SEQUENCE`. You'll actually see that used a lot in X.509 certificates...embeded or recursive ASN.1.[^3]

[^2]: 0x10 is a sequence, but the next higher bit determines whether it's a 0=pimitive or 1=construct. So, in binary 00110000==0x30==(construct of)(sequence).

[^3]: Did I mention that ASN.1 is the best case argument for something like JSON?

If you grab that and run it through a reverse-hexedit (and then the appropriate commands, you'll get the PKCS#1 key back. (Unless it's encrypted.)

In any case, generate the private key with the following command:

```bash
openssl asn1parse -genconf rsa.cnf -noout -out - | openssl rsa -inform DER
```

## Public Key

The public key is similarly wrapped, which is in [rsa-pub.cnf](rsa-pub.cnf). The public key can
be produced with:

```bash
openssl asn1parse -genconf rsa-pub.cnf -noout -out - | openssl rsa -inform DER -pubin
```

## Encrypted Private Key

If you've ever looked at an encrypted private key, there's a header at the top that,
among other things, shows that a) it's encrypted, and b) the algorithm along with
the initialization vector and/or salt for that particular algorithm. Take a look
at the following example:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,C0CDBC9A6333A23F9429871FA1649CD9

xDxzso8lRYQFveMgawVLNaZE6qDrNX1YxyLoZmRowj5XDvlSspMv3WMV71/9IJx9
elBcFAMv2I301GGvo3tkum4Mcx6TLHPgpS5qLEV/QJGGJDJSiDmQcG4Nj/+6c/06
18F1Fvs/XbxxRkipJZJ/TkxFmNhMRf7MTbm0uJSZwTZEc7AWQK/geauRfmQIu1t+
2/RcifKULBUQWWo8LJu0EoCcCQ7meaqkr0ZbUVGiZnI3kjnIbsq9NB213JpVikqN
I1yBloKDYvHdEVjQKlgCpqnnpWiWO6lk0M6u9/axr7sSIp91Sh0y1maXv2rDXO+d
Fyx3o/lsCULwhjTDZQIpab9/YBGeNHTKetzKDrr+b6sADKCt4ElD5792kVkLGCZr
wZxbpWWK+uy+f9WdGUWA2tLI0ta/JTIzNucEda+f8Xg=
-----END RSA PRIVATE KEY-----
```

That "Proc-Type" and "DEK-Info" are actually part of the **P**rivacy
**E**nhanced **M**ail standard ([RFC1421](https://datatracker.ietf.org/doc/html/rfc1421.html)),
which is where the **PEM** format comes from.[^1]

[^1]: Apparently, base64-encode-some-binary-stuff-and-stick-a-header-and-footer-on-it-so-you-know-what-it-is-and-so-you-can-pick-it-out-of-surrounding-text
(**BESBSSAFOI**) doesn't roll off the tongue as well as **PEM** does.

Anyway, what's left of the key is simpley a binary blob encrypted
via AES. The thing is, there's a lot more to decent encryption,
particularly password-based-key-derivation functions, like salts,
IV's, etc, that just have to be assumed in PKCS#1. In PKCS#8, there
are places for those things.

Take a look at [rsa-encrypted.cnf](rsa-encrypted.cnf) and you can
see iterations, salts, IV's, and the encrypted blob. Full discussion
of those terms is outside the scope of this repository, but
can be found in [PKCS#8 specification, RFC5208](https://datatracker.ietf.org/doc/html/rfc5208).

Reassembling the private key can be done with the following command.
You'll need the password **testing** to do so.

```bash
openssl asn1parse -genconf rsa-encrypted.cnf -noout -out - | openssl rsa -inform DER
```
