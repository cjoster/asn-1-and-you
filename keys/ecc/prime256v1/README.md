## EC Parameters

The prameters file contains a single OID, the OID of the named eliptical curve.
Generating the prameter file:

```bash
openssl asn1parse -genconf prime256v1-param.cnf -noout -out - | openssl ecparam -inform DER
```

## EC Private Key

The private key sequence is defned in [RFC5915](https://datatracker.ietf.org/doc/html/rfc5915) as follows:

```
ECPrivateKey ::= SEQUENCE {
  version        INTEGER { ecPrivkeyVer1(1) } (ecPrivkeyVer1),
  privateKey     OCTET STRING,
  parameters [0] ECParameters {{ NamedCurve }} OPTIONAL,
  publicKey  [1] BIT STRING OPTIONAL
}
```

You'll find a key in [prime256v1.key](prime256v1.key), and a corresponding configuration file
[prime256v1.cnf](prime256v1.cnf). The key can be generated with...

```bash
openssl asn1pares -genconf prime256v1.cnf -noout -out - | openssl ec -inform DER
```
