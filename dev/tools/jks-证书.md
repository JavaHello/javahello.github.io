# java

## pkcs12

```sh
sudo openssl pkcs12 -export -in vault.crt -inkey vault.key -out vault.p12 -name "vault"
```

## jks

```sh
sudo keytool -importkeystore -srckeystore vault.p12 -srcstoretype PKCS12 -deststoretype JKS -destkeystore vault.jks
```

## import

```sh
keytool -list -keystore $JAVA_HOME/lib/security/cacerts

# 默认密码 changeit
keytool -import -alias <证书别名> -keystore $JAVA_HOME/jre/lib/security/cacerts -file your.crt

```

keytool -import -alias vault -keystore cacerts -file /etc/pki/ca-trust/source/anchors/tls.crt
