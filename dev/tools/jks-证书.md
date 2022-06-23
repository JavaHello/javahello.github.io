# pkcs12

```sh
sudo openssl pkcs12 -export -in vault.crt -inkey vault.key -out vault.p12 -name "vault"
```

# jks

```sh
sudo keytool -importkeystore -srckeystore vault.p12 -srcstoretype PKCS12 -deststoretype JKS -destkeystore vault.jks
```
