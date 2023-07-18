# tls-certificate-chain
create our own tls certificate chain steps

#  get the config file into the current directory:

```
git clone https://github.com/mosesalphonse/tls-certificate-chain.git

cd tls-certificate-chain

```

# Root CA private key ( for self-sign CA, sign the server certificate)

```
openssl genrsa -out sash_root.key 4096

```

# Create CA CSR and self-sign CA cert using the above Key

```
openssl req -x509 -new -nodes -key sash_root.key -days 3650 -config sash.cfg -out sash_root.pem

```

# Generate private key for server certificate:

```
openssl genrsa -out sash_cert.key 4096

```

# Create CSR for the server certicate:

```
openssl req -new -key sash_cert.key -out sash.csr -config sash.cfg

```

# Sign the server certificate with Root CA

```
openssl x509 -req -in sash.csr -CA sash_root.pem -CAkey sash_root.key -CAcreateserial  -out sash_server.crt -days 365 -sha256

```
