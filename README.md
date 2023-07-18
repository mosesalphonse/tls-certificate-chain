# tls-certificate-chain
create our own tls certificate chain steps

#  get the config file into the current directory:

```
git clone 

cd 
```

# Root CA key for self-signing

```
openssl genrsa -out sash_root.key 4096
```
# Create CSR and self-sign CA cert
```
openssl req -x509 -new -nodes -key sash_root.key -days 3650 -config sash.cfg -out sash_root.pem
```

# Generate a private key for our certificate:

```
openssl genrsa -out sash_cert.key 4096
```

# Create CSR for the server:

```
openssl req -new -key sash_cert.key -out sash.csr -config sash.cfg
```

# Sign our certificate with Root CA

```
openssl x509 -req -in sash.csr -CA sash_root.pem -CAkey sash_root.key -CAcreateserial  -out sash_server.crt -days 365 -sha256
```
