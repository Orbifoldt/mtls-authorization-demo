# Generate certificates for Mutual-TLS
This explains how to generate server and client certificates by creating your own Certificate Authority with openssl.

## Creating a Certification Authority certificate

First, we need a Certificate Authority (CA) which provides the root certificate. Start by generating a private key for the root CA:
```bash
openssl genrsa -out rootCA.key 2048
```
Then create a self-signed certificate with this:
```bash
openssl req -x509 -sha256 -new -nodes -key rootCA.key -days 3650 -out rootCA.crt -subj "/C=NL/ST=Noord Holland/L=Amsterdam/O=Test Inc./OU=Sec/CN=my-root-ca.com"
```
This is the signed certificate of our root CA. To view this root CA certificate run:
```bash
openssl x509 -in rootCA.crt -text -noout
```


## Creating and signing server certificate
Create a private key and a Certificate Signing Request (CSR) for our certificate that we want to be used by the server:
```bash
openssl req -nodes -sha256 -newkey rsa:2048 -keyout server.key -out server.csr -subj "/C=NL/ST=Noord Holland/L=Amsterdam/O=Test Inc./OU=Services/CN=my-provider-server.com"
```
Verify this CSR using:
```bash
openssl req -text -noout -verify -in server.csr
```
Now, using our root CA, we can sign our server certificate:
```bash
openssl x509 -req -in server.csr -days 365 -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out server.crt
```

Then, additionally, we can create a keystore. This is a file that contains both the private key as well as the public, signed certificate.
```bash
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt -passout pass:my_pw
```
Here, `my_pw` is the password used to encrypt this keystore.

Sometimes you also want to include the root CA certificate with which the certificate was signed:
```bash
openssl pkcs12 -export -out server-including-root.pfx -inkey server.key -in rootCA.crt -in server.crt -passout pass:my_pw
```
Note that here the `-in <cert-file>` arguments should obey the following order: root, intermediate (if any) and then finally the certificate in question.



## Creating and signing client certificate
Creating a signed certificate for the client is exactly the same procedure as how the server certificate was generated. Skipping any verification, the commands you need are:
```bash
openssl req -nodes -sha256 -newkey rsa:2048 -keyout client.key -out client.csr -subj "/C=NL/ST=Noord Holland/L=Amsterdam/O=Test Inc./OU=Consumers/CN=my-client.com"
openssl x509 -req -in client.csr -days 365 -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out client.crt
openssl pkcs12 -export -out client.pfx -inkey client.key -in client.crt -passout pass:my_pw
openssl pkcs12 -export -out client-including-root.pfx -inkey client.key -in rootCA.crt -in client.crt -passout pass:my_pw
```
