# Create Certificate Authority

## Generate private key of root CA encrypted with AES
`openssl genrsa -aes256 -out rootCA.key 2048`

## Create root CA certificate
`openssl req -x509 \
	-sha256 -days 3650 \
	-subj "/CN=smskt.local/C=IN/L=Rajapalayam" \
	-key rootCA.key -out rootCA.crt`

# Create Self-Signed Certificates

## Generate the server private key
`openssl genrsa -out server.key 2048`

## Create Certificate Signing Request config file

`
cat > csr.conf << EOF

[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn


[ dn ]
C=IN
L=Rajapalayam
O=SMSKT
OU=Home Server
CN=homeserver.local

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = homeserver.local
DNS.2 = *.homeserver.local

EOF
`

## Generate CSR using server private key
`openssl req -new -key server.key -out server.csr -config csr.conf`


## Create external config file
`
cat > cert.conf << EOF

authorityKeyIdentifier = keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = homeserver.local
DNS.2 = *.homeserver.local

EOF
`

## Generate SSL certificate with Self-Signed CA
`
openssl x509 -req \
	-in server.csr \
	-CA rootCA.crt -CAkey rootCA.key \
	-CAcreateserial -out server.crt \
	-days 365 \
	-sha256 -extfile cert.conf
`
