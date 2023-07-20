# tls-certificate-chain
Create our own TLS certificate chains

# 1) Create server certificate chain without any intermediate certificate:


   
## Root CA:


### Root CA private key ( for self-sign CA, sign the server certificate)

```
openssl genrsa -out Sash_CA.key 2048

```

### Create CA CSR and self-sign CA cert using the above Key

```
openssl req -new -sha256 -key Sash_CA.key -out Sash_CA.csr -subj "/C=UK/ST=Gloucester/L=Cirencester/O=Sash/CN=SASH CA CERTIFICATE"

```

### Generate CA Certificate

```
openssl x509 -signkey Sash_CA.key -in Sash_CA.csr -req -days 3650 -out Sash_CA.pem

```

### View Certificate

```
openssl x509 -text -noout -in Sash_CA.pem

```

## Generate Server Certificate directly signed by CA

### Generate private key for server certificate:

```
openssl genrsa -out sash1.key 2048

```

### Create CSR for the server certicate:

```
openssl req -new -sha256 -key sash1.key -out sash1.csr -subj "/C=UK/ST=Gloucester/L=Cirencester/O=Sash/CN=ssh.com/subjectAltName=sash1.com"

```

### Sign the server certificate using Root CA

```
openssl x509 -req -in sash1.csr -CA Sash_CA.pem -CAkey Sash_CA.key -CAcreateserial -out sash1.pem -days 3650 -sha256

```

### View Certificate

```
openssl x509 -text -noout -in sash1.pem

```

### verify Certificate chain's signature using CA's public key 

```
openssl verify -CAfile Sash_CA.pem -untrusted  Sash_CA.pem sash1.pem

```
**where** 
  -CAfile --> CA certificate
  -untrusted --> First value should be issuer certificate, in this case CA is the issuer. In case of multiple intermidiate CAs, combine all the 
   intermidiate CA certificates in one single file

            --> Second value is server(leaf) certificate

**Note**: While verifying the signature of the whole certificate chain, it uses public key which is embbed in the issuer certificate.


  
