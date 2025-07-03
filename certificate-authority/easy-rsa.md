# Certificate Authority with Easy-RSA

## Root CA

1. Download Easy-RSA and extract
2. Rename to `root-ca` if you want (optional)

3. Edit or create file `vars` at `root-ca`

```sh
set_var EASYRSA_REQ_COUNTRY    "VN"
set_var EASYRSA_REQ_PROVINCE   "Ho Chi Minh City"
set_var EASYRSA_REQ_CITY       "Ho Chi Minh City"
set_var EASYRSA_REQ_ORG        "DINHPHU28 Root CA"
set_var EASYRSA_REQ_EMAIL      "ca@dinhphu28.com"
set_var EASYRSA_REQ_OU         "Community"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"
set_var EASYRSA_CURVE          "secp384r1"
```

4. Init pki

```sh
./easyrsa init-pki
```

5. Build CA

```sh
./easyrsa build-ca
```

> Enter and remember the password of Root CA

You now have:

- `pki/private/ca.key`
- `pki/ca.crt` (Root certificate)

For security, keep the Root CA offline and private, for sign certificate, create Intermediate CA.

6. Create custom x509-types

At directory `x509-types`, create your custom type for sign the intermediate CA `x509-types/caRestricted`

```properties
# X509 extensions for a ca

# Note that basicConstraints will be overridden by Easy-RSA when defining a
# CA_PATH_LEN for CA path length limits. You could also do this here
# manually as in the following example in place of the existing line:
#
# basicConstraints = CA:TRUE, pathlen:1

basicConstraints = critical, CA:TRUE, pathlen:0
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
keyUsage = critical, cRLSign, keyCertSign
```

## Intermediate CA

1. Do as step 1..3 of Root CA, but edit `vars`

```sh
set_var EASYRSA_REQ_COUNTRY    "VN"
set_var EASYRSA_REQ_PROVINCE   "Ho Chi Minh City"
set_var EASYRSA_REQ_CITY       "Ho Chi Minh City"
set_var EASYRSA_REQ_ORG        "DINHPHU28 EC CA"
set_var EASYRSA_REQ_EMAIL      "ca@dinhphu28.com"
set_var EASYRSA_REQ_OU         "Intermediate"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"
set_var EASYRSA_CURVE          "secp384r1"

```

2. Init pki (as Root CA)
3. Generate cert request

```sh
./easyrsa gen-req intermediate-ca
```

Generates:

- `pki/private/intermediate-ca.key`
- `pki/reqs/intermediate-ca.req`

Now copy `intermediate-ca.req` to the **offline Root CA**.

3. Go to the Root CA then import cert request

```sh
./easyrsa import-req /path/to/intermediate-ca.req intermediate-ca
```

4. Sign the cert request

```sh
EASYRSA_NO_SAN=1 ./easyrsa sign-req caRestricted intermediate-ca
```

5. Back to Intermediate CA
6. Copy the signed certificate back to Intermediate CA and rename to `ca.crt`

```sh
cp /path/to/intermediate-ca.crt pki/ca.crt
```

7. Copy the Root CA certificate to Intermediate CA and rename to `root-ca.crt`

```sh
cp /path/to/root-ca/pki/ca.crt pki/root-ca.crt
```

8. Create CA chain

```sh
cat pki/ca.crt pki/root-ca.crt > pki/ca-chain.crt
```

Now you are become Intermediate CA.

9. Create CRL (Optional but recommended because of security)

---

## Create Certificate For Microsoft 365 Authentication (MFA)

1. Create config file

```properties
# user_openssl.cnf
[ req ]
default_bits       = 384
prompt             = no
distinguished_name = dn
req_extensions     = req_ext
string_mask        = utf8only
default_md         = sha512

[ dn ]
CN = Dinh Phu Nguyen

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
otherName = 1.3.6.1.4.1.311.20.2.3;UTF8:dinhphu@office.dinhphu28.com

```

2. Generate CSR and key

```sh
openssl ecparam -name secp384r1 -genkey -noout -out dinhphu.key

openssl req -new -key dinhphu.key \
  -out dinhphu.csr \
  -config user_openssl.cnf
```

This CSR now includes your UPN

3. Go to Intermediate CA
4. Import and sign CSR

```sh
./easyrsa import-req /full/path/to/dinhphu.csr dinhphu

EASYRSA_CERT_EXPIRE=60 ./easyrsa --subject-alt-name="otherName:1.3.6.1.4.1.311.20.2.3;UTF8:dinhphu@office.dinhphu28.com" sign-req client dinhphu
```

5. Verify UPN in cert

```sh
openssl x509 -in pki/issued/dinhphu.crt -noout -text | grep -A5 "Subject Alternative Name"
```

Expected result:

```properties
X509v3 Subject Alternative Name:
    otherName:1.3.6.1.4.1.311.20.2.3;UTF8:dinhphu@office.dinhphu28.com
```

6. Create `pfx` file

Prepare:

- dinhphu.key
- dinhphu.crt
- ca-chain.crt

```sh
openssl pkcs12 -export \
  -inkey dinhphu.key \
  -in dinhphu.crt \
  -certfile /path-to/ca-chain.crt \
  -out dinhphu.pfx \
  -name "Dinh Phu Nguyen"
```

You'll be prompted for a password to protect the `.pfx` 

**Note:** Above client cert `dinhphu.crt` and packaged `dinhphu.pfx` is only contain client cert (not included )

To verify the client cert valid, we not only need the Root CA cert but also Intermediate CA.
`Root CA -> Intermediate CA -> Client cert`

Most of SSL Tools or System use use client cert with fullchain (intermediate-a -> leaf) to verify the client with only Root CA cert.

To do it:

```sh
cat /path/to/intermediate-ca/pki/ca.crt dinhphu.crt > dinhphu-fullchain.crt
```

Package `pfx` file again with fullchain cert.

### macOS cert conversion

Because of this `pfx` using pkcs12 is not compatible with macOS, so we need to convert it to legacy.

```sh
openssl pkcs12 -in dinhphu.pfx -out dinhphu.pem
openssl pkcs12 -export -in dinhphu.pem -out dinhphu_macos.pfx -legacy
```

### Note for Microsoft Entra

#### Authentication Binding

**Default authentication strengh**: Mutli-factor

**Required Affinity Binding**: Low

Add rule

1. Policy OID

- Certificate attribute: Policy OID
- Policy OID: `1.3.6.1.4.1.311.20.2.2`
- Authentication strength: Mutli-factor authentication
- Affinity binding: Low

#### Username Binding

| Certificate field | Affinity binding | User attribute    |
| ----------------- | ---------------- | ----------------- |
| PrincipalName     | Low              | userPrincipalName |
| RFC822Name        | Low              | userPrincipalName |

For more: https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-certificate-based-authentication#step-4-configure-username-binding-policy
