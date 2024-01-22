```bash
openssl asn1parse -genconf ed25519.cnf  -noout -out - | openssl pkey -inform DER
```
