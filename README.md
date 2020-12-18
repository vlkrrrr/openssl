# Setup a root ca and create client certs without intermediate cert

### openssl specifics
- A csr request doesn't need to be a seperate step; it can be included in a openssl command
- req command is for csr (create/view)
- [ext] group in a openssl cnf file for csr might not be nescessary, because a ca can decide on its own which extensions it grants to the certificate.
- data in the subject, like localityName, must be mentioned in the policy, too: localityName = optional

### create root ca csr and key (openssl can do this together)
openssl req -new -config my-ca.cnf -out my-root-ca.csr -keyout my-root-ca.key

### create db
touch my_ca_index.db 
echo 01 > ca_my_index.db.serial   //serial file for db
to create a nicer serial: openssl rand -hex 16  > mySerial.ser

### create root ca cert
openssl ca -selfsign -config my-ca.cnf -in my-root-ca.csr -out my-root-ca.crt -extensions ca_ext

### create client cnf
by hand
### create client keypair
openssl genrsa -aes256 -out my-client.key 2048
### create client csr
openssl req -new -config my-client.cnf -key my-client.key -out my-client.csr

### create client cert and sign it with ca (this done with use of my-ca.cnf)
openssl ca -config my-ca.cnf -in my-client.csr -out my-client.crt -extensions client_ext


## helpful commands
### review csr:
openssl req -in my-root-ca.csr -text

### generate public key
openssl rsa -in fd.key -pubout -out fd-public.key

### verify (intermediate not needed)
openssl verify -CAfile RootCert.pem -untrusted Intermediate.pem UserCert.pem

### create pem
openssl x509 -inform der -in cert.cer -out cert.pem
delete not needed stuff from original root and client crt

### create pkcs12
some systems require to have the cert chain inside .p12. you can just copy the root/itnermediate pem cert inside the client-pem cert. you can add additional certs to the .p12 with the -certfile more.crt option.
openssl pkcs12 -export -name "myclient" -out my-client.p12 -inkey my-client.key -in my-client.crt

## to create intermediate crt add these steps
### create sub-ca csr
openssl req -new -config sub-ca-cert.cnf -out my-sub-ca.csr -keyout my-sub-ca.key

### create sub-ca cert from csr
openssl ca -config my-ca.cnf -in my-sub-ca.csr -out my-sub-ca.crt -extensions sub_ca_ext