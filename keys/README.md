# RSA keys

RSA keys come in two different formats these days.
The first is [PKCS#1](https://en.wikipedia.org/wiki/PKCS_1), which was the original [Public Key
Cryptography Standards (PKCS)](https://en.wikipedia.org/wiki/PKCS) private key format. [RSA](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29)
was all there was when public key cryptography was developed.

This document and repository covers both private and public RSA key formats in PKCS#1. For [DSA](dsa) keys, see the [dsa](dsa) directory.
Additionally, are are both [Prime256v1](ecc/prime256v1) and [Ed25519](ecc/ed25519) keys in the [ecc/prime256v1](ecc/prime256v1) and
[ecc/ed25519](ecc/ed25519) directories respectively. Lastly, the PKCS#8 versions of this RSA key (and the DSA, Prime256v1, and Ed25519 keys)
can be found in the [pkcs8](pkcs8) directory.

## Private Keys

### PKCS#1 private key format

The format for PKCS#1 can be found in [RFC3447]https://datatracker.ietf.org/doc/html/rfc3447). Specifically, the format
for private keys is in [Appendix 1.2](https://datatracker.ietf.org/doc/html/rfc3447#appendix-A.1.2),
reproduced here:

```
RSAPrivateKey ::= SEQUENCE {
    version           Version,
    modulus           INTEGER,  -- n
    publicExponent    INTEGER,  -- e
    privateExponent   INTEGER,  -- d
    prime1            INTEGER,  -- p
    prime2            INTEGER,  -- q
    exponent1         INTEGER,  -- d mod (p-1)
    exponent2         INTEGER,  -- d mod (q-1)
    coefficient       INTEGER,  -- (inverse of q) mod p
    otherPrimeInfos   OtherPrimeInfos OPTIONAL
}

Version ::= INTEGER { two-prime(0), multi(1) }
    (CONSTRAINED BY {
        -- version must be multi if otherPrimeInfos present --
    })

OtherPrimeInfos ::= SEQUENCE SIZE(1..MAX) OF OtherPrimeInfo

OtherPrimeInfo ::= SEQUENCE {
    prime             INTEGER,  -- ri
    exponent          INTEGER,  -- di
    coefficient       INTEGER   -- ti
}
```

For simplicity, we will ignore the multi-prime version and focus on
the (usual) two-prime version of the key. This repository will not discuss
how the values are obtained; for details about how to calculate
them, please refer to [RFC3447](https://datatracker.ietf.org/doc/html/rfc3447)[^1].

[^1]: If you enjoyed reading the [Magna Carta](https://www.nationalarchives.gov.uk/wp-content/uploads/2015/01/magna-carta-1215-salisbury-cathedral1.jpg), it should be right up your alley.

You can obtain the plain-text version of an RSA private key with the following command:

```bash
openssl genrsa 512 2>/dev/null | openssl rsa -text -noout
```

In fact, the PEM encoded and plain-text version of the key used in this example
can be found in [rsa.key](rsa.key).

You'll find two `.cnf` files in this directory that will generate (the same)
RSA private key. They are [rsa-hex.cnf](rsa-hex.cnf) and [rsa-dec.cnf](rsa-dec.cnf).
The only difference is that [rsa-hex.cnf](rsa-hex.cnf) has values in hexadecimal, while [rsa-dec.cnf](rsa-dec.cnf)
has them in base-10 decimal.

You can use the `asn1parse function of OpenSSL to dump out the structure. Unfortunatly, unlike
other OpenSSL commands, the `asn1parse` command isn't smart enough to skip over plain-text at the
beginning of the file, so you have to strip it out. Using the command:

```bash
openssl rsa -in rsa.key -traditional | openssl asn1parse -dump
```

Will yield the output:

```
writing RSA key
    0:d=0  hl=4 l= 314 cons: SEQUENCE          
    4:d=1  hl=2 l=   1 prim: INTEGER           :00
    7:d=1  hl=2 l=  65 prim: INTEGER           :E6409D0AA3B3B94FEA1547AD950335ABA3F1364468A779580B4FF6E72538B53C21482779E9AD5D06D95DFD1D4F219C2749B4A992766E97FF5FA4F9E32BE6FEBF
   74:d=1  hl=2 l=   3 prim: INTEGER           :010001
   79:d=1  hl=2 l=  64 prim: INTEGER           :60DD9FF3A0E8F43605818C551F52695ADB2E9828F16A3B6769E2EB3954F46571A92CA4FDA4226F850D81A6FF57EF865F4BB0AD61EE9AFCF3E15C71073E53BAE1
  145:d=1  hl=2 l=  33 prim: INTEGER           :F8968B6EFF8AA66D2B2F4D5906AD0655C96A5CE0990D63C307ED3A85532AB1B1
  180:d=1  hl=2 l=  33 prim: INTEGER           :ED1E1D28A9B6F8D39768A15D94B97F1BCF320711BE35D65D48889A760F14E36F
  215:d=1  hl=2 l=  32 prim: INTEGER           :31362C5847027DBBF2E6A45B517503620C43A02B5E6146349FE718C4B81825A1
  249:d=1  hl=2 l=  32 prim: INTEGER           :7448EC6BE0AF46E01DC4C63E2A8DBDF4596C636324312AEB9C82C19D5C501629
  283:d=1  hl=2 l=  33 prim: INTEGER           :9097D6C30246D00AD2342B29EE085BDD3BA68448A7E48575883B4F97480D5AD8
```

Similarly, we can use the `asn1parse` function of OpenSSL to put the key back together. Keep in mind
that doing so will output the binary ([DER-encoded](https://en.wikipedia.org/wiki/X.690#DER_encoding)) version of the key. To turn it
back into [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail), you just have to pass
`-inform DER` to any given `openssl` command.

```bash
openssl asn1parse -genconf rsa-hex.cnf -noout -out - | openssl rsa -inform DER -text -traditional
```

## Public Keys

Just like the private key found in [rsa.key](rsa.key), you'll find the public key--in both plain text
and PEM encoded format in [rsa.pub](rsa.pub). The ASN.1 syntax can be found in [Appendix 1.1](https://datatracker.ietf.org/doc/html/rfc3447#appendix-A.1.1)
of [RFC3447](https://datatracker.ietf.org/doc/html/rfc3447), reproduced here:

```
RSAPublicKey ::= SEQUENCE {
    modulus           INTEGER,  -- n
    publicExponent    INTEGER   -- e
}
```

Incidentally, I had a hard time getting any OpenSSL tooling to spit out this format for the public key.
To get the [public key](rsa.pub) in this directory, I had to manually build it, and then manually PEM encode
it, which can be done thusly:

```bash
(echo "-----BEGIN RSA PUBLIC KEY-----";
    openssl asn1parse -genconf rsa-pub.cnf -noout -out - | base64 -w 64;
    echo "-----END RSA PUBLIC KEY-----") > public-key.pem
```

Again, you can use the `asn1parse` command to parse the public key. The only problem
is that no OpenSSL command *won't* convert it form PKCS#1 to PKCS#8. So you'll have to skip
over the plain text at the top. Be weary the unnecessary `cat`, but it's here for
simplicity:

```bash
cat rsa.pub | awk '/^-----BEGIN/{p=1}p{print}' | openssl asn1parse 
```

...which will yield:

```
    0:d=0  hl=2 l=  72 cons: SEQUENCE          
    2:d=1  hl=2 l=  65 prim: INTEGER           :E6409D0AA3B3B94FEA1547AD950335ABA3F1364468A779580B4FF6E72538B53C21482779E9AD5D06D95DFD1D4F219C2749B4A992766E97FF5FA4F9E32BE6FEBF
   69:d=1  hl=2 l=   3 prim: INTEGER           :010001
```
