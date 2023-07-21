# SSL / TLS certificate-chain

**Usefull Openssl commands to create complete SSL/TLS certificate chain, verify the signature, encrypt and decypt sample file.**

<li>
Create complete chain of SSL/TLS certificates
   </li>
   <li>
Verify the signature using the CA's public keys
      </li>
      <li>
Encrypt the file using public key and dercypt the encrypted text using private key
</li>

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
       -   --> Second value is server(leaf) certificate

**Note**: While verifying the signature of the whole certificate chain, it uses public key which is embbed in the issuer certificate.


# 2) Create server certificate chain with one intermediate CA certificate:  


## Root CA:


### Root CA private key ( for self-sign CA, sign the server certificate)

```
openssl genrsa -out RootCA.key 4096

```
### Create CA CSR and self-sign CA cert using the above Key

```
openssl req -new -x509 -days 1826 -key RootCA.key -out RootCA.crt

```
### view the Root CA certificate

```
openssl x509 -in RootCA.crt -noout -text

```
### Verify the Root CA's signature (its self signed)

```
openssl verify -CAfile RootCA.crt -untrusted RootCA.crt RootCA.crt

```

## Intermediate  CA


### Intermediate CA private key

```
openssl genrsa -out IntermediateCA.key 4096

```
### Create Intermediate CA CSR and sign with its private Key
```
openssl req -new -key IntermediateCA.key -out IntermediateCA.csr

```
Create domainCA.ext file with the below content. This will enable the intermediate certificate as CA (CA:TRUE) extentions

**domainCA.ext**
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:TRUE
keyUsage = keyCertSign, cRLSign, digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
```

### Sign Intermediate certificate with Root CA's private key and generate Intermediate Certificate

```
openssl x509 -req -days 1000 -in IntermediateCA.csr -CA RootCA.crt -CAkey RootCA.key -CAcreateserial -out IntermediateCA.crt -extfile domainCA.ext

```
### view the Intermediate CA certificate

```
openssl x509 -in IntermediateCA.crt -noout -text

```
### Verify the Intermediate CA's signature (it is signed by Root CA)

```
openssl verify -CAfile RootCA.crt -untrusted RootCA.crt IntermediateCA.crt

```

## Server (leaf) Certificate:


### Server(leaf) private key

```
openssl genrsa -out Server.key 2048

```
### Create server (leaf) CSR and sign with its private Key
```
openssl req -new -key Server.key -out Server.csr

```

### Sign server(leaf) certificate with Intermediate CA's private key and generate server (leaf) Certificate

```
openssl x509 -req -days 1000 -in Server.csr -CA IntermediateCA.crt -CAkey IntermediateCA.key -set_serial 0101  -out Server.crt -sha1

```
### view the Intermediate CA certificate

```
openssl x509 -in Server.crt -noout -text

```
### Verify the Intermediate CA's signature (it is signed by Root CA)

```
openssl verify -CAfile RootCA.crt -untrusted IntermediateCA.crt Server.crt

```


# 3) Encrypt and Decrypt:  


### Extract public key from the certificate

```
openssl x509 -pubkey -noout -in Server.crt  > pub.pem

openssl x509 -pubkey -noout -in IntermediateCA.crt  > inter_pub.pem

```

### Encrypt the sample file using public key

```
openssl rsautl -encrypt -pubin -inkey pub.pem -in sash.txt | base64 > sash_encrypted.txt

openssl rsautl -encrypt -pubin -inkey inter_pub.pem -in sash.txt | base64 > inter_encrypted.txt

```

### Decrypt the encrypted file using private key

```
cat sash_encrypted.txt | base64 -d | openssl rsautl -decrypt -inkey Server.key

cat inter_encrypted.txt | base64 -d | openssl rsautl -decrypt -inkey IntermediateCA.key

```
