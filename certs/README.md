```bash
openssl asn1parse -genconf google.com.cnf -noout -out - | opnessl x509 -inform DER | diff -u - google.com.crt
```
