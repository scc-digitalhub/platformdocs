Smallstep CA & cli
======================

Smallstep (https://smallstep.com/) provides tools for **identity-driver security**.

In the DigitalHUB context, the *Step Certificates (CA)* project is used as a PKI certification authority
in order to implement a Zero Trust schema for cloud databases.

Via secure TLS proxies (see ghostunnel) cloud databases are safely accessible from end clients, thanks 
to **mutual authentication**: both servers and clients have to possess a valid and mutually accepted certificate 
in order to establish an encrypted communication channel.

*Step Certificates* and the associated *Step cli* are the building blocks for a system which can
automatically craft and deliver *identity based* certificates to end users, via a connection to an
*OIDC identity provider* such as AAC.

Users can perform the authentication process via browser, securely accessing the IdP in a private and safe way
which doesn't expose their credentials to the command line tools.

After completing the *authorization request*, the IdP will provide *step* a JWT encoded OpenID connect **identity token**.

Step will then ask the **certification authority** to produce a short-lived X509 certificate to represent such identity.
After collecting both the *public certificate* and the associated *private key*, users can utilize them to 
establish a connection with a proy like ghostunnel.

1. Server configuration
------------------------

Smallstep Certificates can be executed via Docker or kubernetes. 

See the official doc at https://github.com/smallstep/certificates/

For running the CA in production, operators will have to provide a valid configuration with 
- root and intermediate CA
- secrets for private keys
- JSON configuration for identity providers
- certificate configuration (duration, renowal..)

Everything can be set up by executing the docker image interactively with a local volume, see the upstream docs.
To add an OIDC provisioner execute the ``step cli`` tool and provide:
- valid client-id and client-secret for Oauth
- the `.well-known` configuration URL

Example
```
$ step ca provisioner add aac --type oidc --configuration-endpoint "http://127.0.0.1:9090/aac/.well-known/openid-configuration" --client-id XX --client-secret XX
```


After gathering the required files, the CA can be executed as a deamon and directly exposed on the web via 
port mapping. 
Avoid the usage of a proxy like Nginx, because this would result in the addition of another certificate and a double TLS layer.
Do note that the public facing port should match the locally configured port in order to avoid errors with clients.

Important: save the CA fingerprint because clients will need it to validate the CA certificate on bootstrap.

2. Client execution
------------------------

Client side, the only software required is the `step cli` which can be downloaded as a binary release from github.
The process for obtaining a valid certificate is:

- import the root CA certificate from the public facing URL via ``step bootstrap``.
  This will require the public URL and the CA fingerprint.

- generate a certificate for a given identity via ``step ca certificate`.

Example
```
$ step ca certificate test test.crt test.key
```

Do note that the given identity ("test" in the example) has to match the identity provided by the IdP via OIDC token.
Furthermore, the server component will validate again the JWT token against the JWKS keys associated with the IdP, 
to avoid spoofing.

Example certificate
```
$ step certificate inspect test.pem 
```


Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 168759030285531132221109068689398630028 (0x7ef5ce96530097408229d3d02e8d068c)
    Signature Algorithm: ECDSA-SHA256
        Issuer: CN=step-ca Intermediate CA
        Validity
            Not Before: Jun 28 13:56:35 2019 UTC
            Not After : Jun 29 13:56:35 2019 UTC
        Subject: CN=17
        Subject Public Key Info:
            Public Key Algorithm: ECDSA
                Public-Key: (256 bit)
                X:
                    f6:a2:6d:d2:b3:98:76:ec:4e:b1:90:99:8c:96:29:
                    ad:25:79:ef:70:d6:94:80:a7:94:e4:87:80:f8:fc:
                    53:a3
                Y:
                    47:54:26:cd:b9:f5:5f:d2:e5:39:d9:05:77:df:3f:
                    d5:13:2d:a3:1f:e1:e5:b1:1e:08:f8:23:d8:e9:30:
                    31:98
                Curve: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Subject Key Identifier:
                53:BD:55:0C:39:1C:68:25:1A:48:04:D4:CD:3C:91:92:5A:75:45:EC
            X509v3 Authority Key Identifier:
                keyid:94:7C:89:2F:BC:14:0F:B4:FE:CC:23:2A:EF:44:8A:C0:4C:90:60:54
            X509v3 Subject Alternative Name:
                email:admin
            X509v3 Step Provisioner:
                Type: OIDC
                Name: aac
                CredentialID: da894353-1c0b-4fad-9d0f-cf83e89166ae

    Signature Algorithm: ECDSA-SHA256
         30:46:02:21:00:be:24:a8:d7:e0:8c:f3:fb:62:27:3c:2a:3e:
         3b:08:9e:4e:86:89:d8:93:a2:37:c9:74:da:81:70:27:aa:3f:
         fc:02:21:00:8f:a2:18:da:15:d9:92:a4:48:c1:0d:99:cc:ef:
         f0:ef:7a:b5:6f:42:e0:7d:69:75:78:b0:55:9e:3d:c2:fa:91





